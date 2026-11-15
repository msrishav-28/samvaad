# Validation and limitations

[Documentation](index.md) · [Development](development.md) · [Operations](operations.md)

Implementation, automated verification, live integration acceptance, and student acceptance are different forms of evidence. This page describes how to collect them; it does not claim that a deployment has passed. Record a release's revision, date, environment, actual commands/results, manual outcomes, and unresolved issues in its review record without exposing credentials or student content.

## Automated checks

Run from a clean checkout with Node 24:

```sh
npm ci
npm run check
npm run gateway:build
npm run build
```

The [CI workflow](../.github/workflows/check.yml) runs these checks on push and pull request. `check` covers Prettier's configured source paths, local Markdown links/headings, ESLint, strict TypeScript, and Vitest. Markdown files are not part of the package's Prettier target list. Documentation link checking does not render Mermaid or validate remote URLs.

| Test area                   | Evidence supplied by the suite                                                                                                   |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| Account and HTTP boundaries | Origin/identity checks, auth flows, safe redirects, body validation and owned operations                                         |
| Session/domain behavior     | Setup limits, creation replay, quotas, history cursors/comparisons and score calculation                                         |
| PostgreSQL definitions      | Embedded role permissions, ordered SQL behavior, immutable finals, worker fencing, transactional completion and bounded attempts |
| Provider isolation          | Mocked provider envelopes, selected credentials, timeouts, response bounds, invalid quotes and no automatic fallback             |
| Voice transport             | Mocked WebSockets, frame pacing, burst bounds, cancellation, empty finals, sequence finalization, storage retry and liveness     |
| Browser components          | Simulated microphone states, mute/controls, resource polling, account forms and preview isolation                                |

Tests use synthetic fixtures and an embedded PostgreSQL engine. They do not contact a remote Supabase instance or certify actual provider speech, host networking, physical microphone behavior, browser visual layout, external retention, or spend. Simulated DOM/canvas behavior is not a screenshot or screen-reader review.

For documentation-only work, check links, source consistency, excluded files, and formatting of the new documents. Do not claim a fresh application test/build unless actually run for that change.

## Live acceptance

Use only an authorized environment, synthetic test content, and approved provider usage. Check every mode/duration combination intended for availability; one successful group discussion does not establish acceptance for the others.

1. **Account access:** create/sign in, sign out, and change a password while signed in. Verify the configured email policy and that forbidden recovery flows remain blocked.
2. **Ownership:** with two isolated accounts, verify that one cannot read or act on the other's session by changing a URL or request ID. Confirm signed-out denial and rejected foreign origins.
3. **Setup and allowance:** verify available options, blocked unavailable choices, duplicate creation replay, maximum duration, daily reset display, one-live-round restriction, and shared allowance rejection.
4. **Real microphone:** test permission denial, hardware silence, late permission after navigating away, successful four-second input, and device release after leaving.
5. **Start:** verify gateway wake behavior, one grant reservation, no duplicate start, and a full timer that begins only after recognition readiness.
6. **Conversation:** hear and distinguish the disclosed participants, verify English-only mode behavior and difficulty, test barge-in, microphone mute and speaker mute. Presentation must protect delivery before questions.
7. **End and final words:** end manually and at the deadline. Compare a known final spoken phrase with stored recognized text. Test speech close to the cutoff, delayed finals, silence, disconnection, and provider/storage interruption. Do not equate the drain marker with proof of completeness.
8. **Visible report:** inspect the actual rendered transcript, eight unique dimension scores, reasons, quoted student evidence, computed total, strongest/improvement moments, and exactly three actions. Unsupported/missing evidence must not produce a fabricated result.
9. **History and recovery:** leave and return during processing; check eventual status and report through History. Exercise transient reads, expired work and exhausted scoring without creating another live run. Measure the actual scheduled-recovery delay.
10. **Repeat:** verify previous actions appear for an owned matching mode/topic, and history compares only compatible settings/rubrics.
11. **Privacy and cost:** verify no raw audio persists in application storage/logs, inspect public assets for private credentials, verify the provider/contact disclosure, establish external retention terms, and measure actual account usage/cost.
12. **Student trial:** observe a real student completing setup, microphone check, practice, report interpretation, and a repeat attempt. Record usability outcomes separately from synthetic acceptance.

For intermittent transport issues, require repeatable complete rounds under representative conditions rather than selecting one successful attempt. Record failures alongside passes.

## Interface and device review

Review desktop and mobile widths, keyboard-only navigation, visible focus, radio/select labels, auth errors, microphone denial/silence, end-dialog focus/Escape, score evidence selection, transcript disclosure, history empty/error states, and private-page access.

Test true 200% zoom, screen-reader announcements, reduced-motion settings, and playback/capture on each proposed browser/device. Safari/mobile and physical microphones require their own results. Preview screens can help inspect states but cannot establish live audio support. Include slow or unstable networks and background-tab behavior.

## Known limits visible in source

| Area                | Current behavior or unresolved limit                                                                                                                                                        |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Voice finalization  | `known-utterances-drained-v1` drains recognized known utterances; there is no provider completeness acknowledgement validated by the application. Final-word capture needs live acceptance. |
| Interrupted voice   | A run is bound to one non-resumable connection. Partial evidence does not guarantee a finalized transcript/report.                                                                          |
| Background recovery | Supplied cron is daily and handles at most two due sessions per invocation. Timely completion after browser closure is not guaranteed.                                                      |
| Accounts            | Sample email ownership is unverified and automated password recovery is disabled. No self-service account deletion exists.                                                                  |
| Retention           | No application retention purge or self-service transcript export/deletion. External provider policies are not established by code.                                                          |
| Budget              | Planned-minute admission is shared across users and is not a monetary cap. Failed reservations still count.                                                                                 |
| Availability        | Practice options derive from configured prerequisites and limits; they do not probe all dependencies or certify each mode.                                                                  |
| UI descriptions     | Group-discussion setup/microphone text still describes an older one-partner arrangement; the live gateway/room use a moderator and two peers.                                               |
| Policy duplication  | Mode rules appear in domain policy and voice dialogue prompts; product descriptions are separate. Changes must keep these representations aligned.                                          |
| Historical SQL      | Legacy voice columns/functions remain in ordered SQL. They do not imply current application endpoints or integrations.                                                                      |
| Observability       | Sanitized console events exist; no central alerting/monitoring service is configured in the repository.                                                                                     |
| Feedback            | Text-based AI coaching can be wrong; recognized text is not proof of acoustic delivery, emotion, pronunciation, or student outcomes.                                                        |

These limitations are disclosed, not silently fixed by this documentation. Changes to account policy, database structure, hosting, recovery cadence, or customer-visible behavior require their own scoped implementation and verification.
