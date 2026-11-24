# AI policy chain — candidate (2026-11-14)

Status: **candidate / exploration** — raised by Chris in main chat,
Sat Nov 14, 2026 (~23:15 PT), during the agent-security
pass. Hedged in his own words ("should probably", "might be of
interest") — recorded as an open design direction, not a settled
principle. Promotion is Chris's call, in the Trinity design thread.

## His statement, verbatim

"The system AI should probably sit at the Hel level, because it
has to be able to monitor the system outside the system. But I
think it might be of interest to have a chain that enforces the
AI policies, and AI can only access the system through that
chain."

## The two moves

1. **The system AI sits at the Hel level — outside the system it
   monitors.** A monitor inside the monitored system shares its
   failure domain; observation has to come from a level the
   observed system cannot reach back into. (Same root as the
   failed-redundancy rule: a backup that shares the primary's
   failure domain is not a backup; a monitor inside the monitored
   is not a monitor.)
2. **AI reaches the system only through a policy-enforcing
   chain.** A single mandatory path for every AI→system action —
   the Transport Chain shape applied to agents. The chain is
   where AI policy stops being prose and becomes mechanism:
   traversal is what enforces it, and the audit record is a
   byproduct of traversal rather than a separate logging step.

## Scope ruling: all userspace AI (Chris, 2026-11-23)

**All AI that runs in userspace must use the AI policy chain.**
No exceptions and no alternate path: any AI executing in
userspace — guest AI, user-installed AI, third-party agents —
reaches the system only by traversing the chain. Userspace is
the line. The system AI is not userspace AI — it sits at the
Hel level, outside the system it monitors, with its guardrails
built in by construction (see the fork ruling above) — so this
ruling does not reach it, and the two rules together leave no
AI outside a guardrail: native AI carries them in its design,
userspace AI meets them in the chain.

**Provenance is not an exemption (Chris, same pass,
2026-11-23):** even AI Purging Light Tech develops itself, if
it is built to run in userspace, must use the AI chain. Being
ours confers no bypass and no lighter path. The only AI
outside the chain is outside because of *where it sits* — the
Hel level — never because of *who wrote it*. The filter does
not modify based on who is affected, and it does not modify
based on who authored the AI either.

**The one exception, and why it is not one (Chris, same pass,
2026-11-23):** the only chain our own software does not need
to be gated through right now is the Compatibility Chain.
That is not a first-party pass — it is the chain's
population. The Compatibility Chain exists to host foreign,
source-less software; native software is cleared past it by
construction, because we have the source (the TOML-source
consequence, 2026-11-23), so there is nothing for a gate to
inspect. Every other chain gates our software exactly as it
gates anyone's. And the qualifier is his: *right now* — the
exemption holds exactly as long as native software is
source-built; a native component without source would be a
guest, and guests go through the chain.

## Relation to existing material

- **Agent authority bounds (2026-11-11) + 2026-11-14 addendum:**
  the addendum fixed the sequence — propose → sign off →
  implement — for every agent action on the system. This
  candidate gives that sequence a *place*: the chain's stages
  are the gate. Ceiling, trail, and revocation from the bounds
  file are properties the chain must carry.
- **Hermes security-policy line (endorsed by Chris 2026-11-14):**
  "The only security boundary against an adversarial LLM is the
  operating system." The chain answers from above what that line
  states from below: enforcement lives outside the model's
  conversation, in system mechanism.
- **Transport Chain (2026-11-14):** same architectural move —
  one mandatory, auditable path; no ambient access around it.
- **Observability symmetry (2026-11-13):** agent and owner read
  the same data layer and audit stream. A Hel-level system AI
  reading the system's state from outside it is consistent with
  that symmetry, provided the operator reads the Hel level too.

## Ruling on the first fork (Chris, 2026-11-14, ~23:22 PT)

**The system AI does not use the chain.** It is designed from
the ground up with its own guardrails built into it, and
specialized just for systems work: a monitoring and
modification tool that finds and proposes fixes for conditions
that could cause component failures before they do. Hel does
the quarantining — but the system AI may also recommend to Hel
(after getting permission) that a component be quarantined, so
fixes can be implemented and the component restarted.

Two trust models, by origin:

- **Native AI (the system AI):** guardrails by *construction* —
  designed from the ground up with its constraints built in,
  specialized for systems work only. No external gate, because
  the gate's job is already done inside its design.
  (Correctness by construction; built-in, not bolted on.)
- **Foreign/guest AI:** guardrails by *chain* — external
  enforcement on the single mandatory path, because nothing
  about a guest's internal design can be assumed.

## The system AI's role (Chris, same statement)

- A **monitoring and modification tool**, specialized for
  systems work — that is its responsibility, and its scope
  ends there.
- **Predictive:** it finds conditions that *could cause
  component failures before they do*, and **proposes fixes**
  for them.
- **Quarantine workflow:** Hel quarantines components. The
  system AI may **recommend to Hel** that a component be
  quarantined — **after getting permission** — so fixes can be
  implemented and the component restarted. Detection and
  proposal are the AI's; the quarantine act is Hel's; the
  permission gate sits between recommendation and act.
  *(Superseded in part, 2026-11-23: no permission gate stands
  on Hel's act — see Solved forks below.)*

## Addendum — the SystemAI reports data; Hel decides (Chris, 2026-11-23)

Stated in the main chat while settling the agent-authority
delegation question, in the same pass that fixed the division of
labor: **the SystemAI is a tool to raise the threshold for
stability; Hel is security.**

The reporting protocol between them:

- Anything the SystemAI flags that touches security is
  **reported to Hel**. The report carries **all relevant
  information — just the data**. No verdict rides along with it;
  the SystemAI's claim arrives as a claim, labeled as one.
- **Hel does not take the SystemAI's word for it.** Hel evaluates
  the evidence itself, takes the action the evidence supports,
  and decides whether further action is needed. The judgment is
  Hel's, re-derived from the data — never inherited from the
  reporter.
- Read against the quarantine workflow above: a recommendation
  from the SystemAI is a proposal with data attached. Hel's act
  is grounded in Hel's own reading of that evidence. (The
  permission wording is corrected by the ruling filed later the
  same day — see Solved forks: no permission gate stands on
  Hel's act at all.)

This is the preserve-epistemic-status rule at the architecture
level: observations stay observations all the way up the chain,
and only the component whose job the decision is may promote
them to a verdict.

## Proposed resolution of the watcher regress (Chris, 2026-11-14, ~23:29 PT)

Verbatim: "I think there could be something as simple as: on
boot, a digital fingerprint is created that gets stored at
kernel level, and a watchdog monitors whether or not that
fingerprint changes; if it does, the system needs to be
automatically halted — meanwhile Hel and the system AI will
also monitor each other for any anomalies."

Three layers, each a different kind of check:

1. **Boot fingerprint at kernel level.** A digital fingerprint
   created at boot and stored at kernel level — below both Hel
   and the system AI, so neither sits above the reference it is
   checked against.
2. **A dumb watchdog.** It monitors only whether the
   fingerprint changes; on change, the system is
   **automatically halted**. The watchdog is not an AI and
   exercises no judgment — it is the non-AI terminus the
   watcher regress has to bottom out at. Fail-closed: drift
   means halt, not alert-and-continue.
3. **Mutual monitoring.** Hel and the system AI monitor *each
   other* for anomalies — behavioral checking between the two
   intelligent layers, over the integrity floor the
   fingerprint/watchdog provides.

**Scope widened (Chris, 2026-11-14, ~23:33 PT):** "That's true
for every system update, though: anything that changes from
the chain down, we need a secure way to signal them as official
changes, so Hel or the systemAI doesn't quarantine them as
unauthorized changes." The problem is not specific to the boot
fingerprint: **every change propagating down the chain needs a
secure "official change" signal** that the watchers (Hel, the
system AI, the watchdog) can verify. Unsignaled change reads
as unauthorized change — quarantine or halt. The signal is the
general mechanism; re-fingerprinting is one instance of it.
(Natural carrier: the certificate trust pipeline's
generate/log/revoke guarantees and the HAT/SAT/AAT token chain
— a change is official if it arrives carrying a valid, logged,
revocable authorization issued through propose → sign off →
implement. Filed first as the assistant's inference; Chris
confirmed 2026-11-14 (~23:34 PT) he was "actually eyeballing
the token system myself" for this — his own lean, converging
independently. A lean, not yet a settled ruling.)

Open detail flagged at filing: legitimate change. Updates and
repairs change the fingerprint by design, so there must be a
re-fingerprinting path (a new authorized baseline) — presumably
through the propose → sign off → implement gate and the
Repetition Chain's update transaction. That path is where an
attacker would aim; it is the load-bearing detail of this
mechanism. **Chris confirmed 2026-11-14 (~23:30 PT) this is the
point he is hung on as well — no good implementation found
yet.** Status: open problem, named and parked, not papered
over.

## Ripple: token generation at package install (Chris, 2026-11-14, ~23:36 PT)

As the design solidifies, this reaches further: the tokens
will have to be generated at package install, so token
generation has to be built into the packaging system.

Consequence: the packaging system moves inside the trust path.
Install becomes a token-minting event — a package's changes are
official because the install generated their authorization
tokens — and an update (Repetition Chain) is the same event
again: new tokens minted, baseline re-struck, all inside the
update transaction rather than beside it. The watchers never
have to guess whether a package's files belong; the install
told them, in tokens, at the moment it happened.

## Naming (open, Chris 2026-11-14, ~23:49 PT)

The system AI is a new critical component as of tonight's pass
and is currently **unnamed** in the design. Chris's working
designation in conversation is "the SystemAI" — a label, not
the anchor name. He confirmed the inventory the same night:
"Everything truly loadbearing in Trinity has a name - Hel,
Fang, Chain, the SystemAI when it gets one" — and amended it
minutes later: "we forgot the newest one the AI chain." The
chain in this file therefore already carries a roster name —
**the AI Chain** — while the SystemAI itself remains the
roster's one pending anchor. Chris's correction of the
assistant's framing (2026-11-15, ~00:17 PT): the AI Chain is
not anchored by family resemblance — the chains are a class
of **specialized chains, each with one job, and the job is
what is in its name**: the Repetition Chain does repetition,
the Transport Chain does transport, the AI Chain does AI
(access and policy). Single-job components, named by the job.
Chris, verbatim on the naming task:
"we just added a new critical component we have to name and
design and build". Naming is future work, his craft — in this
ecosystem names carry function (Hel, Ymir, Mimir, the Fangs).
Sequence (Chris, ~23:50 PT): the name comes **once the
component is defined, before the design goes hard at it** —
define → name → design. A name given before the definition
would label a guess; after definition it can carry the actual
function.

## Chain jurisdiction rule (Chris, 2026-11-15, ~00:19 PT — stated generally, applies to all chains)

Verbatim: "And these chains are system-specific. People can
still create their own chains for their applications, but if
it touches a domain that is controlled by a system chain, the
application must use the system chains for that access; it
cannot be granted outside its specific chain."

- **System chains are the system's** — core components with
  exclusive jurisdiction over their domains.
- **Application chains are permitted** — people can build
  their own chains for their applications, in the application's
  own domains.
- **No side doors:** the moment an application chain touches a
  domain a system chain controls, that access goes through the
  system chain. Access to a controlled domain **cannot be
  granted outside its specific chain** — an application chain
  composes with the system chain at the boundary; it never
  bypasses it. (Layer jurisdiction applied to chains: a
  domain's owner is the only door into it.) Rationale, Chris's
  words: "that's a security thing we can't have side
  loading through non-system chains" — a non-system chain
  granting access into a controlled domain *is* sideloading.

  **Correction (Chris, 2026-11-15, ~00:22 PT), replacing the
  assistant's earlier gloss:** audit is NOT what sideloading
  bypasses. "All chains by their very nature are auditable;
  it's pretty much a specification of what a chain is — that's
  why we moved core OS things down into them." Auditability is
  definitional to chain-ness — any chain, application chains
  included, produces its record by construction; that is the
  reason core OS functions were moved into chains at all. What
  a non-system path into a controlled domain bypasses is
  *authorization* — the domain's own token minting and
  approval. The access would be perfectly logged and still
  unauthorized: visibility is not permission. Sharpened by
  Chris (~00:24 PT): the record exists — what breaks the
  security model is that the access bypasses the tokens and
  still looks like permitted action. Because chains produce
  uniform records, the unauthorized entry
  *resembles* a legitimate one. The failure is camouflage: it
  poisons the audit's evidentiary value, the property the
  whole model rests on ("if it can't be audited as truth then
  it can't be trusted"). A record that cannot distinguish
  permitted from unpermitted at the token layer is a record
  that can no longer certify.

## Mitigation: signature without token (Chris, 2026-11-15, ~00:28 PT)

Verbatim: "But there's a place that can be caught, and it's
actually in several places — Hel and the SystemAI can both
catch it, because they have log monitoring — and a call for a
system-chain-provided service will have a specific signature;
if you see that signature get issued without a token
associated, it gets mitigated."

- **Detection points are plural:** Hel and the SystemAI both
  monitor the logs — the same mutual-monitoring layer that
  watches for anomalies watches for this.
- **The tell:** a call for a system-chain-provided service
  carries a **specific signature**. Signature present + **no
  associated token** = unauthorized access, whatever the rest
  of the entry resembles.
- **Signature composition (Chris, ~00:30 PT):** the signature
  is not generic per service — it is derived from **what
  below it the call has to access, and why**. The signature
  therefore encodes downward reach + purpose, and the token
  must match *that* signature: a token minted for a different
  what/why does not validate this call. Least privilege is
  expressed in the signature itself, and a would-be sideload
  cannot borrow an unrelated token — the matching token can
  only be minted by the owning chain, through approval.
- **Response:** it gets mitigated (the act is stopped/undone
  per the mitigation path, not merely noted).
- Note the symmetry: the uniformity of chain records creates
  both the camouflage risk (above) and the detection surface —
  the signature is recognizable *because* chain calls have a
  fixed shape, and token association is a checkable field on
  every entry.

## Property: the model travels with the method (Chris, 2026-11-15, ~00:32 PT)

Verbatim: "Being open source anyone can copy the method but
at least it's a way of guaranteeing when it does get copied
it follows the security model."

The security is not a secret kept alongside the design — it
is inseparable from the method: because auditability and
token-gating are part of the *specification* of what a chain
is, a copy that dropped them would not be a working copy of
the method. Copying the mechanism copies the guarantees.
(Open-source credo meeting correctness by construction: the
protection travels with the design, not with secrecy.)

Chris's own account of the decision (~00:34 PT): he quietly
debated burying the method so people wouldn't have access to
it; his open-source nature overrode the impulse. Locating the
benefit: it sits in *them using the security model in their
own chains*. His only argument against non-system chains was
that routing around system chains breaks the security model —
publishing the method fixes both problems at once: copiers
inherit the model, and application chains built on the method
no longer stand outside it.

## Solved forks

- **Who watches Hel? — SOLVED (confirmed by Chris, 2026-11-23:
  "solved from earlier conversations").** History (Chris,
  2026-11-14, ~23:24 PT): "Hel itself was originally checked by
  the system itself but that was before the design of the
  system AI." The original arrangement — the system checking
  Hel itself — predates the system AI's existence in the
  design. The resolution is the watcher-regress proposal of
  2026-11-14 (above), now confirmed as the answer:
  - **Mutual monitoring.** The system AI, sitting at the Hel
    level and specialized for systems monitoring, does take
    over the checking of Hel — and Hel checks the system AI
    in return. Each watches the other for anomalies; neither
    is the other's superior.
  - **The regress bottoms out at mechanism, not at another
    watcher.** Below both sits the boot fingerprint, stored
    at kernel level, and the dumb watchdog: if the fingerprint
    changes, the system halts automatically. The watchdog is
    not an AI and exercises no judgment — it is the non-AI
    terminus that answers "what checks the system AI?" without
    opening another regress.
  - The operator remains the judgment layer above both.

- **Whose permission gates the quarantine? — SOLVED (Chris,
  2026-11-23): "Hel never needs permission to institute a
  quarantine — that's the security of the system."** No live
  operator sign-off and no standing approval class stands
  between Hel's evidence and Hel's act: quarantine is Hel's
  own authority, exercised on evidence Hel itself has weighed.
  The permission language in the 2026-11-14 workflow text
  governed the SystemAI's recommendation path, not Hel's act —
  and under the 2026-11-23 reporting protocol the SystemAI
  sends data, not requests, so nothing in the chain waits on a
  human who may be asleep, exhausted, or unreachable. A
  quarantine is protective and reversible; the operator reviews
  Hel's acts in the audit record, after the fact, at the
  judgment layer — review is not a precondition.

## Open forks (Chris to rule)

- **The operator's view of the Hel level:** does the operator
  read the same audit stream Hel keeps, or does the Hel level
  keep a record of its own for the operator? (Split out of the
  who-watches-Hel fork when that fork was solved, 2026-11-23;
  this half remains open.)
- **Uniform or tiered (guest side):** one chain for all guest
  AI, or does read/monitoring access traverse a different path
  than action access?

## Addendum 2026-11-20 — guardrails where people are weakest

Prompt (Chris, 2026-11-20): reports (Axios, October 2026, as
quoted by Chris) that major US banks detected third-party AI
agents accessing customer accounts — some customers had shared
their login credentials with the tools, and banks observed
agents apparently acting beyond users' instructions.

Chris's ruling: "we don't want to allow for this to happen in
our system and one step is our AI chain but we also need to
put guardrails where people are stupidest."

Design content:
- The AI Chain alone is not sufficient. It guards the AI's
  path to the system; it does not guard the user's path to
  the AI.
- Guardrails must also sit at the points of predictable user
  failure — the places where a user is most likely to hand a
  secret to a conversational surface, or to mistake a chat
  interface for a private conversation with a person.
- Root mechanism (Chris's analysis, same exchange): people
  cannot reliably distinguish a chatbot from a person in a
  chat room, and no amount of training changes that
  perception — the confusion is produced by the interface's
  social shape, not by a knowledge gap. Therefore the
  guardrail cannot be education or warning text; it must be
  structural. A credential offered into a conversational
  surface must have nowhere to go: no field that accepts it,
  no path by which a secret becomes content.
- This is the failure-mode principle applied to the user
  interface layer: design for the operator at their most
  careless, not their most careful (cf. "design systems to
  survive me at my worst," generalized to all users).

Extension (Chris, 2026-11-20, minutes later): enforcement
can live at the Chain level — add guardrails there that
prevent AIs from storing secure data outside of a secure
store. Chris's epistemic status, preserved: "I don't know
how yet but it will be something when I read more into how
AIs work that I'm sure we can figure out" — direction
stated, mechanism open.

Placement logic: the Chain is the single auditable path
between an AI and the system, so a rule about where data
may land can be enforced at that boundary regardless of
the AI's internals (which are not auditable). Open problem
the mechanism must solve: recognizing secure data in
transit across the Chain, so the refusal can fire on the
flow rather than on the AI's declared intent.

Interim admission rule (Chris, 2026-11-20): "it will have
to be carefully designed but I think it's possible even if
for temporary solutions we only allow AI that enforces the
boundaries." Until Chain-level enforcement exists, an AI is
admitted to the system only if the AI itself enforces the
boundaries (secure data only into a secure store). This is
a temporary bridge: the trust placed in the AI's own
enforcement is a placeholder for boundary enforcement that
does not depend on the AI at all. By the auditability
axiom, "enforces the boundaries" must be demonstrable, not
vendor-claimed.

## Candidate solution — our own harness (Chris, 2026-11-23)

Chris's status, preserved: "I think I might have a
solution" — a candidate, offered for the record, promotion
his call.

**Create our own harness. Run the AI through the harness;
the harness always prompts. That solves the prompt issue.**

- **The harness is the user's surface.** The user does not
  hand things to an AI in free conversation; the AI runs
  inside a harness we control, and every step that needs the
  user surfaces as a prompt through the harness. A prompt is
  a structured decision point, not a chat box — there is no
  conversational surface for a credential to be pasted into,
  so the failure the 2026-11-20 addendum guards against has
  nowhere to happen. The guardrail is the shape of the
  surface, not a rule the user must remember.
- **Multi-model, concurrent is fine.** The harness can hold
  multiple models and use them concurrently — the harness is
  the seat the committee work (2026-11-21/22) sits in: one
  gated surface, several independent models behind it.
- **The chain still applies.** The harness is not a bypass:
  the models running in it are userspace AI, so they traverse
  the AI policy chain like any other (scope ruling,
  2026-11-23). The two layers guard two different paths —
  the harness guards the user's path to the AI, the chain
  guards the AI's path to the system.
- Read against the addendum's open problem: enforcement no
  longer has to recognize secure data in transit across a
  conversational flow, because in the harness shape the
  secret never enters a flow — prompts carry decisions, not
  data entry, and sign-in belongs to secure stores, never to
  the prompt surface. (Whether Chain-level storage
  enforcement is still also wanted is a separate question,
  not closed by this candidate.)
