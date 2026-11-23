# Trinity — Principles & Philosophy

**Status: SETTLED, living.** These principles govern every design
decision in this repository. A principle stated here changes only on a
strong case argued on evidence. This document is the proof of what the
project is trying to create; the architecture and the component designs
are answerable to it.

Trinity is, at the root, a living document against the grievances of one
complete technology journey: every named component answers a named
grievance — monitoring that had to be done by hand, configuration drift,
repetitive tasks absorbed by a person, failures a person had to catch.
The design rule follows from that origin: **move everything loadbearing
to the core**, solved once, in software, at the layer where the pain
lives — built in, not bolted on.

1. **Source is the truth.** If they can read it and build it, they can
   trust it. Don't trust the system — trust the source. Total openness:
   the full SoC firmware and boot specifications are in the open along
   with everything else, or nothing is.
2. **Openness is ownership.** The system always belongs to the user. No
   telemetry, no central access, no master keys. Granting privileged
   access to any outside entity is designed to be technically
   impossible — "we can't do it," not "we won't do it."
3. **Permissive over copyleft.** Purging Light Tech's own code is MIT.
   Copyleft code is barred from the loadbearing core (documented
   exceptions: libnl, mdBook), because forcing a license choice on the
   user contradicts what open source means here.
4. **The core carries the load.** If a function can't be handled in the
   core, it is not loadbearing — or it absolutely shouldn't be. The user
   layer holds nothing structural; userland is whatever the user designs.
5. **No single point of failure.** Every layer is designed so one
   failure is a degradation, not a collapse — including the failure of
   any single agent, planner, or node.
6. **Built-in failure modes.** When the system fails, there is a
   designed degraded/recovery state — never a silent halt, and the owner
   is notified over a trusted path. Systems are designed to survive
   their operator at their worst.
7. **Auditability.** If it can't be audited as truth, it can't be
   trusted. Every change is documented with why it was made, how it
   improves on what was there, and the history of the decision itself.
8. **Data is the invariant.** The data layer is ground truth — visible,
   reachable, and user-controlled at all layers. Mechanisms are
   replaceable because the data is not. The only other stable things are
   the API and the ABI.
9. **Design for the least capable, scale up.** The floor is the industry
   standard plus a little more; scaling adds capacity without changing
   shape. The same system spans a set-top media box to a portable
   cluster — no stripped editions at the small end.
10. **Replaceability all the way down.** Every piece is designed to be
    replaced, so the system can grow without accumulating parts that no
    longer have a reason to exist. Once a replacement is verified, the
    old is retired just as easily.
11. **Concurrency.** Concurrency means maximizing the resources at
    hand: not scaling to bigger machines, but wasting nothing on the
    one you have — every core, every cycle accounted for.
12. **Safety.** Safety is built by construction: pieces are designed so
    illegal or undesirable states simply can't be represented — if the
    bad state can't exist, no rule is needed against it. The security
    model follows the same honesty: never claim unbreachable — lay out
    the reasons breaching is unprofitable, so it costs more to breach
    than the breach is worth.
13. **Modularity.** Homogeneous and modular at once: the system speaks
    one design language — uniform principles, contracts, and
    interfaces — while every component is a replaceable module.
    Extending functionality means designing, building, and installing a
    new module suited to your workflow, without breaking the
    homogeneous whole. Homogeneity in the contract, modularity in the
    parts.
14. **User experience.** Never regress the user experience: system
    changes must enhance it or at least keep it at the norm — UX
    regressions are never acceptable. "We do not break userspace" is
    not just about the kernel ABI; the user's workflow is a contract.
    Security, modularity, and concurrency never get to bill the user in
    broken workflows.
15. **TOML source for every component.** Every component built by us
    for the system carries a TOML source document: the hand-edited TOML
    is that component's single source of truth, and everything else —
    configuration, renderings, the build itself — is generated from it,
    never hand-edited into disagreement. Even the Fangs built for Ymir
    depend on the same TOML as Ymir. If everything has a TOML source,
    the system is rebuilt for a new machine from source — compatibility
    lives in the source layer, not the binary layer.
