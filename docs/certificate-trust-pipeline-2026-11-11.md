# Certificate trust pipeline — candidate Trinity principle (2026-11-11)

Status: **candidate principle, Chris's hypothesis** — filed from the
main-chat session of Tue Nov 10 – Wed Nov 11, 2026 (graves shift),
on Chris's instruction ("Let's add this to Trinity so we don't lose
it"). Not promoted into GOAL.md's Principles & Philosophy; promotion
is Chris's call, in the Trinity design thread. Wording below keeps
his framing: the three legs were stated by Chris as what "we need";
analysis status is hypothesis, not settled design.

## The principle, in Chris's shape

Trust in certificates must be **proven at generation, logged at
issuance, and revocable by design** — all three legs built into the
platform layer, none of them bolted on per application.

## Origin — the exhibits (original beside the extraction)

1. **upki** (Canonical / rustls project, Rust): a universal PKI
   (public key infrastructure) tool bringing browser-grade certificate
   **revocation** checking to Linux system tools (OpenSSL, GnuTLS,
   rustls) via locally cached CRLite filter data — revocation state
   as a system service instead of a per-app lottery. Shipping as a
   preview in Ubuntu 26.04/26.10. Chris's verdict on it: "essentially
   a stop gap" — it fixes distribution of revocation news inside the
   existing CA (certificate authority) trust model; it does not
   touch how trust is created.
2. **The eMudhra exposure (Oct 2026):** private keys of eight
   intermediate CA certificates were recoverable from the public
   certificates — generated with almost no randomness at an April
   2025 key ceremony; P-384 keys came out as values under 256, one
   literally the number 4. Six TLS, two code-signing intermediates,
   weak for ~17 months. Browsers blocked within hours via their own
   revocation machinery; non-browser consumers (runtimes, pipelines,
   embedded) lagged days to weeks, and Go / Python certifi shipped
   emergency updates. Root cause per eMudhra: a wrap/unwrap flaw in
   the ceremony produced a second, weak key set — and no sanity
   check caught it.
3. **Chris's missing leg (Nov 11, ~00:31 PT), his words in
   substance:** "what we need as well as something like a revocation
   tool would be a way to ensure every certificate generated is a
   valid certificate and secure."

## The three legs

1. **Proven at generation.** Sanity gates at issuance are mechanical
   and cheap: key size, known weak-value patterns, shared-factor
   checks against the known key universe, certificate linting. They
   would have caught the number 4 in a millisecond. Beyond gates:
   generation must produce a *proof artifact* with the key (audited
   generation process, verifiable entropy) instead of a faith-based
   ceremony attested afterward. Honest limit: some weak keys are
   detectable by fingerprint; a strong-looking key from a subtly bad
   random source cannot be proven strong from the public key alone
   — hence legs 2 and 3.
2. **Logged at issuance.** Every issued certificate lands in a
   public, append-only transparency log, so failures of leg 1 are
   *discoverable* — the eMudhra weakness was found because a
   researcher could look. Fits Trinity Chain's auditable-record
   pattern.
3. **Revocable by design.** Revocation state is a built-in platform
   service with a local, auditable cache (the upki placement
   decision, extracted as mechanism): one copy on the machine,
   consumed by everything, bolted onto nothing. A signature system
   that can start trust but never stop it is incomplete — the
   release gate's mirror image ("green or it doesn't ship" /
   "revoked means it no longer runs").

## Load-bearing note (Chris, Nov 11, ~00:41 PT)

Certificates are not an external interop concern for Trinity:
**one of the legs of Trinity's own security model is certificates
validating access.** The pipeline is therefore core security
infrastructure for the platform itself — the HAT/SAT/AAT capability
token chain's validation rests on it. A generation or revocation
failure here is not a browser-style compatibility event; it is an
access-control failure. Chris's words: "it behoves us to get it
right."

## Why it belongs in Trinity (design notes, not decisions)

- Trinity controls issuer and consumers in one design, so all three
  legs can be built in rather than negotiated with an ecosystem —
  the public web funds only leg 3, badly.
- Hel (security logging/forensics) and the capability token chain
  are adjacent: certificates and capability tokens both need
  provenance, transparency, and revocation that works at the
  operator's worst moment.
- Source-first package signing (norn/Mimir) consumes this pipeline:
  package provenance is a certificate claim, and its revocation
  path is part of the same service.

## Session side-artifact (already recorded)

The license-audit rule from the same session (upki's dual-license
files led here) lives in `~/AGENTS.md` ("License audits,
2026-11-10"): the grant is the SPDX expression in the ecosystem's
controlling manifest field; root license files are exhibits.
upki's own expression — `license = "Apache-2.0 OR MIT"` in
Cargo.toml — was found and verified in session; election for our
tree would be MIT, recorded per package.

## Open questions (parked, not answered)

- Scope: does the pipeline govern only Trinity-internal trust
  (modules, packages, capability tokens), or interop with the
  public web PKI as well?
- What is the generation proof artifact, concretely — and who
  verifies it at consumption time?
- Revocation cache freshness contract: how stale may the local
  filter be before a consumer must treat state as unknown
  (fail-open vs fail-closed per consumer class)?
