# Trinity — The Skeleton Walk

**Status: APPROVED — 2026-11-23.** A walk through the system as a whole,
following the path a machine takes from power-on to a working shell,
and then one ordinary act traced end to end. The component designs live
in their own documents; this document holds almost no detail of its
own. Its subject is the handoffs — what crosses each boundary, in what
form, and what each layer refuses to pass along. Where the design is
open, the walk says so rather than stepping over the gap.

## How to read this

The component documents describe parts. A system is not its parts; it
is what the parts hand each other. Read this first, and the component
documents afterwards read differently, because you will know where
everything plugs in.

One shape repeats at every level: privilege decreases as you climb.
The bottom knows the least about purpose and holds the most power over
the machine; the top holds no power at all and is entirely the user's.

## The walk — power-on to shell

### 1. Ymir — the silicon

The walk starts in hardware: Ymir, the RISC-V system-on-chip
co-designed with Trinity. The chip and its drivers share one TOML
source — the contract between them is written chip-provides /
Trinity-assumes, from a single hand-edited source, so the two sides of
the boundary cannot drift into disagreement.

Ymir is itself a modular system. An SoC is composed for the task at
hand out of modules — CPUs, GPUs, NPUs (neural processing units), and
the rest — in whatever combination the task actually needs: n of any
module, combined to order. A light build might be a CPU/GPU-only SoC.
A heavier one might carry two CPUs, one NPU, and one GPU. There is no
standard Ymir that every user gets a copy of; there is a composition,
chosen per task.

That modularity is why no part of the chip is allowed to become a
dependency. If the design required a GPU, every composition would be
forced to carry one whether the task called for it or not — the mean
of the general population, imposed in silicon. Instead, a module is
present because the task asked for it, and the software above is
written so the CPU path can always carry the work: an accelerator
speeds a job up, it never becomes the only thing that can do the job.
Customization and non-dependence are the same decision, seen from two
sides.

What crosses upward: capability, honestly reported. Accelerators are
optional with a CPU fallback, never load-bearing. What Ymir refuses:
assumptions about the general population of users — the processor is
designed for the tasks it will actually run.

### 2. The kernel — deliberately forgettable

Above the silicon sits a minimal microkernel: 12 syscalls —
isolation, protection, and scheduling, the irreducible mechanics, and
nothing else. It has no name yet, and that is
deliberate — a name is given when it is earned, and the kernel's job is
to be beneath notice. Everything with a named job lives above it.

What crosses upward: isolated, protected execution and a small, fixed
set of mechanisms. What the kernel refuses: knowledge. It does not know
what an update is, what a file means, or what the user wants. Policy
never lives here, so policy can never be trapped here.

### 3. Hel and the SystemAI — the witnesses

Alongside the kernel, at the level that can see the system from outside
it: Hel, which keeps the security log and the forensic record —
metadata only, never content, so the audit trail is not itself an
attack surface — and the SystemAI, which monitors, predicts, and
proposes. Both hold capability tokens — authority in this system is a
token you can inspect, not a status you inherit — but neither runs on
the Chains. That separation is designed, not incidental: the Chains
are a surface Hel and the SystemAI are supposed to audit, so the
auditors cannot also be tenants of the surface they audit.

What crosses upward: a trustworthy record of what happened, and
proposals. What they refuse: Hel never holds content; the SystemAI
never acts on the system without the approval gate — propose, sign off,
implement.

### 4. Fangs — one per piece of hardware

Each hardware block gets exactly one Fang: the driver that speaks that
hardware's API and ABI, and no other. A Fang is the only thing in the
system allowed to know its hardware's dialect. Swap the SoC and you
install the new Fangs, boot, verify, and clean up the old ones last —
the rest of the system never notices the accent change.

What crosses upward: hardware, presented in the system's one design
language. What a Fang refuses: every other Fang's dialect, and any
knowledge of why the hardware is being used.

### 5. Chains — the meta-libOS layer

Above the Fangs, the system is composed of Chains: specialized modules,
one job each, capability-token mediated, communicating by signals only.
The Chains *are* the meta-libOS layer — there is no second layer
between them and the system. A Chain holds no OS functions of its own;
it inherits nothing; it signals. Named so far: the Transport Chain
(inter-module communication), the Repetition Chain (recurring work,
absorbed by the system instead of by a person), and the AI Chain
(policy enforcement for guest AI).

What crosses upward: composed behavior — many small, replaceable,
single-purpose modules acting as one system. What the Chains refuse:
ambient authority and inheritance. Nothing reaches across a Chain
except through its auditable path.

### 6. The shell — where the user lives

The walk ends where the person begins. The shell is literally a shell
on the meta-libOS; the desktop is a composition of swappable modules.
Userland is whatever the user designs — the core holds everything
structural so that this layer holds nothing structural. Every front-end
exposes the full capability of what is underneath, in its own idiom:
a door you can step through, never a cage.

What crosses upward: nothing. This is the top. The system belongs to
the user, and the walk ends in their hands.

## One act, traced — a recurring update

Boot shows the layers arriving. To watch them *be* a system, follow one
ordinary act: a recurring software update, the Repetition Chain's filed
test case.

1. **Intent.** The update exists because the system's own schedule says
   so — no person remembers, checks, or babysits. The Repetition Chain
   takes the job: discover, fetch, verify, stage, apply at your
   restart, verify again, commit — or roll back cleanly.
2. **Down through the Chains.** The Repetition Chain signals the
   Transport Chain; modules speak only through the auditable path,
   each holding a capability token scoped to exactly this job.
3. **To the hardware.** Fetching reaches a Fang — the network block's
   own Fang, speaking its dialect and no other. Staging writes through
   the storage Fang to a versioned address: the new version lands
   beside the old, never over it.
4. **Verification before commitment.** The staged version is verified
   before anything is eligible to switch. If it fails, nothing has
   changed: the old version still runs, the failure is recorded, and
   the owner is notified over a trusted path. There is no half-updated
   state.
5. **The switch belongs to the user.** Nothing changes under a running
   program — never a patch in place, and never a swap imposed
   mid-session. You keep running your version until you decide to close
   it and restart. At restart, your own setting governs: be notified
   that a new version is available and asked whether to use it, or move
   over automatically on restart. The control is yours. LTS (long-term
   support) versions carry one obligation on the system's side: when
   your version is deprecated, the system securely messages you on the
   system message bus that on your next restart you will need to be on
   an updated version — and asks whether you want to retire the old LTS
   in favor of the new one. Even there, the decision is the user's;
   the system's job is to make sure it is an informed one.
6. **The record.** Hel has metadata for every step: what was fetched,
   what verified, what swapped, when. The SystemAI watched the shape of
   the event, not its content. The kernel, throughout, knew none of
   this — it isolated and scheduled, and that was all.
7. **Back at the shell.** The user sees: the update happened, or it
   didn't and here is why. The recurring headache was absorbed by the
   layer where the pain lives, once, in software.

## The seams, stated openly

- The **filesystem design** is open. The trace above says "storage"
  because the filesystem that will sit against the kernel is still
  being designed; when it settles, this walk gets its name.
- The **unified IPC chapter** — the exact contract every signal in
  step 2 obeys — is open.
- The **module boundary contract** is open: what a Chain may and may
  not hold is designed in practice before it is written as law.
- The **SystemAI** is unnamed, for the same reason the kernel is: the
  name comes when the thing is understood well enough to carry one.

*This walk is a rendering of the working design files, regenerated as
the design moves. If it ever disagrees with them, the working files are
where the truth is being decided.*
