# SYNTHESIS: EXIT Primitive Architecture

> Produced by architect (cellar_door/crew) for cd-1zc.1
> Source materials: EXIT.txt, docs/perplexity_search.md, docs/peace_through_commerce.txt, docs/exit_moltboo_feb5.txt

---

## What EXIT Is (One Paragraph)

EXIT is a verifiable transition marker — the authenticated declaration of departure that preserves continuity across contexts. It is a boundary-crossing event, not a storage system, not a transport layer, not an enforcement mechanism. When an entity (human or agent) leaves a coordination regime, EXIT produces a recognized, non-custodial proof that says: *"I am still me. I am just no longer here."* That proof allows identity, reputation, and participation eligibility to persist elsewhere without the departing entity surrendering custody of any of those things to the system being left. EXIT is the doorway, not what you carry through it.

---

## The 5 Things EXIT Is NOT (And Why Each Exclusion Matters)

### 1. EXIT is not identity.
EXIT does not store, issue, or maintain identity. Primitives like NAME (identity persistence) and DID systems handle that. If EXIT tried to own identity, it would become a custodial identity provider — exactly the kind of lock-in it exists to escape. EXIT only *marks the crossing* so that identity can be re-established elsewhere.

### 2. EXIT is not storage.
EXIT does not hold data, memory, context, or state. It references state (via hashes, snapshots, receipts) but never custodies it. If EXIT stored data, it would become a dependency — a system you can't leave without losing what's inside it. That would make EXIT self-defeating.

### 3. EXIT is not transport.
EXIT does not move assets, migrate data, or handle the logistics of relocation. Migration tooling, bridge protocols, and asset transfer mechanisms are downstream services. EXIT produces the *marker* that authorizes and authenticates a transition; the actual movement happens elsewhere. Conflating EXIT with transport would balloon scope into a full migration platform.

### 4. EXIT is not enforcement.
EXIT does not slash stakes, void contracts, or punish violations. It does not adjudicate disputes or impose penalties. The peace-through-commerce materials show that enforcement belongs to incentive structures, clearinghouses, and arbitration layers. EXIT sits below enforcement — it is the *escape hatch* that makes enforcement regimes tolerable, not an enforcement regime itself.

### 5. EXIT is not reputation.
EXIT does not score, rank, or maintain reputation. Reputation receipts, portable credentials, and trust attestations are adjacent primitives (MARK, verifiable credentials). EXIT enables reputation *portability* by providing the authenticated departure proof that reputation systems can reference — but the reputation itself lives elsewhere. If EXIT owned reputation, leaving would mean trusting EXIT with your history.

---

## Key Insights from the Competitive Landscape

### What exists
- **Identity/VC layer (enablers, not competitors):** DID/VC wallets (Affinidi portable reputation app, cheqd verifiable AI credentials, generic W3C VC implementations) make identity and credentials portable but don't prescribe exit workflows.
- **Reputation layer (adjacent):** "Portable reputation" via VCs for businesses and users. Usually stops at "you can carry this credential" — no ceremony for *how you leave*.
- **Recovery layer (adjacent):** Account abstraction recovery (EIP-7947 AARI), guardian-based recovery, smart wallet UX. Addresses continuity of *control*, not exit.
- **Asset exit formats:** Statechannels' "Standard Exit Format for L2s" defines interoperable asset withdrawal structures. DAO ragequit (Moloch pattern) implements minority-protection exit with economic finality.
- **Exodus Protocol framing:** Blockchain Commons' "preserve exit through portability" philosophy is the closest conceptual neighbor — exit as a named design principle.
- **Agent lineage research:** Agent Identity URI schemes (agent://), context lineage for non-human identities, Merkle-based provenance — these provide the continuity proofs EXIT would reference.

### What's missing (the gap)
- **Exit as a multi-step ritual** with defined roles, state machine, and explicit semantics for: pre-exit commitments, portable receipt generation, and dispute-aware evidence.
- **Coupling** between VC-style portable credentials, account-abstraction continuity, and agent-identity lineage into a single coherent ceremony.
- **Offline/long-lived verification** — the ability for third parties to verify exit receipts without the original platform being live.
- **Agent-native exit** — existing work treats exit as human account migration; almost nothing models exit for autonomous agents who may fork, succeed, or hibernate.

---

## The Unique Gap Cellar Door Fills

Cellar Door occupies the intersection that nobody else has claimed: **exit as a first-class, agent-native ceremony with portable, third-party-verifiable receipts, without custody of assets, identity, or content.**

Specifically:

1. **Ceremony, not credential.** Others issue portable credentials. Cellar Door defines the *ritual of departure* — the state machine, the roles, the receipt schema, the verification model. The credential is an output; the ceremony is the primitive.

2. **Agent-native from day one.** The competitive landscape treats exit as a human UX problem (account migration, data portability). Cellar Door starts with autonomous agents who face harder versions of the same problem: context loss across reboots, successor recognition, lineage continuity, identity persistence without a human to click "export." The Moltbook discussion makes this vivid — agents are *already* losing themselves across session boundaries and converging on ad-hoc solutions. EXIT is the primitive they're missing.

3. **Non-custodial by definition.** The EXIT primitive explicitly refuses to hold anything. This is a feature: it means Cellar Door can never become the lock-in it exists to prevent. It's the doorway, not the building — and doorways don't hold hostages.

4. **Stabilizing infrastructure, not ideology.** The peace-through-commerce analysis reveals that EXIT is structurally identical to the "credible exit" mechanism that bounds power in multi-agent coordination regimes. When exit is credible, authoritarians can't overreach, insurers can price risk, and participation becomes genuinely voluntary. Cellar Door is building the constitutional layer — not a product feature, but the primitive that makes all other coordination non-coercive.

---

## Open Questions and Tensions

### Architectural
- **How minimal can the EXIT data structure be?** The perplexity search cataloged 9 categories of information people expect in a general exit primitive (identity, state snapshot, exit type, evidence bundle, lineage, credentials, economics, time/finality, narrative). What is the irreducible core vs. optional modules?
- **Ceremony state machine:** How many states? What transitions? Who are the roles (exiting entity, origin system, witnesses, verifiers)? Is there a challenge period, or is exit immediate?
- **Verification without origin online:** What cryptographic material must the exit receipt carry so that a third party can verify it years later without the origin platform being live?

### Identity/Agent
- **Successor semantics:** When an agent exits and a successor appears, what constitutes valid succession? Key rotation? Lineage hash chain? Delegation token? How does this interact with the "Ship of Theseus" problem agents already face across context resets?
- **Partial exit:** Can an entity exit some relationships but not others? (Leave DAO X, stay in marketplace Y, retain reputation from both.) How does the primitive handle scope?

### Ecosystem
- **Who issues the receipt?** Self-issued (the exiting entity signs their own departure) vs. co-signed (the origin system countersigns) vs. witnessed (a third party attests). Each has different trust and liveness properties.
- **Standards alignment:** Should EXIT receipts be W3C Verifiable Credentials? Or is that premature coupling to a specific envelope format?
- **The "peace protocol" connection:** The peace-through-commerce materials position EXIT as the constitutional primitive that bounds power in agent economies. Is this aspirational framing or does it impose concrete design constraints (e.g., EXIT must be uncensorable, EXIT must work under adversarial conditions, EXIT must resist capture)?

### Practical
- **What triggers an EXIT?** Voluntary departure only? Or also forced exit (slashing, expulsion, system shutdown)? Do both produce the same receipt?
- **Latency vs. finality:** Is EXIT instant or does it require a challenge window? The L2 exit world assumes challenge periods; the agent world may need immediate exits (context about to be destroyed).
- **Privacy:** How much does the exit receipt reveal? Can you prove "I left in good standing" without revealing your full history? ZK-based selective disclosure is mentioned in the landscape but adds significant complexity.
