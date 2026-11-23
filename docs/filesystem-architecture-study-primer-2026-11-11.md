# Filesystem architecture — study primer (prepared 2026-11-11)

**Status: STUDY LIST ONLY.** Prepared in the background after the
filesystem seed was filed (filesystem-cluster-rebuild-2026-11-11.md).
This is not a design document, not a spec, and it closes no open
question in the seed. It exists so the hash-out starts from primary
sources — the design documents and papers written by the people who
built the filesystems — instead of summaries. Order follows the
owner's sequencing rule stated 2026-11-11: the architecture blank
gets filled in the direction the OS-on-hardware simulation needs
first (single-board correctness and crash behavior before cluster
behavior).

Terms, defined at first use:
- **Copy-on-write (COW):** a filesystem update never overwrites a
  block in place; it writes a new block elsewhere and then updates
  the pointer to it, so the old, consistent version survives until
  the new one is fully on disk.
- **Crash consistency:** the guarantee that after a power loss or
  crash at any moment, the on-disk filesystem is either the old
  consistent state or the new one — never a half-written mix.
- **Journaling:** before changing the filesystem, write a record of
  the intended change to a dedicated log area; after a crash,
  replay the log to finish or discard the interrupted change.
- **fsck:** a filesystem check-and-repair program run after an
  unclean shutdown on filesystems that cannot guarantee crash
  consistency by construction.

## The hole in the exhibit (read this first)

Matthew Dillon's HAMMER2 DESIGN document, version 6 dated
08-Dec-2018 (the latest version in the DragonFly source tree,
checked 2026-11-11 via the BSD cross-reference at bxr.su,
/DragonFly/sys/vfs/hammer2/DESIGN), lists the filesystem core as
operational — snapshots, compression, COW, filesystem consistency —
but the **clustering core** as incomplete:

- Network message core — operational; network block device —
  operational
- Error handling — under development
- Quorum Protocol (the rule by which nodes agree which copy of the
  data is authoritative when they disagree) — under development
- Synchronization — under development
- Transaction replay — not specced yet
- Cache coherency (keeping every node's cached view of a file the
  same) — not specced yet

And under Pending Features: "Constructing a filesystem across
multiple nodes … CURRENT STATUS: Not yet operational," and
background synchronization/mirroring likewise "Not yet operational."

Implication for the rebuild, stated as an observation, not a
verdict: the cluster layer is the reason HAMMER2 is the exhibit,
and it is the layer the exhibit itself has not finished. A
rebuild-and-modify extraction therefore cannot extract the cluster
mechanism from HAMMER2 — that part has to be designed, informed by
the single-board mechanisms HAMMER2 did finish (COW topology,
blockref checksumming, flush/sync model). This sharpens, and does
not answer, the seed's first open question (the consistency model
when a node vanishes mid-write).

Source: http://bxr.su/DragonFly/sys/vfs/hammer2/DESIGN

## Reading order

### 1. The basics — on-disk structure and vocabulary
"KHB: A Filesystems reading list," Valerie Henson, LWN.net —
a curated primary-paper list. Start with the paper it calls
absolutely foundational: "A Fast File System for UNIX" (McKusick,
Joy, Leffler, Fabry) — the FFS/UFS design most production UNIX
filesystems descend from — plus the Vnodes paper (Kleiman) for the
interface layer a filesystem plugs into.
Source: https://lwn.net/Articles/196292/

### 2. Crash consistency — the four approaches
Same LWN list, "Slaying the fsck dragon" section: journaling
(Tweedie's ext2 journaling paper), log-structured filesystems
(Rosenblum & Ousterhout), soft updates (McKusick & Ganger), and
copy-on-write. This is the fork every filesystem design must take
explicitly; HAMMER2 and ZFS both chose COW, and the seed's
single-board stage (stage 1 of the testing chapter) is where this
choice is proven first.
Source: https://lwn.net/Articles/196292/

### 3. Copy-on-write in production
"File System Design for an NFS File Server Appliance" (Hitz, Lau,
Malcom) — WAFL, the first major production COW filesystem, via the
LWN list above. Then "The Zettabyte File System" (Bonwick et al.)
for ZFS's version: COW tree, transaction groups, uberblock (the
single root pointer whose atomic update publishes a whole
consistent state). ZFS is in the owner's lived history; this is the
architecture layer under that operator experience.
Sources: https://lwn.net/Articles/196292/ and the ZFS paper as
linked from the course notes at
https://naizhengtan.github.io/23spring/notes/week14a.txt

### 4. The exhibit's predecessor
"THE HAMMER FILESYSTEM," Matthew Dillon, 21 June 2008 — HAMMER1's
design: single global B-Tree, full history retention, no-fsck
crash recovery. Read before HAMMER2 so the delta between the two
designs (B-Tree history machine → COW radix-tree design) is visible
as Dillon's own corrections.
Source: https://agc.asuis.workers.dev/read-http-www.dragonflybsd.org/hammer/hammer.pdf
(Proxy copy of the dragonflybsd.org original; treat wording as
Dillon's, hosting as a mirror.)

### 5. The exhibit itself
HAMMER2 DESIGN document v6 (08-Dec-2018), Dillon — read with the
status table at the top kept in view: operational vs. under
development vs. not specced is the author's own grading, and the
cluster sections are explicitly the unfinished part.
Source: http://bxr.su/DragonFly/sys/vfs/hammer2/DESIGN

### 6. Scale and extents (later, when single-board is understood)
"Scalability in the XFS File System" (Sweeney et al.), via the LWN
list — extent-based layout, B+ trees, journaling at scale. XFS is
also in the owner's lived history; parked last because the
simulation needs single-board crash/COW behavior before it needs
scale lessons.
Source: https://lwn.net/Articles/196292/

## What this primer does not do

- It does not select which lessons from other filesystems get
  designed in — that selection is the owner's, per the seed's
  open-questions list.
- It does not propose a consistency model, an on-disk format, or a
  name.
- Every source above was opened and read on 2026-11-11, not just
  listed from a search index.
