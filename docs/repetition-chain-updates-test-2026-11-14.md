# Repetition Chain test candidate — recurring software updates (2026-11-14)

Status: Chris identified recurring software updates as the test candidate for the repetition Chain ("my big headache right now"). Same session, his words: "I hate cron so I don't use it though I know I should" — the incumbent tool for exactly this job is one he refuses to live with, which is the headache in concrete form. The repetition Chain's dedicated design pass is still owed; this note records the test case and what it must exercise, not a completed design.

## Why this test case
A reminder only proves that a timer fired. Software updates prove the whole repetition mechanism because the recurring action is consequential, state-changing, and failure-prone. If the Chain can make updates boring and auditable, ordinary recurring tasks are covered by the same machinery.

## What the test must exercise
- **Recurrence:** check cadence, missed-run catch-up, retry/backoff, and no duplicate concurrent runs.
- **Update transaction:** discover → fetch → verify → stage → apply at a safe boundary → verify → commit, or roll back cleanly. No half-applied state.
- **Module lifecycle contact:** installed vs enabled are separate; running modules are immutable at runtime. A replacement must wait for a disable/reboot/session boundary or follow the established conflict path.
- **Policy contact:** active-work protection, resource thrift (CPU/bandwidth windows), deferral, and which update classes — if any — require explicit user approval before applying.
- **Audit:** every scheduled check, source, version delta, deferral, failure, and result is enumerable through the Chain. A failed or skipped run stays visible; silence is not a result.
- **Failure behaviour:** partial failure rolls back or holds at the last verified state and reports; the recurrence continues from a known state rather than compounding the failure.

## Open questions for the design pass
- Default posture: check-and-stage with user-applied, check-and-apply within policy, or class-dependent?
- Do security updates get a different approval/deferral class?
- What is the safe application boundary for each module class — immediate, session restart, full reboot?
- How many prior versions are retained for rollback?
- In a portable cluster, are updates per-node independent, staggered, or orchestrated from one node?
