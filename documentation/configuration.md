# Configuration

[Documentation](index.md) · [Development](development.md) · [Operations](operations.md)

This reference follows [.env.example](../.env.example), [server configuration](../src/lib/server/config.ts), [reasoning profiles](../src/lib/reasoning-config.mjs), and the [gateway entry point](../gateway/main.ts). Defaults below describe code behavior, not account availability or provider guarantees.

Keep local values in ignored `.env.local`. Only the Supabase URL and publishable key use `NEXT_PUBLIC_`; every other credential stays in trusted server processes. Never put credentials in documentation, browser URLs, screenshots, or logs.

## Web application settings

| Variable                               | Meaning and default                                                                                                                                                                                       |
| -------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `APP_ORIGIN`                           | Exact web origin. Sample: `http://127.0.0.1:3013`. Required for write-origin checks and voice configuration. Use the canonical HTTPS origin when hosted.                                                  |
| `NEXT_PUBLIC_SUPABASE_URL`             | Supabase project URL; also needed by the gateway.                                                                                                                                                         |
| `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` | Browser-safe Supabase publishable key used for cookie-based authentication.                                                                                                                               |
| `SUPABASE_SECRET_KEY`                  | Privileged Supabase server credential; also needed by the gateway. Never expose to a browser.                                                                                                             |
| `AUTH_EMAIL_VERIFICATION_REQUIRED`     | Verification is required unless the value is exactly `false`. The sample sets `false`. Match the Supabase confirmation policy.                                                                            |
| `GNANI_SPEECH_API_KEY`                 | Private speech credential. The web checks presence; the gateway makes live speech requests.                                                                                                               |
| `SAMVAAD_VOICE_WS_URL`                 | Owned gateway URL with the exact `/speech` path. Sample: `ws://127.0.0.1:3015/speech`. WSS is required outside the permitted local HTTP/loopback combination. No credentials, query, or fragment allowed. |
| `VOICE_RUNTIME_ENABLED`                | Live session creation is enabled only when exactly `true`; sample/default is disabled.                                                                                                                    |
| `VOICE_GATEWAY_WARMUP_ENABLED`         | When exactly `true`, test the gateway's `/health` before reserving a grant or voice allowance. Default disabled.                                                                                          |
| `MAX_SESSION_DURATION_SECONDS`         | `180` or `300`; default `300`. Controls which durations the server exposes and accepts.                                                                                                                   |
| `MAX_DAILY_SESSIONS_PER_USER`          | Integer `1..100`; default `10`. Counts created rounds per owner per UTC day.                                                                                                                              |
| `MAX_MONTHLY_VOICE_MINUTES`            | Integer `1..10000`; default `30`. Shared admission limit based on planned duration reserved across all users in a UTC month. It is not a currency spending cap.                                           |
| `CRON_SECRET`                          | Application recovery bearer secret, at least 32 characters. Required by `/api/internal/reconcile`.                                                                                                        |
| `PRIVACY_CONTACT_EMAIL`                | Operator-provided public data-request contact. The configuration checker requires an email-shaped value. It is not a login or recovery credential.                                                        |
| `UI_PREVIEW_ENABLED`                   | Synthetic review route is enabled only when exactly `true`; otherwise `/preview` returns 404. Keep disabled publicly.                                                                                     |

Changing the email-verification setting changes sign-in and recovery behavior. In the supplied unverified-email setup, Supabase must automatically confirm accounts and the app disables forgotten-password emails. Signed-in password changes remain available. The code's verified-email branch is not evidence of a configured or tested mail-delivery service.

## Reasoning service

Both the web scorer and gateway dialogue controller use the same resolver. Configure the same selection deliberately in both processes. Requests use the selected provider only; errors never trigger a switch to another provider.

| Selection                   | Required private key | Model setting and code default              | Base URL in code                                          |
| --------------------------- | -------------------- | ------------------------------------------- | --------------------------------------------------------- |
| `REASONING_PROVIDER=groq`   | `GROQ_API_KEY`       | `GROQ_MODEL`, default `openai/gpt-oss-120b` | `https://api.groq.com/openai/v1`                          |
| `REASONING_PROVIDER=gemini` | `GEMINI_API_KEY`     | `GEMINI_MODEL`, default `gemini-3.8-flash`  | `https://generativelanguage.googleapis.com/v1beta/openai` |
| `REASONING_PROVIDER=custom` | `REASONING_API_KEY`  | Required `REASONING_MODEL`                  | Required `REASONING_BASE_URL`                             |

The sample explicitly selects Groq. If `REASONING_PROVIDER` is absent, a manual base URL or model selects `custom`; otherwise code selects Gemini. Set the provider explicitly to avoid this implicit selection.

Custom URLs must be HTTPS without embedded credentials, query, or fragment. `REASONING_PROVIDER_LABEL` optionally supplies the custom provider's privacy-page label. The selected model must be nonempty, have no outer whitespace, and contain no line breaks. Keep the actual service identity accurate.

Dialogue uses `/chat/completions` with JSON-object output. Scoring requests strict JSON-schema output for the two named Groq GPT-OSS models in the code and JSON-object output otherwise. Catalog visibility does not prove that a model accepts these requests or returns valid feedback within the application's deadlines. These defaults are source values, not a fresh verification of vendor offerings.

## Gateway-only settings

| Variable              | Behavior                                                                             |
| --------------------- | ------------------------------------------------------------------------------------ |
| `SPEECH_GATEWAY_PORT` | Preferred listening port; integer `1024..65535`. If absent, use `PORT`, then `3015`. |
| `PORT`                | Hosting-platform port used only when `SPEECH_GATEWAY_PORT` is absent.                |
| `SPEECH_GATEWAY_HOST` | Defaults to `127.0.0.1`. Container deployment sets `0.0.0.0`.                        |

Gateway startup also requires `GNANI_SPEECH_API_KEY`, `NEXT_PUBLIC_SUPABASE_URL`, `SUPABASE_SECRET_KEY`, `APP_ORIGIN`, and a valid reasoning configuration. The Supabase URL must be HTTPS. The process does not read the web application's `VOICE_RUNTIME_ENABLED` flag; that flag blocks grant creation in the web application. Stopping existing live calls is a separate operational action.

## Deployment split

| Process | Settings to supply                                                                                            |
| ------- | ------------------------------------------------------------------------------------------------------------- |
| Web     | All web settings above, plus the selected reasoning provider/key/model settings                               |
| Gateway | Exact `APP_ORIGIN`, Supabase URL and secret, Gnani key, reasoning selection/key/model, host and platform port |

The [Render descriptor](../render.yaml) contains the core gateway variables with Groq selected. It is not a complete environment manifest for every alternative provider. The [Vercel descriptor](../vercel.json) selects Next.js and a recovery schedule; it does not contain web credentials.

The local Next.js process reads `.env.local`; `gateway:start` uses Node's environment-file option. Hosted processes receive their own environment settings. Changes to public build-time settings require a new web build.

## Safe checks

```sh
npm run config:check
npm run speech:check
npm run reasoning:check
```

These commands load local configuration and report status without printing private values or making provider requests. `config:check` deliberately reports `releaseReady: false`, even when its configuration checks pass. It does not fully validate every runtime flag, gateway route, database definition, provider entitlement, or live session; inspect those separately.

See [Development](development.md#commands) for the separate opt-in network probes and [Validation](validation.md) for acceptance.
