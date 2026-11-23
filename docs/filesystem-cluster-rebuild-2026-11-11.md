# Trinity filesystem — HAMMER2 rebuild seed (opened 2026-11-11)

**Status: OPEN DESIGN, filed to hash out.** This is a design seed,
not a spec and not a settled decision. Until today the owner had
purposely avoided filing this — the design was (and remains) open.
On 2026-11-11 he directed it be put into Trinity to hash out.
Nothing in this file is closed; the open questions at the end are
the work.

## The intention, in the owner's words

- "I like Hammer2 but it's not available outside one BSD."
- "I was going to rebuild it and modify it based on other
  filesystems I've used."
- "It needs to be able to go across clusters easily because I
  still want to build the portable cluster using my own boards."

## What this is

A **rebuild-and-modify extraction**, in the owner's established
pattern (cf. Trinity Chain from blockchain, /usr/apps from the
macOS app model, the licensing theory from FreeBSD): study the
mechanism in the wild example, walk away, rebuild it in his own
materials, modified by what every other system he has run
taught him. It is explicitly **not a port** of HAMMER2.

## The exhibit: HAMMER2 (DragonFly BSD)

Verified 2026-11-11 (web, two sources):

- HAMMER2 is DragonFly BSD's filesystem, its default since
  DragonFly 5.2; redesigned from HAMMER1 specifically to support
  **clustering**. Features: copy-on-write, writable snapshots,
  online and batched deduplication, directory entry indexing,
  multiple mountable filesystem roots, mountable snapshots,
  low memory footprint (a deliberate contrast to ZFS),
  compression, encryption, zero-detection, data and metadata
  checksumming, synchronization to other filesystems or nodes.
- Filesystems are immediately available after a crash — no fsck.
- As a read-write filesystem it exists **only on DragonFly**.
  Tomohiro Kusumi ported the hammer2 utility to FreeBSD, NetBSD
  and OpenBSD, but for a **read-only** implementation only.
- Non-portability is by design, not neglect: HAMMER2 is welded
  to DragonFly's VFS layer, scheduler and buffer cache, and
  portability was explicitly not allowed to impede its goals.
  Matthew Dillon built it for his kernel.
- Owner's own field experience (2026-10-10): he stood up a
  stable DragonFly BSD install with a full desktop; it never
  became his daily OS (limited software availability, painful
  configuration). He liked HAMMER and wished he could use it on
  his Linux box.

## The destination: the portable cluster

The cluster requirement is not abstract. The owner has wanted,
since first entering technology, a **custom portable cluster in
a wheeled case** — take it anywhere, set it up, continue working.
The cluster is to be built from **his own boards** (Ymir, his
RISC-V SoC), running Trinity built to one shared spec, so
compatibility is by construction (see the co-design principle
in GOAL.md). The filesystem is what makes that case one machine
rather than a row of islands: one storage fabric spanning the
nodes, built in — not per-board filesystems with a sync layer
bolted on.

## The hard shape of the problem (noted, not decided)

A portable cluster is a hostile environment for a cluster
filesystem, harder than the datacenter case:

- Nodes power down when the case closes; links vanish because
  the case was wheeled somewhere else.
- The "cluster" is sometimes a single board on a desk.
- Membership is fluid by default, not by failure.

"Goes across clusters easily" therefore has to survive
dissolving and reforming as a normal state, not an exception.

## Chapter — Portable cluster testing (before the user layer)

Added 2026-11-11 on the owner's direction: a dedicated chapter
for portable cluster testing, and it sits **before** any
user-layer work in the sequence. The cluster behavior is proven
at the filesystem layer before anything user-facing is built on
top of it — the same dependency discipline as the rest of the
project (nothing builds on an unproven layer).

**Timing correction (owner, 2026-11-11): design now, test
later.** The cluster capability is **designed for at the core**
from the start — it is a core-level design input, not a feature
layered on afterward (a cluster fabric retrofitted onto a
single-machine filesystem design is exactly the bolted-on shape
this project refuses). But the **testing itself does not happen
until the foundation is built**: the stages below execute only
once the filesystem foundation they test against exists.
Designing for it at the core and testing it after the
foundation are not in tension — the first is what makes the
second possible without redesign.

Test stages to define and prove, in order (criteria for each
are part of the hash-out; none are pre-written here):

1. **One board alone** — the degenerate cluster: single Ymir
   board, filesystem correct with no peers. This is also the
   laptop case. It goes first on the owner's stated reasoning
   (2026-11-11): a single board is easier to test and
   increment — prove single board before scaling. One variable
   at a time: on one board, a failure is a filesystem failure;
   at cluster scale, a failure could be the filesystem, the
   fabric, or the timing, and the read is polluted from the
   start.
2. **Two boards** — one storage fabric across two nodes; a
   file written on one is the same file on the other.
3. **Dissolve and reform** — nodes powered down and back up in
   arbitrary order; the fabric re-forms without manual repair.
4. **Vanish mid-write** — a node disappears during an active
   write (case closed, board pulled); the surviving nodes'
   view of the data is defined and provable, and the returned
   node's state reconciles by the design's rule, not by luck.
5. **The case test** — the actual portable cluster: wheeled
   case closed, moved, reopened elsewhere, powered up in a new
   location; work continues where it left off.
6. **Simulation first** — per the project's method (Design →
   Build → Prove → Checkpoint), each stage is proven in
   simulation before it is asked of real boards, consistent
   with the sequencing pressure above: the OS-on-hardware
   simulation is only as accurate as this layer is defined.

Only after these stages are proven does design work move to
the user layer above the filesystem.

## Open questions — the hash-out list

- Consistency model: what does the filesystem promise when a
  node vanishes mid-write — and when it comes back with writes
  of its own made while separate?
- Single-board mode: is a one-node cluster the degenerate case
  of the same design, or a distinct mode? (Owner's working
  machines are laptops; the filesystem also has to be right on
  one board alone.)
- Which modifications from other filesystems he has used get
  designed in — the owner has not yet named the specific
  lessons (ZFS, Btrfs, XFS, APFS and others are all in his
  history; the selection is his to make in the hash-out).
  **First owner-named candidate (2026-11-11): BFS (the Be
  File System, via Haiku).** The owner likes BFS's features
  and has directed they be examined deeper in the filesystem
  review: extended attributes as first-class data, attributes
  indexed by the filesystem itself, and live queries (a folder
  as a standing query rather than a location) — metadata and
  search built into the filesystem instead of bolted on by an
  external indexer. Whether any of it survives the hash-out
  into the design is undecided; it is on the study list, not
  in the design. The owner's stated reason it caught his eye
  (2026-11-11): "the metadata — because we use that in a lot
  of places." Metadata is load-bearing across Trinity, not a
  filesystem-local concern: Hel's audit is metadata-only by
  design, the interface-parity audit stream logs commands
  (metadata, not content), Mimir's manifests and the
  checkpoint records are metadata structures. A filesystem
  whose metadata layer is native, indexed and queryable is
  therefore a platform-level question: several components'
  designs simplify or harden depending on the answer.
- On-disk format stability contract: a filesystem's format is
  forever in a way APIs are not — what is the migration story
  when the format must change?
- Layer placement: how does the filesystem sit against Garmr /
  the Fang layer, and what does Hel's metadata-only audit mean
  for filesystem events?
- Snapshot/history retention policy: HAMMER-style history is
  powerful and storage-hungry; what prunes it, and who decides?

## Sequencing pressure (noted 2026-11-11)

The owner expects this design to come **sooner rather than
later**, for a dependency reason, in his words: you "can't
simulate OS on hardware accurately without it." The Trinity /
Ymir co-design work (Part II onward: CPU and memory in
simulation against the manuals, Spike as golden reference)
can only model the OS's real behavior as far as its storage
layer is defined — an undefined filesystem makes the
simulation's storage behavior a guess, and guesses are not
allowed to load-bear in this project's method. The filesystem
design therefore sits on the critical path of the hardware
simulation work, not behind it.

## What is NOT decided

Everything above the open questions is origin and exhibit only.
No name, no on-disk design, no consistency model, no schedule.
The owner closes design questions; this file exists so the
hash-out starts from the true origin instead of a reconstruction.
