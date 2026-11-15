# API and voice protocol

[Documentation](index.md) · [Architecture](architecture.md) · [Configuration](configuration.md)

The browser uses same-origin HTTP requests with Supabase session cookies. Private session reads and every session write require an authenticated owner. POST requests require an `Origin` header exactly matching `APP_ORIGIN`. There is no public machine-to-machine API key or transcript-upload endpoint.

## HTTP endpoints

| Method and path                         | Access and input                                                   | Success body                                        |
| --------------------------------------- | ------------------------------------------------------------------ | --------------------------------------------------- |
| `GET /api/health`                       | Public liveness only                                               | `{status, time}`                                    |
| `POST /api/auth`                        | Same-origin; strict action body                                    | `{message, next}`                                   |
| `GET /api/topics`                       | No user identity required; authentication configuration must exist | `{topics}` with at most five active topics          |
| `GET /api/practice-options`             | Signed-in user                                                     | `{contractVersion, modes, quota}`                   |
| `POST /api/sessions`                    | Signed-in user; setup body and UUID `Idempotency-Key` header       | `{session}`                                         |
| `GET /api/sessions`                     | Signed-in user; optional bounded query                             | `{sessions, stats, nextCursor}`                     |
| `GET /api/sessions/{id}`                | Owned UUID session                                                 | `{session}`; may schedule due processing            |
| `POST /api/sessions/{id}/voice-session` | Owned UUID session; exact body `{}`                                | `{runId, grant, websocketUrl, expiresAt, protocol}` |
| `POST /api/sessions/{id}/complete`      | Owned UUID session; exact body `{}`                                | `{session}`; may schedule processing                |
| `GET /api/internal/reconcile`           | `Authorization: Bearer` followed by `CRON_SECRET`                  | `{checked}`                                         |

Success responses use HTTP 200. JSON helper responses set `Cache-Control: private, no-store`. Unknown product paths return `SESSION_NOT_FOUND`. Product routes use the Node runtime and declare `maxDuration=120` seconds where background processing is needed; host support must be verified separately.

### Authentication

The strict body accepts `action` from `login`, `signup`, `reset`, `update`, `logout`; optional `email`, `password`, and `next` fields depend on the action. Login/signup require email and password. Passwords are 10–128 characters. Update requires a verified current server session and a password. Reset is blocked with HTTP 409 when email verification is disabled. Logout signs out through Supabase.

The next destination must be a local path beginning with one slash; absolute/protocol-relative destinations, backslashes, and line breaks are rejected in favor of `/`. `/auth/callback` exchanges a Supabase code and redirects only to a safe local destination. Provider authentication failures use a generic HTTP 400 `AUTH` error rather than exposing provider details.

### Practice options and creation

The current server emits `contractVersion: 2`, five unique mode entries, up to five topics per mode, permitted durations, rubric versions, availability/reasons, and a daily quota. Configuration-derived availability is not a live provider-health or browser-acceptance check.

Creation accepts exactly these fields:

| Field             | Accepted value                                                                                              |
| ----------------- | ----------------------------------------------------------------------------------------------------------- |
| `topicId`         | UUID of an active database topic                                                                            |
| `difficulty`      | `gentle`, `standard`, `intense`                                                                             |
| `durationSeconds` | `180` or `300`, within the configured maximum                                                               |
| `mode`            | Optional: `interview`, `viva`, `debate`, `group_discussion`, `presentation`; omitted means group discussion |

Reuse the same UUID creation key for an uncertain retry with identical input. A changed topic, difficulty, duration, or mode with the same owner/key conflicts. A different owner has an independent key namespace. Creation checks quota atomically in the database before creating a new record. The browser cannot supply a user ID, transcript, score, provider execution ID, or voice participant policy.

### Session and history responses

A session summary includes `id`, `mode`, `rubricVersion`, `topic`, `difficulty`, `durationSeconds`, `status`, timestamps, and nullable `overallScore`. Details add nullable `transcript`, `scorecard`, and `error`. A completed report must have a valid transcript and scorecard whose calculated totals agree; unfinished sessions do not expose an overall score.

Transcript entries contain `speaker`, `text`, and optional `actor`/`delivery`. Actor/delivery must agree with the student or assistant evidence channel. Session and response validators are defined in [product-contracts.ts](../src/lib/product-contracts.ts).

History accepts `limit` from 1 to 50, default 20; optional `status`; and an opaque `cursor` returned by the previous response. Cursor content is bounded and validated. Ordering is descending by creation time and ID. A next cursor appears only if another page exists. Use the returned cursor unchanged.

History statistics include completed count, average, focus, and an optional compatible comparison with at most 20 chronological points. Compatibility includes mode, topic, rubric, pressure, and duration. Summary statistics can span all completed rounds; the comparison chart is deliberately narrower.

### Completion and recovery

`complete` does not finalize microphone evidence and accepts no transcript. Only the authenticated gateway can finalize the evidence snapshot. The endpoint requests/checks durable work for the existing owned session. It cannot resume a voice run or reserve a replacement grant.

The recovery endpoint is an operator endpoint. It expires created sessions older than 30 minutes, then attempts at most two due active/processing records. `checked` counts selected records, not guaranteed reports. Scheduled recovery frequency is defined by [vercel.json](../vercel.json).

## Errors

The normal endpoint wrapper returns `{error: {code, message, retryable}}`. The auth provider-failure branch omits `retryable`. Never rely on raw database or provider messages reaching a browser.

| HTTP status | Representative codes                                                                                                                         | Caller behavior                                                |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------- |
| 400         | `VALIDATION_ERROR`, `AUTH`                                                                                                                   | Correct the request or account details.                        |
| 401         | `UNAUTHENTICATED`                                                                                                                            | Sign in again.                                                 |
| 403         | `FORBIDDEN`, `VERIFY_EMAIL`                                                                                                                  | Do not bypass origin, secret, or identity checks.              |
| 404         | `SESSION_NOT_FOUND`                                                                                                                          | Missing and unowned sessions are indistinguishable.            |
| 409         | `IDEMPOTENCY_CONFLICT`, `SESSION_NOT_ACTIVE`, `SESSION_NOT_READY`, `SESSION_ALREADY_ACTIVE`, `SESSION_EXPIRED`, `EMAIL_RECOVERY_UNAVAILABLE` | Inspect the existing session or supported account flow.        |
| 413         | `VALIDATION_ERROR`                                                                                                                           | Request body exceeded 32,000 bytes.                            |
| 429         | `RATE_LIMITED`, `CAPACITY_LIMIT`                                                                                                             | Wait for allowance reset or operator action.                   |
| 500         | `INTERNAL_ERROR`                                                                                                                             | Report the sanitized failure.                                  |
| 503         | `CONFIGURATION`, `SERVICE_UNAVAILABLE`, `VOICE_WARMING`                                                                                      | Service unavailable; apply only the safe retry behavior below. |

Read-only status polling retries eligible network/500/502/503/504 failures at most twice. Grant requests retry only an explicit `VOICE_WARMING` response, which occurs before reservation, within the browser's bounded warmup window. Never automatically repeat a grant request with an unknown outcome. `retryable` metadata does not authorize replaying a paid or single-use operation.

## Owned WebSocket protocol

The gateway is a separate process. `GET /health` is public liveness. Upgrade requests must use `/speech`, the exact configured origin, and fit the gateway's eight-client limit. Other upgrades are rejected. The server's maximum WebSocket payload is 4,096 bytes; the first client authentication message must arrive within five seconds.

Connect to the URL returned by the grant endpoint and authenticate using a JSON control message containing `type: authenticate`, the returned `grant`, and `protocol: samvaad-speech-v1`. The grant is sent in a message, never a URL query. It is consumed before a provider connection starts.

| Direction         | Message        | Contract                                                                                  |
| ----------------- | -------------- | ----------------------------------------------------------------------------------------- |
| Client to gateway | `authenticate` | Strict fields: `type`, `grant`, `protocol`; one attempt per connection                    |
| Client to gateway | Binary PCM     | Exactly 1,024 bytes/frame; 16 kHz mono PCM16 little-endian, 32 ms/frame; only after ready |
| Client to gateway | `end`          | Strict `type` only; request manual finalization                                           |
| Client to gateway | `ping`         | Strict `type` only; keepalive                                                             |
| Gateway to client | `ready`        | `runId`, ISO `deadline`, `sampleRate:16000`, `remainingMilliseconds`                      |
| Gateway to client | `state`        | `state`: listening/thinking/speaking; `actor`: student/coach/moderator/peer_a/peer_b      |
| Gateway to client | `clear`        | New playback `generation`; discard stale audio                                            |
| Gateway to client | `audio`        | `generation` and base64 PCM in `pcm`                                                      |
| Gateway to client | `ended`        | `reason`: manual/deadline                                                                 |
| Gateway to client | `error`        | Public `code: VOICE_FAILED`; no provider body                                             |
| Gateway to client | `pong`         | Keepalive response                                                                        |

There is no resume, browser transcript, caption, or client-authored final-segment message. Protocol validation is in [protocol.ts](../src/lib/voice/protocol.ts); transport lifecycle is in [gateway.ts](../src/lib/voice/gateway.ts).
