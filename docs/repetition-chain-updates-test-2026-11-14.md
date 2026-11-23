# Repetition Chain test candidate — recurring software updates (2026-11-14)

Status: Chris identified recurring software updates as the test candidate for the repetition Chain ("my big headache right now"). Same session, his words: "I hate cron so I don't use it though I know I should" — the incumbent tool for exactly this job is one he refuses to live with, which is the headache in concrete form. The repetition Chain's dedicated design pass is still owed; this note records the test case and what it must exercise, not a completed design.

## Why this test case
A reminder only proves that a timer fired. Software updates prove the whole repetition mechanism because the recurring action is consequential, state-changing, and failure-prone. If the Chain can make updates boring and auditable, ordinary recurring tasks are covered by the same machinery.

## What the test must exercise
- **Recurrence:** check cadence, missed-run catch-up, retry/backoff, and no duplicate concurrent runs.
- **Update transaction:** discover → fetch → verify → stage → apply at the user's restart → verify → commit, or roll back cleanly. No half-applied state. (The safe boundary is the user's restart — settled 2026-11-23; see the settled posture below.)
- **Module lifecycle contact:** installed vs enabled are separate; running modules are immutable at runtime. A staged replacement waits for the user's close-and-restart; at restart the user's setting governs (notify-and-ask, or auto-move). Nothing is swapped under a running program.
- **Policy contact:** active-work protection, resource thrift (CPU/bandwidth windows), deferral, and which update classes — if any — require explicit user approval before applying.
- **Audit:** every scheduled check, source, version delta, deferral, failure, and result is enumerable through the Chain. A failed or skipped run stays visible; silence is not a result.
- **Failure behaviour:** partial failure rolls back or holds at the last verified state and reports; the recurrence continues from a known state rather than compounding the failure.

## Settled posture (Chris, 2026-11-23)
- **Default posture:** check, stage, and verify in the background; the user keeps running their version until they decide to close and restart it. At restart the user's setting governs — notify that a new version is available and ask whether to use it, or auto-move to the new version on restart. The control belongs to the user.
- **LTS exception:** when a user's LTS version is deprecated, the system securely messages them on the system message bus that the version has been deprecated and that on app restart they will need to be using an updated version — then asks whether they want to retire the old LTS in favor of the new LTS. The decision is the user's.
- **Safe application boundary:** the user's restart. Nothing changes mid-session.

## Open questions for the design pass
- Do security updates get a different approval/deferral class?
- How many prior versions are retained for rollback?
- In a portable cluster, are updates per-node independent, staggered, or orchestrated from one node?
