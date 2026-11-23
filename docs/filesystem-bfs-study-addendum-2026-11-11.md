# BFS study addendum — for the Trinity filesystem review (prepared 2026-11-11)

**Status: STUDY ADDENDUM ONLY.** Companion to
`filesystem-architecture-study-primer-2026-11-11.md` and the seed
`filesystem-cluster-rebuild-2026-11-11.md`. BFS (the Be File System,
BeOS's and Haiku's native filesystem) was named by the owner on
2026-11-11 as the first owner-named candidate for deeper examination
in the filesystem review, for its metadata features. This addendum
puts the primary source and the examination questions in one place.
It adopts nothing; whether any BFS mechanism survives the hash-out
is the owner's call, exactly as the seed records.

Terms, defined at first use:
- **Extended attribute:** a typed name/value pair attached to a file
  in addition to its name and data — e.g. an email file carrying
  sender and subject as filesystem data, not inside the file body.
- **Index (filesystem sense):** a sorted lookup structure (BFS uses
  B+ trees — balanced tree indexes that keep keys sorted for fast
  range lookup) the filesystem itself maintains over selected
  attribute values, so a query reads the index instead of scanning
  every file.
- **Live query:** a saved query whose result set the filesystem
  keeps updated as files change — a folder that is a standing
  question, not a location.

## The primary source

Dominic Giampaolo, *Practical File System Design with the Be File
System* (Morgan Kaufmann, 1999; ISBN 1-55860-497-9) — written by
BFS's designer (BFS was developed by Giampaolo and Cyril Meurillon
in 1996–97). Out of print; freely available as a full-text PDF /
archive.org full text. The load-bearing chapters for this review:

- Ch. 5 (book's Ch. 4 in some paginations) — Attributes, Indexing,
  and Queries: attributes as typed data, which attributes get
  indexed and why, and the query language.
- The vnode-layer chapter — the Node Monitor (the filesystem's
  change-notification mechanism) and Live Queries: how a query
  stays live instead of being re-run.

Second sources checked 2026-11-11: the Be Book (Haiku's hosted
BeOS API reference, Storage Kit / fs_attr and fs_query functions)
for the programmer-facing surface, and the OSDev Wiki BeFS page
for the on-disk structures (superblock carries an indices root;
each inode carries an attributes directory pointer and inline
small-data attributes).

Sources:
- https://archive.org/stream/practical-file-system-design/practical-file-system-design_djvu.txt
- https://www.haiku-os.org/legacy-docs/BeBook/AttributeFunctions_Overview.html
- https://wiki.osdev.org/index.php?title=BeFS&oldid=29835

## What BFS actually is, in the review's terms

- 64-bit, **journaled** (a log of intended changes is written
  before the change, and replayed after a crash — see primer for
  the term) — notably **not** copy-on-write. BFS takes the
  journaling fork where HAMMER2 and ZFS took the COW fork; its
  crash story is a different branch of the primer's section 2.
- Attributes are first-class and typed; applications define the
  fields that make their data useful. Indexing is selective: an
  attribute is indexed only when asked, because every index is a
  structure the filesystem must update on every write to that
  attribute — the speed of queries is paid for in write cost.
- Name, size, and last-modified time are indexed by default.
- Queries run against the indexes; live queries ride the Node
  Monitor so results update as matching files appear, change, or
  vanish.
- BFS is a **single-node** design. It has no cluster, replication,
  or multi-node coherency story at all — the same shape of hole
  the primer found in HAMMER2's clustering core, except BFS never
  attempted the layer. Any cluster use of its metadata ideas is
  design work, not extraction.

## Examination questions for the review (open, not answered)

1. **Index cost accounting:** if Trinity's metadata (Hel's
   metadata-only audit records, Mimir manifests, checkpoint
   records) were filesystem attributes, which of them earn an
   index under BFS's pay-on-write rule, and which stay unindexed?
2. **Live queries across the case:** a live query is maintained
   by one node's Node Monitor. What is the equivalent when the
   volume spans boards that dissolve and reform (seed testing
   stage 3)? The mechanism does not transfer as-is; what, if
   anything, replaces it?
3. **Journaled metadata vs. COW metadata:** BFS journals its
   attribute/index updates. HAMMER2's checksummed COW topology
   protects metadata by construction. If the rebuild extracts
   BFS's attribute model onto a COW base, do the index structures
   inherit COW protection, or do they need their own consistency
   rule? This is an observation about a fork, not a verdict.
4. **Attribute size discipline:** BFS keeps small attributes
   inline in the inode and spills larger ones to an attribute
   directory. Trinity's metadata structures vary in size by
   orders of magnitude — where would the inline/spill line sit?

## What this addendum does not do

- It does not add BFS to the design, rank it against HAMMER2, or
  close any seed open question.
- It does not summarize the book in place of reading it — the
  chapters above are the reading assignment, in the owner's
  two-pass pattern: read to enjoy, study to understand.
