# Trinity

**Don't trust my claims about the system — examine it yourself.**

Trinity is a unified platform under design at Purging Light Tech: a
reimagined operating system (a minimal microkernel, with Hel, Fangs, and
the Chains — the meta-libOS layer), the MKUltra programming language
(Maia, compiled; Koda, interpreted), and Mimir, its native package
manager — co-designed with Ymir, a RISC-V system-on-chip.

This repository is the design, published while it is being designed.
These are living documents, updated weekly as the design moves. Nothing
here is a release, a promise of availability, or a claim that any
component is built. Every document carries its own status on its face;
where the design is open, the documents say so.

## Start here

- [Principles & philosophy](PRINCIPLES.md) — the guiding principles
  every design in this repository is answerable to.
- [Architecture design](ARCHITECTURE.md) — the whole system in one
  document: principles, core shape, packaging, AI policy, compatibility,
  hardware, governance, and an honest list of what is still open.

## Document status

| Document | Status |
|---|---|
| [Principles & philosophy](PRINCIPLES.md) | Settled, living |
| [Architecture design](ARCHITECTURE.md) | Draft — 2026-11-22, living |
| [Transport Chain](docs/transport-chain-2026-11-14.md) | Endorsed design |
| [Repetition Chain — recurring updates test](docs/repetition-chain-updates-test-2026-11-14.md) | Filed test candidate |
| [AI policy chain](docs/ai-policy-chain-candidate-2026-11-14.md) | Candidate, with 2026-11-20 guardrails addendum |
| [Agent authority bounds](docs/agent-authority-bounds-2026-11-11.md) | Candidate principle |
| [Certificate trust pipeline](docs/certificate-trust-pipeline-2026-11-11.md) | Candidate principle |
| [System freeze feature](docs/system-freeze-feature-2026-11-12.md) | Candidate feature |
| [Filesystem architecture — study primer](docs/filesystem-architecture-study-primer-2026-11-11.md) | Study |
| [BFS study addendum](docs/filesystem-bfs-study-addendum-2026-11-11.md) | Study addendum |
| [Filesystem — HAMMER2 rebuild seed](docs/filesystem-cluster-rebuild-2026-11-11.md) | Open design seed |
| [MKUltra — Maia grammar (EBNF)](docs/mkultra-grammar-ebnf.md) | Draft v0.1.1 |
| [MKUltra — error handling model](docs/mkultra-error-handling.md) | Draft v0.1 — error-union decision locked 2026-09-24 |
| [Self-repairing OS — design brief](docs/self-repair-design-brief-2026-10-15.md) | Concept brief |
| [Project website concept](docs/website-concept-2026-11-18.md) | Concept, held |

**Open, stated openly:** the filesystem design itself, the unified IPC
chapter, the module boundary contract, and the naming of the SystemAI
are not settled. They will appear here when they are.

## Principles, in short

- Source is the truth. If you can read it and build it, you can trust it.
- Everything is open, or nothing is — including firmware and boot specs.
- Permissive licensing: MIT. Credit to Purging Light Tech, greppable in
  the source (or, for binary-only releases, in the documentation), is
  the only condition that matters to us.
- Replaceability all the way down: every piece is designed to be
  replaced, and verified replacements retire the old.
- If the whole system isn't green, it doesn't get released.

## Updates

This repository is a rendering of the working design files, regenerated
weekly. The working files are the source; if the two ever disagree, the
design record — not this page — is where the truth is being decided.

## License

MIT — see [LICENSE](LICENSE).
