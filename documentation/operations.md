# Operations

[Documentation](index.md) · [Configuration](configuration.md) · [Validation](validation.md)

The repository includes a Vercel web descriptor and a Render Docker gateway descriptor. They describe intended deployment mechanics, not the current health, billing, or release status of any hosted account. Deployment, shared database changes, provider requests that consume usage, and account policy changes require the operator's authorization.

## Deployment topology

The Next.js application serves pages, authentication, session APIs, and scoring. The voice gateway requires a persistent process that accepts WSS connections. It cannot be replaced by a short-lived web request handler. Supabase, Gnani, and the reasoning service are separate external dependencies.

The [gateway Dockerfile](../Dockerfile.gateway) builds with Node 24, installs runtime dependencies separately, and runs as the unprivileged `node` user. [render.yaml](../render.yaml) uses that Dockerfile, `/health`, a `free` plan selection, and disables automatic deployment triggers. Verify current plan availability, persistent-connection behavior, wake time, and price with the operator; source configuration cannot establish those external facts.

## Prepare a release

1. Identify the exact source revision and target environment. Review outstanding [acceptance items](validation.md).
2. Run `npm ci`, `npm run check`, `npm run gateway:build`, and `npm run build` in a clean checkout.
3. Have an authorized operator compare target database history with the [seven SQL files](database.md#sql-sequence), back up the database, and apply only reviewed pending definitions.
4. Configure the gateway's private environment, matching `APP_ORIGIN` and the selected reasoning provider to the web process. Use `SPEECH_GATEWAY_HOST=0.0.0.0` for the container; omit `SPEECH_GATEWAY_PORT` when the host supplies `PORT`.
5. Deploy the gateway behind TLS and verify `/health`. The public endpoint must be `wss://` with the exact `/speech` path.
6. Configure and deploy the web application with that gateway URL. Keep preview disabled and voice disabled until the intended test scope is ready. Public Supabase settings must be present at web build time.
7. Verify origin rejection, signed-out denial, cross-account access denial, and actual authentication policy before enabling authorized voice testing.
8. Run the [real browser acceptance flow](validation.md#live-acceptance) and inspect the rendered transcript and report. Enable broader access only after the required evidence is accepted.

Do not interpret an environment check, a successful deployment, or an HTTP 200 health response as voice-to-report acceptance. There is no deployment CLI wrapper in the repository.

## Health and diagnostics

| Check                                  | Proves                                                                        | Does not prove                                                              |
| -------------------------------------- | ----------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| Web `GET /api/health`                  | Web handler responds, with a UTC timestamp                                    | Database, identity, voice, scoring, or cost readiness                       |
| Gateway `GET /health`                  | Gateway HTTP process responds                                                 | Provider connectivity, grants, database writes, or audio upload reliability |
| `config:check`                         | Selected local fields/limits/contact/recovery secret pass the script's checks | Complete runtime validation or release readiness                            |
| `speech:check`                         | Speech credential presence/shape and reported prerequisites                   | Account entitlement or a live session                                       |
| Authorized `reasoning:check -- --live` | Model catalog response and configured ID visibility                           | Inference quality, quota, latency, or JSON compatibility                    |
| Authorized `speech:probe -- --live`    | One bounded synthetic REST round trip                                         | Browser audio, realtime WebSocket transport, or a valid student report      |

The application emits structured, sanitized console events. No centralized log collector, dashboard, alert configuration, or external error tracker is included. The host/operator must provide log retention and monitoring appropriate to the deployment.

| Event                                                | Investigate                                                                                            |
| ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| `request_failed`                                     | Web error code and HTTP status; compare with configuration/dependency availability.                    |
| `provider_request_rejected`                          | Operation and upstream status, without upstream bodies.                                                |
| `session_processing_deferred`                        | Work remains durable; check dependency availability and recovery scheduling.                           |
| `scorecard_rejected`                                 | Schema stage, bounded field paths, token limit, or ungrounded quote category.                          |
| `gateway_storage_rejected`                           | Sanitized SQL/operation/action category; distinguish semantic rejection from unavailable storage.      |
| `dialogue_rejected`                                  | Fixed dialogue format/transport/category; eligible unpublished transient requests have only one retry. |
| `speech_synthesis_rejected`, `speech_input_rejected` | Synthesis/framing/pacing bounds or provider transport failure.                                         |
| `speech_transport_stopped`, `speech_run_failed`      | Failure category and bounded timing/frame counters.                                                    |

Keep provider keys, grants, session identities, account details, audio, transcripts, and raw provider bodies out of logs and public incident reports. Existing transport counters help compare failures but do not identify a root cause on their own.

## Connection failure procedure

If students report an interrupted round, direct them to **Check round status** or **History**. A voice stream is non-resumable. Never reserve another grant automatically after an unknown outcome and never convert partial evidence into a successful report to hide a failure.

For diagnosis, record a sanitized UTC time window, visible error, browser/device, network conditions, release revision, and fixed log categories. Compare the browser upload path, gateway input counters, provider connection, and database availability. A healthy gateway does not rule out upload stalls. Do not remove bounds or inflate queues without evidence explaining where the interruption occurs.

## Durable processing and recovery

Reading an owned active session and requesting completion can schedule server processing. The feedback page polls every four seconds until terminal status, with bounded read retries. Browser polling does not restart voice or create new provider identities.

`GET /api/internal/reconcile` requires the bearer recovery secret. Use the approved scheduler/secret store; never place the secret in a URL. The endpoint expires unstarted rounds older than 30 minutes and checks at most two due nonterminal sessions. SQL rejects stale leases, expired grants, and incomplete voice streams. Scoring has two persisted attempts at most.

[vercel.json](../vercel.json) schedules recovery daily at midnight UTC. If a browser closes and no status request arrives, recovery is not promised within minutes. An expired processing deadline may produce failure instead of a report. The two-record batch can also leave a backlog. A different schedule/worker is a separate reviewed operational change; no stronger recovery guarantee is implemented here.

## Stop new usage and roll back

To stop new voice grants, set `VOICE_RUNTIME_ENABLED=false` on the web application using the host's approved release procedure. Existing grants and in-progress gateway connections are not revoked by this flag. Stopping/restarting the gateway fails its live calls and loses in-memory audio queues; warn affected users and choose the interruption window deliberately.

Keep the previous web and gateway release artifacts available. Roll back compatible application versions through the host. SQL rollback is a separate backup/restore decision; there are no automatic reverse SQL files. Do not reset a shared database to repair a voice or documentation issue.

## Privacy and cost

Samvaad persists account/session information, final text evidence, and feedback, not raw audio. It has no self-service deletion, transcript export, automated retention purge, or provider-data deletion workflow. The operator must handle data requests privately and establish external provider retention/training terms before rollout.

The default daily limit is ten created rounds per user. The monthly limit is thirty reserved planned voice minutes for the entire application. Reservations are counted even when the round fails; there is no refund ledger. These limits do not cap the currency bill, because reasoning requests, speech requests, hosting, and provider charging rules are separate. Inspect actual account usage and billing before increasing allowances; do not promise a price or free tier from source alone.

Authentication has no independent application-level login-rate limiter; the operator must review Supabase account protections. Unverified-email mode disables application recovery emails. Do not restore account access based only on a claimed email address.
