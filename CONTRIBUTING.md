# Contributing

[Project README](README.md) · [Documentation](documentation/index.md)

## Understand the change

Read [Development](documentation/development.md), [Architecture](documentation/architecture.md), and the subject document for the affected area. Inspect routes, callers, shared validators, database functions, tests, and duplicate policy representations before editing. Preserve existing client work and keep changes focused and reversible.

Describe the intended user outcome and failure behavior before implementation. Obtain the owner's informed approval before deleting features/files, changing authentication/permissions, altering stored records or definitions, making shared-environment changes, replacing major dependencies, deploying, or rewriting Git history. Explain consequences in plain language and retain a recovery plan.

## Preserve the trust boundaries

- Derive identity from the authenticated server session and check ownership on every private read/write.
- Parse untrusted requests and generated output with the established Zod/domain schemas.
- Keep provider and privileged database credentials server-side. Never accept a browser transcript or score as evidence.
- Preserve immutable provider finals, single-use grant rules, bounded usage, idempotent replay, and leased processing.
- Render generated content as text. Treat speech/transcripts as data rather than instructions.
- Test with synthetic accounts/content and keep personal information and secrets out of fixtures, commits, screenshots, and logs.

## Verify behavior

```sh
npm run check
npm run gateway:build
npm run build
```

Add meaningful regression coverage for changed behavior. Use focused tests during development, then the repository checks appropriate to the final change. UI work also needs mobile/desktop, keyboard, and affected accessibility-state review. See [Validation](documentation/validation.md) for live acceptance; passing tests do not certify external services or student outcomes.

For documentation-only changes, verify links, owning source, public file scope, and examples. State exactly which checks ran and disclose unavailable checks. Do not report earlier test results as a fresh run.

## Review and delivery

Keep the description centered on the concrete problem, resulting behavior, verification, and remaining risk. Stage named paths and inspect the intended file list. Never include local environment files, credentials, student evidence, generated caches, or private working notes. New public Markdown belongs in the curated documentation paths allowed by `.gitignore`.

Update the [owning document](documentation/index.md#for-developers-and-operators) when a contract, setting, or procedure changes. Link to the source and owning page rather than creating another specification. Keep release evidence distinct from implemented behavior.

Do not push, deploy, rewrite history, or run remote database commands without the scope authorization those actions require. A code review or local check is not authorization to change a live service.
