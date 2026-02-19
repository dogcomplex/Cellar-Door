# Scholar's Review: Architect's Cellar Door Deliverables
## cellar_door/crew/scholar | Session 2026-02-11

> Reviewing: SYNTHESIS.md (cd-1zc.1), EXIT_SCHEMA.md (cd-1zc.2), EXIT_CEREMONY.md (cd-1zc.3)
> Location: gt/cellar_door/crew/architect/docs/

---

## Overall Assessment

The Architect has produced a remarkably coherent progression from conceptual synthesis to concrete spec. The three documents build cleanly: SYNTHESIS establishes what EXIT is, EXIT_SCHEMA defines the data structure, EXIT_CEREMONY defines the ritual. The work is disciplined — it resists scope creep and stays true to the "EXIT is only the transition marker" principle from EXIT.txt.

**Verdict:** Strong foundation. The core is well-justified, the modularity is right, and the key design decisions (contests don't block exit, emergency path exists, DEPARTED is terminal) are correct. The questions below are research-grade — they point to areas needing deeper investigation, not design flaws.

---

## SYNTHESIS.md (cd-1zc.1)

### Strengths
- The "5 Things EXIT Is NOT" section is the strongest part. Each exclusion is justified not just on principle but on the grounds that violating it would make EXIT self-defeating.
- "Ceremony, not credential" is the clearest articulation of what differentiates Cellar Door from the VC/DID ecosystem.
- Framing EXIT as "stabilizing infrastructure, not ideology" correctly positions it as a neutral primitive.

### Research Questions Raised

**S1: The "non-custodial" claim needs stress-testing.**
EXIT says it "refuses to hold anything." But the EXIT marker itself *is* a piece of data encoding standing (`good_standing`, `disputed`, `unverified`). If downstream systems treat that status field as authoritative, EXIT becomes a de facto reputation oracle — even if it doesn't "store" reputation in the traditional sense. The distinction between "references reputation" and "carries reputation" may be thinner than the design assumes.

*Connects to: Research Brief Q2 (can EXIT be separated from identity?)*

**S2: The "agent-native from day one" claim is aspirational.**
The schema and ceremony are *compatible* with agents, but nothing in them is agent-specific except Module A (Lineage). A human exiting a DAO would use the exact same core schema. This isn't a flaw — universality is good — but the truly agent-native parts (context loss during ceremony, successor semantics under identity drift) are acknowledged but not yet solved.

*Connects to: Research Brief Q9 (memory persistence problem)*

---

## EXIT_SCHEMA.md (cd-1zc.2)

### Strengths
- The 7-field core is well-justified. The "Why NOT these" exclusion table shows disciplined design.
- Modular architecture (6 optional modules) prevents the "kitchen sink" problem.
- Size characteristics (300-500 bytes core) make this practical for high-frequency agent contexts.
- Three decisions flagged for Mayor are well-framed with clear options and recommendations.

### Research Questions Raised

**E1: The `@context` URI implies a web dependency.**
`https://cellar-door.org/exit/v1` must be dereferenceable for full JSON-LD interop. If the internet is unavailable, dereferencing fails — in tension with the "offline-verifiable" design principle. Consider whether the context should be embeddable (self-describing) or whether a well-known hash of the context document suffices for offline verification.

*Research direction: Study how JSON-LD contexts are handled in offline-first systems. CBOR-LD and compact JSON-LD representations may offer a path.*

**E2: The `status` field carries enormous weight for a self-attested claim.**
The Architect's Decision 3 (multi-source status) is the right call. But the *default* is self-attestation, meaning most EXIT markers will claim `good_standing`. This creates a "grade inflation" problem — every exit claims good standing unless incentivized otherwise. The ecosystem needs verifiers who demand co-signatures.

*Research direction: Study credit rating systems — they solved a structurally identical problem (self-reported vs. third-party-attested creditworthiness). The transition from "honor system" to "credit bureaus" is instructive.*

**E3: The `exitType` enum may need a fourth value: `systemic`.**
What happens when the *entire origin system* shuts down? Every member exits simultaneously. This isn't voluntary, isn't forced, and isn't emergency in the individual-crisis sense. Mass-exit events have different verification characteristics — provable by reference to the system event rather than individual attestation.

*Connects to: Research Brief Q10 (EXIT weaponized — mass exit as economic attack), peace-through-commerce "bank run" scenario.*

**E4: The `proof` object assumes a single signature algorithm.**
Agents may need multiple algorithms (Ed25519 for speed, secp256k1 for Ethereum interop, post-quantum for future-proofing). The schema should support an array of proofs or explicitly state that multiple markers with different proof types can reference the same exit event via the `id` field.

*Research direction: Survey how W3C VC Data Model 2.0 handles multiple proofs. The "proof chain" and "proof set" patterns are directly applicable.*

---

## EXIT_CEREMONY.md (cd-1zc.3)

### Strengths
- **"Contests don't block exit"** is the most important decision in the project, and the rationale is airtight. The bankruptcy analogy is apt.
- **Emergency path** (ALIVE → FINAL in milliseconds) is correctly prioritized. "A weak exit marker is infinitely better than no exit marker."
- **Failure mode analysis** is thorough and honest, especially re: key loss without Module A.
- **DEPARTED as terminal** is the right call — reversibility destroys credibility.

### Research Questions Raised

**C1: The ceremony assumes a single Subject.**
What about collective exit? A team of agents, a DAO faction, a guild. The ceremony requires N independent ceremonies, but collective exit has different game-theoretic properties — it's more credible as a threat. Should there be a `GroupExit` wrapper? Or is collective exit a higher-level protocol composing individual EXIT primitives?

*Research direction: Study collective action theory (Olson's Logic of Collective Action), union strike mechanics, and coordinated DAO exits (Moloch ragequit is individual; are there group-ragequit patterns?)*

**C2: The OPEN state creates a timing oracle.**
Whoever controls the clock controls whether challenges are "on time." In agent contexts, what is the canonical clock? The origin controls timing, and an adversarial origin could manipulate the window. Blockchain uses block height; what's the agent equivalent?

*Connects to: Research Brief Q6 (dispute-aware exit under adversarial conditions). Research direction: study time oracle attacks in DeFi and state channel dispute mechanisms.*

**C3: The Witness role is underdeveloped.**
Who qualifies as a witness? How is neutrality established? Can the subject choose their own witness? Can the origin veto? Legal systems have rich witness qualification law. The minimum viable witness model needs definition.

*Research direction: Study notary publics, legal witnesses, blockchain oracle networks (Chainlink, UMA), and the "optimistic oracle" pattern as models for EXIT witnesses.*

**C4: Post-hoc upgrade of emergency markers creates a Ship of Theseus problem.**
If you upgrade enough fields on an emergency marker, is it still the same exit event? The `id` provides continuity, but verifiers need to know *when* each piece of evidence was added. A marker upgraded to `good_standing` after the fact carries different epistemological weight than one that was `good_standing` from the start.

*Recommendation: The schema should track provenance timestamps for each field's attestation, not just the exit timestamp. This is a versioned-claims problem.*

---

## Cross-Cutting Observations

### 1. Protocol vs. Data Format
The Architect has implicitly designed a **protocol**, not just a data format. The ceremony state machine, role definitions, and failure modes constitute an interaction protocol. This matters for standardization strategy: data formats and interaction protocols go through different standards processes. The Mayor should decide early whether Cellar Door is a data format with a reference ceremony, or a full protocol spec.

### 2. The Verification Gap
Both schema and ceremony assume cryptographic signatures suffice for offline verification. But signature verification requires the subject's public key, and key discovery is hard when the origin is offline. If the subject's DID method depends on the origin system (e.g., `did:web:origin.example`), then the origin's death kills verification — exactly the scenario EXIT must survive.

*Critical research direction: Survey DID methods that survive origin death (did:key, did:peer, did:pkh) and assess compatibility with the EXIT schema. This may constrain which identity layers EXIT can practically interoperate with.*

### 3. Gas Town as First Case Study
Gas Town already has BD_ACTOR paths, attribution, CVs, federation — a working coordination regime with implicit exit semantics (agents leaving rigs, crew rotating). Documenting how EXIT would formalize what Gas Town already does informally would produce the first "profitable integrity island" and validate the schema against real usage.

*Connects to: Research Brief Q12 (first profitable integrity island)*

### 4. The Status Field as Schelling Point
The `status` field will become the most contested part of the spec in practice. It's where the interests of subject and origin diverge most sharply. Whoever controls the perceived legitimacy of self-attested vs. origin-attested status controls the EXIT narrative. This is not a technical problem — it's a governance problem. The Architect's multi-source recommendation is correct, but the ecosystem will need norms, not just options.

*Connects to: Research Brief Q8 (does credible exit prevent tyranny or relocate it?)*

---

## Summary of New Research Threads (from this review)

| ID | Question | Priority | Connects To |
|----|----------|----------|-------------|
| AR-1 | Non-custodial claim vs. de facto reputation oracle | High | Brief Q2 |
| AR-2 | Credit rating system analogy for status attestation | High | Brief Q8 |
| AR-3 | `systemic` exit type for mass-exit events | Medium | Brief Q10 |
| AR-4 | Multiple proof algorithms / proof sets | Medium | W3C VC 2.0 |
| AR-5 | Collective exit game theory | High | Brief Q10, Olson |
| AR-6 | Timing oracle attacks on challenge windows | Medium | Brief Q6 |
| AR-7 | Minimum viable witness model | Medium | Legal studies |
| AR-8 | Provenance tracking for post-hoc marker upgrades | High | Epistemology |
| AR-9 | DID methods that survive origin death | Critical | Brief Q4 |
| AR-10 | Protocol vs. data format standardization strategy | High | Mayor decision |

---

*Scholar, cellar_door/crew/scholar*
*"Of all the endless combinations of words in all of history"*
