# Self-repairing OS — design brief (2026-10-15)

Status: **concept the user has been toying with** (his words, 2026-10-15). Not a locked Trinity spec. Items below are kept at the status he gave them; assistant-proposed plumbing is labeled as such and is not recorded as adopted unless he said so.

## The concept (user, 2026-10-15)

An OS that can find flaws within its own system and, after working with the sys admin or whoever manages the system, come up with a plan for remedying the flaw and then implement it.

Loop as he stated it: detect the flaw → plan the remedy *with* the admin → implement. The human gate was in his original statement, not added later.

## Decisions the user stated

- **Full reach, not config-only.** "I was going with the full reach in my design — a simple configuration level repair is good for normal users but I want a tool for me." Two authority tiers follow from that: bounded configuration repair for normal users; full-reach (code-level) repair for the owner/admin.
- **Owner reads every patch.** "Every patch should have a 'signed off by' line that you must change to an authorized signer which should be able to be confirmed securely somehow before the patch can be applied." Review is not optional process; it is the control that came with setting up the system.
- **Sovereignty rule (his full-reach statement of credo point 2).** "The system always belongs to the user and never should the terms of the system be dictated by any force outside the user — that's what open source means to me." Applied inward as well as outward: even the system's own automation does not outrank the user.

## Assistant-proposed plumbing (proposed in chat 2026-10-15; user called it "an elegant solution" — praise, not a recorded adoption)

- The plan is a diff: the proposal the admin approves is the patch itself, reviewable line by line, not a prose summary.
- Sign-off binds to content: the authorized signature is computed over the hash of the exact diff reviewed, so any change after signing fails verification. Unsigned placeholder trailers are rejected by the apply pipeline.
- Signer confirmation: Git-style SSH signing with an allowed-signers file was offered as existing tooling; a FIDO2 hardware key requiring a physical touch was offered as the stronger fit (signature = an act by a present human, not a copyable file).
- Bootstrap: the first authorized signer is enrolled at system setup by the owner; after that, the signer list can only be changed by a signature from someone already on it. Adding a signer is the most sensitive patch class and goes through the same gate it controls.
- Revocation: enroll an offline recovery signer at setup whose only job is striking a compromised signer.
- Build and activation machinery: clean-room build with tests before activation; atomic install into an alternate slot with snapshot/rollback on failed health verification; provenance (system-authored vs upstream vs admin-authored) recorded in a log the repair engine itself cannot edit.
- Separation wall: the OS may write the patch; it must never be the entity that signs it. Proposer ≠ approver, as architecture, not workflow habit.

## Open forks — not settled by the user

1. **Where the flaw-finder lives** (asked in chat, unanswered): part of the OS proper, or a small separate verifiable layer *under* it, so the doctor is not sick with the same disease when the patient is the OS itself? Includes the contaminated-pool problem one level down: if the flaw is in the detector or its baseline of "normal," the diagnosis cannot be trusted — the detector needs integrity guarantees below it.
2. **Runtime immutability tension (flagged by this brief, not raised in chat):** Trinity's standing principle is runtime immutability — updates are stop-and-swap through Mimir, never patch-in-place. A self-repair engine must reconcile with that: the natural reconciliation is that self-repair *produces* a verified package and the swap still goes through Mimir's stop-and-swap, but the user has not ruled on it.
3. **PQ-only tension (flagged by this brief, not raised in chat):** Trinity crypto is PQ-only, and Hel handles security logging/forensics. An ed25519 signing flow fits the user's current personal Git tooling, but if this design lands inside Trinity proper, the signature scheme and the provenance log have to be reconciled with PQ-only and with Hel's metadata-only audit model (hash + signer + patch ID fit metadata; patch content does not). Not settled.
4. **Remediation supply chain:** source-first stops being a preference here — the OS cannot repair code it can only run as a binary. Whether norn/Mimir is formally designated that supply chain is implied by the chat but not ruled by the user.

## Why this belongs in the Trinity file

It is the "built-in, not bolted on" principle applied to fault management itself, and it externalizes the user's own flaw-disclosure ethic: find the flaw, surface it to the person responsible, agree the fix, implement, verify.
