# Transport Chain — inter-module communication (2026-11-14)

Status: Chris endorsed the core idea in-session ("that's a winner — this is exactly the right use case"). Shape below as discussed; three forks remain open for his ruling.

## Problem
User applications are modules: separate, isolated — but able to communicate as needed. Chris's framing: a capabilities-based transport layer, like in a cluster.

Origin (Chris, same session): the question fell out of a harder upstream problem he was working through — how to make *existing* software into modules. Legacy programs assume ambient authority (files, sockets, shared memory, global namespaces), so modularizing them means a wrapper that translates ambient calls into capability-checked transport messages. The Transport Chain serves native modules and wrapped legacy modules identically — neither side can tell which it's talking to.

## Decided shape
- **Module = actor.** Isolated address space, own state. Shared mutable state (`ref`) lives only inside one actor; anything crossing a boundary moves as `iso` (ownership transfer), `val` (immutable), or `tag` (identity only) — Maia's existing data-capability rules become the transport's message rules. One vocabulary, no translation layer.
- **Capability = unforgeable token; holding it is the permission.** No ambient authority. A capability names an endpoint plus rights (send-only, request/reply, attenuate-and-pass-on). Rides the existing HAT/SAT/AAT capability token chain.
- **Transport is a Chain.** All inter-module communication passes through the Transport Chain — single mandatory path, so every grant, introduction, and message is auditable by construction (same pattern as the repetition Chain). Modules never wire to each other directly.
- **Location transparency.** The Chain does not distinguish a local endpoint from a module on another node of the portable cluster — same message, same capability, Chain routes. Cluster semantics for failure: a crashed module looks like a dead node, not a hung caller.
- **Introductions are brokered.** Initial capabilities are granted by the shell / system builder from the module's TOML manifest declaring what it may request. Modules cannot discover or reach each other without an introduction.

## Test candidate — file browser (Chris, same session)
Chris identified the file browser as a good test candidate for the Transport Chain.
A file browser must browse the tree, but opening a file means handing it to the right environment (editor, viewer, player) — which requires communicating with that environment to load the file. Shape this implies:
- The browser holds *directory* capabilities (enumerate/traverse the scopes the user granted it) — listing names and metadata does not imply it can read contents.
- On open, the browser does not read the file itself and does not hold standing capabilities to every application. It sends an open request through the Transport Chain, and the handler for that file type is resolved and, if needed, launched by the broker/shell.
- What crosses is an *attenuated file capability* — that one file, read-only for a viewer, read/write for an editor — passed to the handler module, which loads the file under its own authority. The browser never gets the handler's powers; the handler never gets the browser's tree.
- The user's selection is the authority source: the gesture picks the file, the Chain mints/passes the scoped capability, the audit trail records the handoff.

## Open forks (Chris to rule)
1. **Synchronous calls vs pure messages.** Cluster thinking favors messages: a dead endpoint is observable, not a hang. Assistant lean: messages.
2. **Revocation.** Epoch/generation scheme: bump the generation and all tokens from the old generation are void. Cheap, total, auditable — but all-or-nothing per epoch.
3. **Delegation.** Can a module pass a capability onward without asking the broker? Assistant lean: grants brokered and logged; passing allowed only attenuated (fewer rights, never more), with the audit trail recording who held what when. This fork shapes the whole design.
