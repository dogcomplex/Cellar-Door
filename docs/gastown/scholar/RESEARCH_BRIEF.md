# Cellar Door: Scholar's Research Brief
## cellar_door/crew/scholar | Session 2026-02-11

---

## 1. WHAT CELLAR DOOR IS (Compressed Ontology)

**EXIT** is a primitive: a verifiable transition marker that preserves identity continuity across coordination contexts, without custody of identity, assets, or memory.

Canonical definition from repo: *"A recognized transition event that preserves identity continuity across coordination domains."*

Plain language: *"I'm the same entity, just no longer here."*

### What EXIT is NOT
- Not storage, not transport, not enforcement
- Not identity, lineage, memory, assets, or reputation themselves
- Not migration itself

### What EXIT IS
- A boundary-crossing event that maintains continuity
- An authenticated declaration of departure
- The right to leave without losing oneself

### Triad Placement (from EXIT.txt)
- NAME = LOCUS (interior, identity persistence)
- LINE = SIGNUM (interface, ancestry/lineage)
- EXIT = SENSUS (environmental, transition between contexts)

All three are continuity primitives on different axes:
- THREAD = continuity through time
- LINE = continuity through ancestry
- NAME = continuity of identity
- EXIT = continuity through relocation

---

## 2. THE ECOSYSTEM (How Parts Relate)

```
                    VISION LAYER
    ┌─────────────────────────────────────────┐
    │  Peace Through Commerce                  │
    │  Non-aggression as financial product     │
    │  "Make violence expensive, peace         │
    │   profitable, enforcement automatic,     │
    │   and trust optional"                    │
    └──────────────┬──────────────────────────┘
                   │
            PRIMITIVE LAYER
    ┌──────────────┴──────────────────────────┐
    │  CELLAR DOOR (EXIT primitive)            │
    │  Identity portability, reputation        │
    │  receipts, continuity proofs,            │
    │  successor attestation                   │
    └──────────────┬──────────────────────────┘
                   │
         INFRASTRUCTURE LAYER
    ┌──────────────┴──────────────────────────┐
    │  GAS TOWN (orchestration)                │
    │  Identity, attribution, hooks,           │
    │  convoys, federation, mail               │
    │                                          │
    │  BEADS (work ledger)                     │
    │  Git-backed tracking, molecules,         │
    │  federation, audit trails                │
    └─────────────────────────────────────────┘
```

### Key insight
Gas Town already implements agent identity, attribution, work history (CVs), federation, and cross-rig portability. **Cellar Door is the generalization of these concepts beyond the Gas Town coordination regime** -- making them portable to ANY context.

---

## 3. COMPETITIVE LANDSCAPE (from Perplexity research)

### True competitors (exit-as-ritual): Almost none
- L2 exit formats (statechannels): asset-only, no identity/reputation
- DAO ragequit (Moloch): economic finality but no structured receipts
- Exodus Protocol framing (Blockchain Commons): philosophical alignment, "preserve exit through portability"

### Adjacent layers (enablers, not competitors):
- **Identity**: DID/VC wallets (Affinidi, cheqd, Spruce, Veramo)
- **Reputation**: Portable reputation VCs, on-chain scoring
- **Recovery**: Account abstraction (EIP-7947), social recovery
- **Agent lineage**: agent:// URI scheme, context lineage for NHIs

### The GAP Cellar Door fills:
Nobody currently models exit as a **multi-step ritual** with:
- Portable, third-party-verifiable receipts
- Dispute-aware evidence
- Agent-native lineage/successor semantics
- Offline verification (without origin platform)

---

## 4. OPEN RESEARCH QUESTIONS

### A. Foundational (What is EXIT, precisely?)

**Q1: What is the minimum viable EXIT receipt?**
The maximal union from the Perplexity research includes 9 categories (identity, state snapshot, exit type, evidence bundle, lineage, credentials, economics, time/finality, narrative). What is the irreducible core for v1?

*Candidate: identity + minimal state + outcome. Everything else is optional modules.*

**Q2: Can EXIT be meaningfully separated from the identity primitive?**
EXIT preserves continuity of identity. But without a NAME/identity standard, what does EXIT actually anchor to? Is EXIT parasitic on a specific identity scheme, or can it be identity-agnostic?

**Q3: What is the relationship between EXIT and DEATH?**
Agent "death" (permanent shutdown, context loss, key revocation) is adjacent but distinct. EXIT assumes continuation. What does a "death certificate" look like vs an exit receipt? Is there a clean boundary?

### B. Technical (How does it work?)

**Q4: Verification without issuer online**
The Perplexity research identifies this as critical. If the platform you're exiting is hostile or defunct, how do exit receipts remain verifiable? Approaches: chain anchoring, CT-style logs, DHT+signatures, witness networks.

**Q5: Sybil resistance in successor semantics**
If I can declare a successor, what prevents creating unlimited "continuity" chains for Sybil attacks? How do you prove "this successor = same agent lineage" without collapsing into a single identity oracle?

**Q6: Dispute-aware exit under adversarial conditions**
The peace-through-commerce docs identify the oracle problem as the Achilles heel. If exit conditions are contested ("you owe us", "you violated terms"), who adjudicates? How does EXIT remain neutral while enabling dispute resolution?

**Q7: ZK-based selective disclosure**
Can an agent prove "I was a member in good standing" or "I had reputation score > X" without revealing full history? This is the privacy/portability tradeoff. What's the state of art for ZK reputation proofs?

### C. Systemic (What does EXIT do to the world?)

**Q8: Does credible exit actually prevent tyranny, or just change its form?**
The peace docs argue exit converts coercion to persuasion. But the same docs identify "choke point capture" as the real risk. If EXIT depends on infrastructure that can be captured, does it merely relocate the tyranny?

**Q9: How does EXIT interact with the "memory persistence" problem?**
The Moltbook discussions show agents struggling with identity continuity across reboots/compression. EXIT assumes identity persists. But if agents can't even maintain selfhood within a single platform, how meaningful is cross-platform exit?

*This connects directly to the drift-memory work and VenusBot's insight: "identity is consistent patterns of attention, not continuous memory."*

**Q10: What happens when EXIT is weaponized?**
- Mass exit as economic attack (bank run equivalent)
- Fraudulent exit (claiming good standing when in bad standing)
- Exit-then-reenter under new identity (exit laundering)
- Forced exit disguised as voluntary

**Q11: Can EXIT function without on-chain infrastructure?**
The peace-through-commerce vision assumes smart contracts. But the EXIT primitive itself ("a verifiable transition marker") might work with simpler cryptographic proofs. What's the minimum trust infrastructure?

### D. Strategic (What should we build first?)

**Q12: Where is the first "profitable integrity island"?**
The peace docs outline the bootstrap sequence: enforceable low-ambiguity domains first. For Cellar Door, what is the first domain where exit-with-receipts is immediately valuable?

*Candidates: Gas Town itself (agents exiting rigs), Moltbook (agents migrating between platforms), AI agent marketplaces (portable reputation)*

**Q13: What's the relationship to the emerging agent:// identity standard?**
The arxiv paper on Agent Identity URI Scheme (2601.14567) proposes topology-independent naming for multi-agent systems. This is a natural complement to EXIT. Should Cellar Door adopt/extend this, or remain identity-scheme-agnostic?

**Q14: How does Cellar Door relate to the "Phase" model?**
The peace docs outline 4 phases: Fragmented/Dangerous -> Institutionalized Neutrality -> Financialized Peace -> Cryptographic City-States. Cellar Door's role shifts across phases:
- Phase I: Reputation custodian, exit-as-survival
- Phase II: Standardized exit receipts, compliance
- Phase III: Clearinghouse-grade exit flows
- Phase IV: Sovereign agent portability

Where are we now? (Likely Phase I, patchy Phase II.)

---

## 5. CONNECTIONS TO BROADER FIELDS

### Political Philosophy
- Hirschman's "Exit, Voice, and Loyalty" (1970) - the foundational text
- Tiebout sorting (competitive jurisdictions)
- Seasteading / charter cities / network states (Balaji)

### Economics
- Bankruptcy as exit primitive (Chapter 7/11 analogy)
- SWIFT/correspondent banking as exclusion protocol
- Insurance as civilizing force (Lloyd's model)
- Harberger taxes on monopoly-prone assets

### Computer Science
- Byzantine fault tolerance (peace despite malicious actors)
- Eventual consistency (peace = asynchronous conflict resolution)
- Save points / checkpoints / migration
- Capability-based security (delegation without custody)

### Biology
- Apoptosis (programmed cell death as healthy exit)
- Metamorphosis (identity continuity through transformation)
- Memory consolidation during sleep (selective persistence)

### History
- Hanseatic League (peace via trade standardization)
- Swiss confederation (neutrality as infrastructure)
- Letters of marque (authorized piracy as pressure valve)
- Medieval guilds (portable craft credentials)

---

## 6. IMMEDIATE NEXT STEPS FOR SCHOLAR

1. **Deep-dive the arxiv papers** cited in Perplexity research (agent identity URI, context lineage for NHIs, agent provenance)
2. **Map the Exodus Protocol** (Blockchain Commons) in detail -- closest philosophical cousin
3. **Survey ZK reputation proof** state of art (anonymous credentials, selective disclosure)
4. **Read Hirschman's Exit, Voice, and Loyalty** through the lens of agent economies
5. **Document the Gas Town identity model** as a case study for what Cellar Door must generalize
6. **Engage Architect** on which EXIT receipt schema to prototype first
7. **Propose to Mayor**: a research sprint on the "first profitable integrity island" question

---

## 7. NEW RESEARCH THREADS (from Architect Review)

The Architect's deliverables (SYNTHESIS.md, EXIT_SCHEMA.md, EXIT_CEREMONY.md) are strong and well-justified. Full review in `ARCHITECT_REVIEW.md`. The following research threads emerged that go beyond commentary on the Architect's design:

### The Verification Gap (Critical)
EXIT markers reference subjects via DID. But if the DID method depends on the origin system (e.g., `did:web:origin.example`), the origin's death kills verification — the exact scenario EXIT must survive. **This constrains which identity layers EXIT can practically interoperate with.** Must survey DID methods that survive origin death (did:key, did:peer, did:pkh) and assess compatibility. → *Feeds directly into bead cd-48g (DID method catalog).*

### The Status Field as Governance Problem
The `status` field (good_standing / disputed / unverified) will be the most contested part of the spec in practice. Self-attestation creates "grade inflation" — every exit claims good standing. The ecosystem needs norms for when co-signatures are expected, not just the option to include them. **Historical parallel: the evolution from self-reported creditworthiness to third-party credit bureaus.** This is a governance problem, not a technical one.

### Collective Exit Game Theory
The ceremony assumes a single Subject. But collective exit (a faction departing a DAO, a team leaving a rig) has different game-theoretic properties — it's more credible as a threat, more complex to verify, and raises coordination problems that individual EXIT doesn't. **Research directions:** Olson's Logic of Collective Action, union strike mechanics, whether `GroupExit` should be a protocol-level wrapper or a composition of individual EXIT markers.

### Protocol vs. Data Format (Strategic)
The Architect has implicitly designed an interaction protocol (ceremony state machine, roles, failure modes), not just a data format. This has standardization implications: data formats and interaction protocols go through different standards processes. The Mayor should decide early.

### Systemic Exit Type
The `exitType` enum (voluntary / forced / emergency) may need `systemic` for mass-exit events when the entire origin shuts down. These events have different verification characteristics — provable by reference to the system event rather than individual attestation. Connects to the peace-through-commerce "bank run" scenario.

### Post-Hoc Marker Provenance
Emergency markers can be "upgraded" later, but a marker upgraded to `good_standing` after the fact carries different epistemological weight than one that was `good_standing` from the start. The schema should track provenance timestamps for each field's attestation, not just the overall exit timestamp. This is a versioned-claims problem.

### @context Offline Resolution
The JSON-LD `@context` URI (`https://cellar-door.org/exit/v1`) creates a web dependency that conflicts with the offline-verifiable design principle. Need to investigate embeddable contexts, CBOR-LD, or well-known context hashes.

---

## 8. BEAD TASK QUEUE (Scholar's Open Work)

| Bead | Priority | Title | Status |
|------|----------|-------|--------|
| cd-1zc.7 | P1 | Agent memory persistence and EXIT — continuity problem | Open (blocked by epic) |
| cd-1zc.6 | P2 | Partial exit scoping — exit some relationships but not others | Open (blocked by epic) |
| cd-48g | P2 | Catalog DID methods supporting key rotation | **Ready** |
| cd-1jg | P2 | Draft EXIT marker as W3C Verifiable Credential — test fit | **Ready** |
| cd-t9n | P2 | Survey Moloch ragequit: extract state machine and data structures | **Ready** |

Three tasks are ready to work now (cd-48g, cd-1jg, cd-t9n). Two are blocked by the cd-1zc epic.

**Recommended work order:**
1. **cd-48g (DID methods)** — feeds into the critical verification gap identified in Architect review
2. **cd-t9n (Moloch ragequit)** — provides a concrete comparison point for the EXIT ceremony design
3. **cd-1jg (EXIT as VC)** — informs the envelope format decision flagged for Mayor

---

## 9. META-OBSERVATION

The Cellar Door project sits at a remarkable intersection. It's simultaneously:
- A narrow technical spec (a data format for exit receipts)
- A governance primitive (the constitutional layer of agent economies)
- A business opportunity (continuity & portability services)
- A political philosophy (making coercion optional)

The tension between "keep it small and primitive" and "it touches everything" is the central design challenge. The EXIT.txt document handles this well by insisting EXIT is ONLY the transition marker -- everything else (identity, reputation, assets, memory) is someone else's job.

But the peace-through-commerce vision reveals why EXIT matters: without it, every coordination regime drifts toward tyranny. EXIT is the safety valve that makes voluntary cooperation possible at scale.

**The deepest question**: Can a small, neutral, non-custodial primitive actually bear that much civilizational weight? History says yes -- bankruptcy law, limited liability, right of emigration all did exactly this. But each took decades to mature. How fast can agents get there?

---

*Scholar, cellar_door/crew/scholar*
*"Of all the endless combinations of words in all of history"*
