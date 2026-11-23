# Trinity — Architecture Design

**Status: DRAFT — 2026-11-22.** Prepared at Chris's direction as a single public-facing architecture document, drafted before any decision to publish the working design files. Nothing in this document is a release, a promise of availability, or a claim that any component is built. Sections carry their own status; where the design is open, this document says so on its face.

**License:** MIT (Purging Light Tech's standing rule for its own work). Credit condition: a greppable Purging Light Tech shoutout in the source, or in the documentation for a binary-only release.

---

## 1. What Trinity is

Trinity is a unified computing platform designed from the core outward: an operating system, its native language, and its package manager designed as one system rather than assembled from parts that were never designed for each other. It is the software foundation of Purging Light Tech, co-designed with its own RISC-V hardware (the Ymir SoC).

Trinity is, at the root, a living document against the grievances of one complete technology journey: every named component answers a named grievance — monitoring that had to be done by hand, configuration drift, repetitive tasks absorbed by a person, failures a person had to catch. The design rule follows from that origin: **move everything loadbearing to the core**, solved once, in software, at the layer where the pain lives — built in, not bolted on.

## 2. Principles

These govern every decision below. A principle stated here changes only on a strong case argued on evidence.

1. **Source is the truth.** If they can read it and build it, they can trust it. Don't trust the system — trust the source. Total openness: the full SoC firmware and boot specifications are in the open along with everything else, or nothing is.
2. **Openness is ownership.** The system always belongs to the user. No telemetry, no central access, no master keys. Granting privileged access to any outside entity is designed to be technically impossible — "we can't do it," not "we won't do it."
3. **Permissive over copyleft.** Purging Light Tech's own code is MIT. Copyleft code is barred from the loadbearing core (documented exceptions: libnl, mdBook), because forcing a license choice on the user contradicts what open source means here.
4. **The core carries the load.** If a function can't be handled in the core, it is not loadbearing — or it absolutely shouldn't be. The user layer holds nothing structural; userland is whatever the user designs.
5. **No single point of failure.** Every layer is designed so one failure is a degradation, not a collapse — including the failure of any single agent, planner, or node.
6. **Built-in failure modes.** When the system fails, there is a designed degraded/recovery state — never a silent halt, and the owner is notified over a trusted path. Systems are designed to survive their operator at their worst.
7. **Auditability.** If it can't be audited as truth, it can't be trusted. Every change is documented with why it was made, how it improves on what was there, and the history of the decision itself.
8. **Data is the invariant.** The data layer is ground truth — visible, reachable, and user-controlled at all layers. Mechanisms are replaceable because the data is not. The only other stable things are the API and the ABI.
9. **Design for the least capable, scale up.** The floor is the industry standard plus a little more; scaling adds capacity without changing shape. The same system spans a set-top media box to a portable cluster — no stripped editions at the small end.
10. **Replaceability all the way down.** Every piece is designed to be replaced, so the system can grow without accumulating parts that no longer have a reason to exist. Once a replacement is verified, the old is retired just as easily.

## 3. Shape of the system

Trinity does not have a kernel in the conventional monolithic sense. Its core is a set of named, anchored components — a component that can't carry a name isn't loadbearing.

- **Garmr** — a minimal microkernel (12 syscalls). Deliberately small: the less that runs privileged, the less that can be maliciously edited into unauthorized access.
- **Hel** — security logging and forensics. Hel keeps the system's only track record, and it stores **metadata, not content**, so the audit trail itself is not an attack surface. Post-quantum-only cryptography throughout.
- **Fangs** — one Fang per hardware target. Each Fang serves only its own hardware's API/ABI. A Fang is how the system meets a specific machine; swapping hardware means installing the new SoC's Fangs, booting, verifying, and retiring the unneeded Fangs last.
- **Chains — the meta-libOS layer.** The Chains *are* the meta-libOS: a layer of specialized modules, one job each, communicating over a capability token chain (unforgeable tokens; a single auditable path for every privileged action). The meta-libOS holds no operating-system functions of its own — it sends signals to Fangs and inherits nothing. Collected libOS designs were studied for a decade; the completion is a layer that coordinates without absorbing. Named Chains so far: the **Transport Chain** (capability-mediated movement of data and files), the **Repetition Chain** (recurring work, with recurring software updates as its filed test case), and the **AI Chain** (the boundary every AI crosses to act on the system — see §6).
- **The shell** — quite literally a shell: it sits on the meta-libOS and provides an interface for modules, nothing more. The desktop is a composition of swappable modules. Every front-end (GUI, CLI, script) must expose the full capability of what is underneath, in its own idiom — no power-stripped editions. A specialized front-end is a door if the user can step through it to the whole system, a cage if it only admits them to a subset; Trinity builds doors.

**POSIX is not the constitution.** The foundational domains all tie into POSIX, and POSIX is written in C — adopting it would import C's assumption set as constitutional law. POSIX-shaped software is a guest (see the Compatibility Chain, §7), never the foundation.

**Runtime immutability.** A running module cannot be created, destroyed, or modified at runtime; the running set is fixed. Modules extend by loading verified modules. Updates are stop-and-swap through the package manager, never patch-in-place.

**Correctness by construction.** Pieces are designed so illegal or undesirable states can't be represented. If the bad state can't exist, no rule is needed against it.

## 4. Software layout and packaging

- **Mimir** — Trinity's native package manager (successor to the norn design work; norn is its working example, not shared code).
- **Versioned addresses.** Each application and version has its own address under `/usr/apps`, so libraries are shared by address: a consumer points at the exact version it needs. The retention window is the current version plus two back — supporting LTS-pinned and rolling consumers at once. Pruning rides the install/verify motion (new version verified green → the window shifts → the oldest retires); it is never a user chore.
- **Configuration is TOML only** — the single hand-edited source. Renderings are generated from it, never hand-edited. Machine-readable manifests (where structured formats serve machines) are a separate layer from the human-edited configuration, and the two are not confused.
- **Release gate.** If the whole system isn't green, it doesn't get released. No partial releases; every layer verifies first. Release notes break down by section, then features and fixes, each entry listing the patch(es) that enabled it — plus a current-limitations section. Successes are celebrated and shortcomings admitted in the same document.

## 5. The language — MKUltra

Trinity's native language is MKUltra, dual-mode by design, on the premise that the right language for the right situation should not force a choice between ergonomics and performance:

- **Maia** — compiled mode, Pascal-family syntax (chosen at the root: Pascal predates the compiled/scripting split, descending from the same ALGOL line the scripting languages' ergonomics came from).
- **Koda** — interpreted mode: shell, web framework, and built-in version control.

Configuration for both modes is TOML only. The language is designed to be able to morph — the core language treated as a package of itself — under one condition: the standard library stays small, and everything else is modular. Modifications are possible at source level only, auditable alongside the affected code; there are no outside mods.

## 6. AI on Trinity

AI is a citizen of the platform, never its sovereign.

- **The approval gate.** Every AI action that acts against the system passes one gate: **propose → sign off → implement**. No exceptions, including for the platform's own AI.
- **The SystemAI** (unnamed — it will be named when it is understood well enough to carry one) monitors, predicts, and proposes fixes. It may recommend quarantine to Hel only after permission; Hel quarantines. It does not use the Chain — native AI is guardrailed by construction; guest AI is guardrailed by the AI Chain.
- **Guardrails where people are stupidest.** The AI Chain guards the AI's path to the system; guardrails must also sit at points of predictable *user* failure. A credential offered into a conversational surface must have nowhere to go — no field that accepts it, no path by which a secret becomes content. This is structural, never educational: people cannot reliably tell a chatbot from a person, and no amount of training changes that perception, because the interface's social shape produces the confusion.
- **Admission rule (interim).** Until Chain-level enforcement exists that prevents AIs from storing secure data outside a secure store, Trinity admits only AIs that themselves demonstrably enforce the boundaries. Vendor claims do not qualify.
- **No queen.** A single presiding agent is a single point of failure with a title. Where independent verification is needed, the design direction is a **committee**: a request is fulfilled by five independent runs — through five *different models*, so their failure modes decorrelate — and reconciled mechanically (vote / diff / median) or by the user, never by a presiding agent's judgment. Divergence between the five is itself the signal: where they split is where the uncertainty lives. Every seat's model carries its own license and provenance grade before the seat can be filled; an honestly empty seat beats an unauditable one. *(Status: proposal under development, first written 2026-11-21/22.)*

## 7. Compatibility

Legacy software is a **guest**, never a foundation. The Compatibility Chain hosts foreign ABIs — the standing example is the legacy 32-bit x86 ABI, carried in a Chain instead of ambient multilib: native on x86 hosts, translation (Box64-class) on Ymir. The compatibility backstop is scaffolding, not foundation: it exists only until the native ecosystem catches up, and its removal is part of the design from day one.

## 8. Hardware and form factors

Three form factors: **Desktop, Laptop, and the portable cluster** (the Cluster F model line).

- **Ymir** — Purging Light Tech's own RISC-V SoC, co-designed with Trinity: every contract is written as chip-provides / Trinity-assumes / the contract between, and a change on either side forces a review of the other. Status: Part I closed 2026-11-06 (testbench green in simulation); Part II in simulation against the RISC-V manuals. Claims about the hardware stand on the simulations — the sims are the floor of proof.
- **Cluster F** — every node is a complete standalone system, and all nodes are equal: there are no master and slave nodes, so one failure is a degradation, not a collapse. The internal fabric is always on because that is how nodes compose — but if the fabric dies, a single node runs just the same. Composition adds; it never becomes a prerequisite. The head node can be switched at boot, including a demonstration head that shows the cluster's strength in a form most people can understand. The single board was proven first, so cluster failures are attributable to the composition layer by elimination.

## 9. Governance

Purging Light Tech has no BDFL — that is against everything the project believes in. The model is the Linux kernel's shape as it actually works: subsystem maintainers own their subsystems; the final say at the top is a narrow certification gate (certifying releases green), not ownership of every decision. Authorship and provenance are kept in separate ledgers: AUTHORS records who had a hand in the source; ROOTS records where the ideas came from. Beyond the authors page, the founder's name appears nowhere on the project — it belongs to Purging Light Tech.

Support follows the Arch model: community support built on exhaustive documentation. Monetization, where it exists at all, is in custom OS builds and complete hardware systems — never in the code, and there are no support contracts. There is no sponsor.

## 10. Status of this design

| Area | Status |
|---|---|
| Principles (§2) | Settled — change-controlled, revisable only on a strong case |
| Core shape: Garmr, Hel, Fangs, Chains (= the meta-libOS layer) (§3) | Settled in structure; component detail in progress |
| Transport Chain | Endorsed — filed design |
| Repetition Chain | Filed test candidate (recurring software updates) |
| Software layout, Mimir, versioned addresses (§4) | Settled in design |
| MKUltra: Maia / Koda (§5) | Framework settled; language design paused deliberately after framework |
| AI approval gate, guardrails, admission rule (§6) | Filed candidate with addendum (2026-11-20) |
| Agent committee (§6) | Proposal — 2026-11-21/22, under development |
| Compatibility Chain (§7) | Candidate — 2026-11-18 |
| Ymir co-design (§8) | Part I closed; Part II in simulation |
| Cluster F (§8) | Settled in structure; external uplink fork open |
| Filesystem | **Open** — seed filed (a HAMMER2 rebuild-and-modify extraction); nothing decided |
| IPC model | **Open** — pieces exist across the design; no unified chapter yet |
| Module boundary contract | **Open** |
| SystemAI definition and naming | **Open** — comprehension precedes the name |

Open questions are listed, not papered over. While new data still exposes a gap at least once a day, the design is not finished — and this document will say so until it is.

---

*Draft prepared 2026-11-22. Working design files remain the source; this document is a rendering of their current state for readers, and will be regenerated as the design moves — never hand-edited into disagreement with them.*
