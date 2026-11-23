# Trinity — System Freeze Feature (candidate, 2026-11-12)

**Status: candidate — existence CONFIRMED by owner
2026-11-12 ("it's just a feature that should be
provided"); shape explicitly undecided, reserved for a
later design discussion.** Not promoted. Origin: Chris's
rejection of RHEL image-mode
sealed images (integrity-by-immutability as a foundation)
plus his observation that immutable systems "have enough
of a following" that Trinity should think about offering
the property without being built on it: "while we don't
have to make it immutable we can provide a freeze feature
that prevents system changes."

## The distinction this candidate protects

- **Foundation immutability (rejected):** the system is
  trustworthy because it cannot change; change requires
  re-imaging; the owner's daily work is subordinated to
  the seal. Rejected 2026-11-12 on both principle and
  workflow grounds.
- **Freeze as a feature (this candidate):** immutability
  becomes one *state* the living system can be put into —
  by its owner, deliberately, revocably — the same way
  32-bit support and GPL code are gated opt-ins rather
  than defaults or bans. The platform ships the mechanism;
  the owner supplies the policy.

## Sketch (all of it open)

- **What freeze would gate:** system changes through the
  Mimir lifecycle (install/enable/disable/purge) and
  module replacement. Frozen = those operations error out
  with the freeze named as the cause (error instead of
  contamination, applied to the owner's own prior
  decision).
- **What freeze must never touch:** the data layer. Data
  stays live and user-controlled in every state
  (data-layer capstone); freezing the data with the
  system would repeat the /etc–/var failure of the
  image-mode seal.
- **Who holds the switch:** the owner, with their own
  keys, locally. Freeze/unfreeze events are recorded in
  the auditable record (Hel) like any other consequential
  transition. No remote party — vendor, employer,
  platform — can set or lift a freeze on a machine they
  don't own; a fleet owner freezing their own fleet is
  the same owner's right at scale, exercised through
  their own management, never through Trinity itself.
- **Freeze granularity:** whole-system vs per-collection
  (e.g., freeze core, leave /usr/apps live) — OPEN.
- **Freeze and the operator at their worst:** a freeze
  set in a known-good state could serve the
  survive-me-at-my-worst standard (the system refuses
  changes its owner pre-committed to refusing) — and the
  unfreeze path must therefore stay usable at the owner's
  worst while remaining unusable by anyone else. UX
  design question, OPEN — same shape as the approval-UX
  question parked in the agent-authority candidate.

## Open questions (parked, owner's to settle)

1. Scope: whole system, core only, or per-collection?
2. Does freeze block *all* lifecycle operations or only
   changes to already-enabled components (can new
   installs land in a scratch area while frozen)?
3. Unfreeze ceremony: key signature only, or key plus
   delay/confirmation at the owner's option?
4. Interaction with stop-and-swap updates: does a
   scheduled update unfreeze, fail loudly, or queue
   behind the freeze? (Default instinct: fail loudly —
   the freeze is the owner's recorded decision and
   automation never outranks the owner.)

Cross-references: agent-authority-bounds-2026-11-11.md
(ceiling/trail/revocation test), the error-instead-of-
contamination rule (2026-11-12), norn's opt-in repo
mechanism pattern, MEMORY.md "Immutability rejection"
(2026-11-12).
