# Development

[Documentation](index.md) · [Configuration](configuration.md) · [Contributing](../CONTRIBUTING.md)

## Prerequisites

The package requires Node.js `>=24 <25`. Use npm and the committed `package-lock.json` with `npm ci`; the repository has no monorepo workspace or separate gateway package. Next.js 16.4.3 and React 19.3.1 are pinned in the manifest. TypeScript, Tailwind CSS, Zod, Supabase clients, and `ws` supply the other main application building blocks.

Before changing framework behavior, read the relevant guide in the installed `node_modules/next/dist/docs/` tree. Verify commands against [package.json](../package.json) rather than assuming a different Next.js version's conventions.

## Install and configure

```sh
npm ci
```

Copy [.env.example](../.env.example) to `.env.local` only when that local file does not already exist. Populate credentials privately. Follow [Configuration](configuration.md) for every supported setting and the web/gateway split.

Public pages do not require a functioning voice account. Protected practice needs Supabase configuration and the database definitions; live speech additionally needs the gateway, Gnani, reasoning, and an explicitly enabled runtime.

For a new database, an authorized operator must apply all seven files in [supabase/migrations](../supabase/migrations/) in filename order, expose the `samvaad` schema through the Supabase Data API, and verify its permissions. There is no repository command that provisions a Supabase project or automatically applies these files. Existing databases require a comparison against recorded migration history; never rerun the full series blindly. See [Database](database.md).

Match Supabase's email-confirmation setting to `AUTH_EMAIL_VERIFICATION_REQUIRED`. The supplied value `false` requires automatic confirmation (`mailer_autoconfirm=true`) and disables application email recovery. Configure the application's origin and any authentication callback allowlist to match the intended web address. This guide does not authorize changes to a shared account or database.

## Run the web application

```sh
npm run dev
```

The script starts Webpack development mode at [http://127.0.0.1:3013](http://127.0.0.1:3013). Use that exact origin for `APP_ORIGIN`; `localhost` is a different origin from `127.0.0.1`.

## Run the voice service

In another terminal at the repository root:

```sh
npm run gateway:build
npm run gateway:start
```

The gateway compiles to `.gateway/` and loads `.env.local` through Node's environment-file option. It binds to `127.0.0.1:3015` by default. It is a separate Node process, not a Next.js route. Its HTTP liveness endpoint is `/health`; its WebSocket endpoint is `/speech`.

Keep `VOICE_RUNTIME_ENABLED=false` until the target database, gateway, credentials, and test scope are ready. Local checks of settings do not verify provider availability or release acceptance.

## Review screens without live services

For isolated local UI review, set `UI_PREVIEW_ENABLED=true` and restart the web process. Open `/preview?screen=setup`. Other screen values are `check`, `live`, `processing`, `report`, `history`, `account`, and `auth`. `state=empty` shows empty history; `state=failed` or `state=expired` shows processing failures.

Preview is visibly synthetic, does not establish voice connectivity, and never substitutes for real product API responses. Its setup remains a legacy group-discussion sample. Set the flag back to `false` before release; the route returns 404 when disabled.

## Commands

| Command                             | Purpose                                                        | External service use                                          |
| ----------------------------------- | -------------------------------------------------------------- | ------------------------------------------------------------- |
| `npm run dev`                       | Local web development on port 3013                             | Service calls when users exercise configured flows            |
| `npm run gateway:build`             | Compile the standalone gateway                                 | None                                                          |
| `npm run gateway:start`             | Run the gateway with local environment                         | Service calls when a voice grant is consumed                  |
| `npm run build`                     | Build the web application with Webpack                         | Not a live voice acceptance test                              |
| `npm run start -- --port 3014`      | Serve an already built web application locally                 | Service calls when users exercise configured flows            |
| `npm run check`                     | Format check, documentation links, lint, types, tests          | Synthetic tests; no intentional live provider requests        |
| `npm run format`                    | Rewrite formatting in the configured source paths              | None; modifies local files                                    |
| `npm run docs:check`                | Check local Markdown file links and headings                   | None; does not verify external URLs or render Mermaid         |
| `npm run config:check`              | Inspect settings and safeguard values without printing secrets | None                                                          |
| `npm run speech:check`              | Inspect speech credential presence and prerequisites           | None                                                          |
| `npm run reasoning:check`           | Inspect selected reasoning configuration                       | None                                                          |
| `npm run reasoning:check -- --live` | Request the selected provider's model catalog                  | Network request; no prompt or inference                       |
| `npm run speech:probe`              | Print instructions for the opt-in speech probe                 | None                                                          |
| `npm run speech:probe -- --live`    | One synthetic REST speech synthesis/recognition round trip     | Consumes provider usage; requires explicit test authorization |

The local production `start` script binds to loopback. Use a separate port to preserve an existing development server. Do not run live probes against client accounts without the required approval.

## Navigate the code

UI components are under `src/components`; route wrappers are under `src/app`. Domain rules are separate from transport. Browser requests use `src/lib/client`; privileged database and scoring work use `src/lib/server`; persistent voice behavior uses `src/lib/voice` and `gateway/main.ts`.

`practice-contracts.ts` validates setup options, while `product-contracts.ts` validates session/history/report responses. Domain policy, gateway dialogue policy, UI descriptions, and SQL constraints are related but separate representations: inspect all affected representations before changing a practice rule. Existing inconsistencies are listed in [Validation](validation.md).

There is no staff/admin site, payment subsystem, upload service, or separate email worker in this repository. Email-based verification/recovery is conditional Supabase functionality; it is not an independently hosted mail service here.

## First configured round

1. Verify local liveness at `/api/health` and the gateway's `/health`.
2. Create or sign in to an authorized synthetic test account.
3. Open Practice, select an available setup, and complete the microphone test.
4. Start one round and verify that the timer begins only once the connection is ready.
5. End the round, open its report, inspect eight dimensions and three actions, and locate a quoted passage in the transcript.
6. Return through History and confirm the same owned result is available.

Follow the full [validation checklist](validation.md) before treating that result as acceptance for any real student rollout.
