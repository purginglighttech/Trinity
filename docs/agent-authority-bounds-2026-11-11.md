# Agent authority bounds — candidate Trinity principle (2026-11-11)

Status: **candidate principle** — filed from the main-chat session
of Wed Nov 11, 2026 (graves shift), on Chris's instruction
("Let's add that to the Trinity review too that's important").
Not promoted into GOAL.md's Principles & Philosophy; promotion is
Chris's call, in the Trinity design thread. Kept separate from
`certificate-trust-pipeline-2026-11-11.md` (filed the same night):
the two are adjacent — the token chain is the mechanism, the trust
pipeline is what validates the certificates under it — but they
are different principles and are recorded as such.

## The position, in Chris's words

"I don't disagree that agents are the future I'm exploring their
use in Trinity but what I disagree with is giving full control
to agents."

Agents are expected citizens of the platform. **Full control is
not a configuration the platform offers.**

## The stance toward the trajectory (Chris, Nov 11, ~03:18 PT)

"I can't change the trajectory but I can make rules and policies
that guard the system and my own control over it."

The industry's direction — agents gaining capability and access —
is accepted as a condition, not fought as a battle. What the
platform controls is the terms on which that direction is allowed
inside it: rules and policies that guard the system and preserve
the operator's control. Note where this sits in Chris's own
framing: the guardrails are stated as rules and policies, but in
Trinity's architecture they cannot rest as prose — the filed
principle "we can't beats we won't" requires them compiled into
the capability construction, so that the guard holds even when
nobody is watching and the operator is at his worst. This is the
same architecture as his verbal circuit breaker: a previously
approved policy, laid down in a clear moment, that the present
moment is not free to renegotiate.

## The compromise, stated (Chris, Nov 11, ~03:21 PT)

"It seems a valid compromise I want to be modern to be modern I
have to adopt certain things but I absolutely refuse to hand over
control of my system to any outside force."

Adoption is not surrender. Modernity is admitted *as a guest under
the constitution*: new capabilities (agents included) enter the
platform on the platform's terms — ceiling, trail, revocation —
and control of the system never transfers to any outside force as
the price of admission. The line is not drawn between old and new
technology; it is drawn between **use** and **control**. Anything
may be used. Nothing outside the operator may control. (Same shape
as credo point 2: "The system always belongs to the user and never
should the terms of the system be dictated by any force outside
the user.")

## The three questions (the review test)

For any agent integration, at any layer, the design must answer —
in the architecture, not in policy prose:

1. **What is the capability ceiling?** What is the maximum
   authority the agent can ever hold, expressed as scope on the
   capability token chain (HAT/SAT/AAT) — least privilege by
   construction, with anything above the ceiling *unrepresentable*,
   not merely discouraged. ("We can't" beats "we won't," applied
   to agents.)
2. **What is the evidence trail?** Every agent action lands in the
   auditable record (Trinity Chain / Hel metadata): what was done,
   under which token, on whose instruction. If an action leaves no
   record, the action is not permitted to exist.
3. **What is the revocation path?** The operator can withdraw an
   agent's authority mechanically and immediately — which is where
   the certificate trust pipeline's third leg (revocable by
   design) carries load: an agent whose credentials cannot be
   revoked is an agent with full control by another name.

An integration that cannot answer all three does not ship, however
useful the agent is. This is the auditability axiom ("if it can't
be audited as truth then it can't be trusted") and Chris's AI
tooling boundary ("a tool he can't control or audit is 'not AI to
me'; AI must sharpen his judgment, never replace it") stated as
platform architecture rather than personal practice.

## Human position in the loop

The operator reads and approves at the judgment layer — review is
what ownership feels like from the inside. Agents may draft,
explore, and execute inside their ceiling; they do not *certify*.
In particular, **agent review of agent-produced work is not a
security gate**: a loop in which agents write and agents approve
has no verifier, only confidence. (Exhibit 2 below is exactly that
loop, shipped as a feature.)

## Exhibits (original beside the extraction)

1. **KORA / Kora OS** (evaluated and rejected Nov 10, 2026):
   pitch — "opens the entire operating system to every model, AI
   service and agent." Chris's first read, held: that access
   pattern is the threat model, not the feature.
2. **Omarchy (DHH)** — It's FOSS interview, published Nov 10, 2026
   (read in session Nov 11):
   - Asked what stands between a malicious plugin and a user in a
     marketplace of thousands of AI-generated plugins, the answer
     was that the marketplace is "agentically reviewed" and "far
     more secure than, say, something like the AUR"; that it is "a
     feature, not a bug, that the majority of these plugins have
     been created by AI," because AI is "very good at reviewing
     incoming code for security vulnerabilities." Agent-written,
     agent-reviewed, no human verifier in the loop.
   - The funding: the Omacom Foundation's ~$18.5M in pledges are
     largely AI tokens from Anthropic, OpenAI, and Meta — an
     agent-first desktop whose runway is donated by the agent
     vendors. Dependence on parties who can change the terms,
     under a banner of independence.
   Chris's summary in session: people are "throwing money at it
   when they won't even use it" — funding as affiliation and
   momentum, never priced against audited use.

## Relation to existing principles

- **Capability token chain (HAT/SAT/AAT):** the mechanism that
  expresses the ceiling. This candidate states the policy the
  mechanism must be able to enforce.
- **Correctness by construction:** above-ceiling authority should
  be unrepresentable, not rule-forbidden.
- **Built-in, not bolted on:** the evidence trail and revocation
  path are platform services, not per-agent add-ons.
- **Certificate trust pipeline (candidate, same night):** supplies
  the generation/logging/revocation guarantees for the credentials
  the token chain rides on.

## The capability-opacity caveat (owner, 2026-11-11, evening)

"The only issue is with how closed the actual AI systems
are — you don't actually know the true capabilities." This
is a real limit on the operating stance below, and the
design answer matters: closed systems resist measurement
(weights, training data, scaffolding and silent updates are
all vendor-side; a behavioral measurement is a snapshot of
a target the vendor can move overnight). Therefore the
guardrails must NOT be anchored to capability estimates.
They are anchored to **authority**: the ceiling governs
what an agent is *permitted to touch*, enforced by the
token chain's construction, and it holds no matter how
capable the agent turns out to be — above-ceiling authority
is unrepresentable, so unknown capability has nothing to
escalate into. Design for measured capability where
measurement is possible; design against *unbounded*
capability at the authority boundary, where it is not.

## Operating stance (owner, 2026-11-11, evening)

Added after a day of watching AI-doom warnings accelerate
AI adoption (fear of the technology losing to fear of
competitors having it): "Maintain a clear view of the actual
capabilities, design for them, and maintain the guardrails —
and there's never any worry." The doom frame and the hype
frame are the same failure in opposite directions: both are
claims about capability with no baseline under them. The
platform's answer is procedural, not emotional: know what
the agents in it can actually do (measured, not marketed),
design to that measured capability, keep the three review
questions answered in the architecture, and neither the
hype cycle nor the panic cycle has anything to grab.

## Addendum — the approval gate stated as a blanket rule (2026-11-14)

Chris, main chat, Sat Nov 14, 2026 (~23:05 PT), after the Hermes
Agent audit (MIT, passed) and his reading of Hermes's security
policy — whose line he endorsed after reading its reasoning:
"The only security boundary against an adversarial LLM is the
operating system."

His statement, verbatim: "I know that every AI that runs on
Trinity needs to have every action that acts against the system
gated by the approval process we mapped before propose sign off
implement."

Read against this file: the three review questions set the
ceiling, the trail, and the revocation path; this addendum
fixes the *sequence* for any action that acts on the system —
**propose → sign off → implement** — as a gate no agent action
crosses ungated, not a per-integration option. The Hermes line
supplies the enforcement half from below (only the OS boundary
holds against an adversarial model); the approval process is
the authority half from above. Neither half substitutes for
the other.

## Settled — delegation (Chris, 2026-11-23)

The parked question was: may an agent holding a token delegate a
narrower token to another agent, and what does the evidence trail
need to show for the chain to remain auditable end to end?

**Answer: no delegation.** An agent holding a token cannot mint or
hand off a narrower token to another agent. Token authority enters
an agent tree at exactly one point — the original grant — and is
never re-granted sideways.

Where an agent spawns sub-agents, the relation is **parent and
child process**, not delegation:

- The child holds **no token of its own**. It acts inside the
  parent's authority and under the parent's ceiling; nothing the
  child can do exceeds what the parent itself could do.
- The parent remains the accountable holder. Every child action
  lands in the evidence trail under the parent's token, recorded
  as the child's act on the parent's instruction — the trail is a
  tree rooted at one grant, not a chain of re-grants.
- Revocation stays mechanical: revoking the parent's token ends
  the whole tree. There is no graph of delegated holders to hunt
  down, because delegation never created one.

The reasoning is the auditability axiom applied to authority: a
delegation chain launders provenance — each hop blurs "on whose
instruction" a little further, and the trail degrades from a record
into a chain of claims. Parent/child keeps one holder, one ceiling,
and one revocation point per tree, which is what makes the three
review questions above answerable end to end.

## Open questions (parked, not answered)

- Ceiling granularity: per-agent, per-task, per-session — or all
  three nested?
- Approval UX at the operator's worst moment: how does the
  judgment-layer review stay real (not a rubber stamp) when the
  operator is exhausted? Trinity is designed to survive its
  operator at his worst; the approval path must be too.
