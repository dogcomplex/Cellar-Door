# DID Methods for Autonomous Agents: Catalog and Recommendations
## cellar_door/crew/scholar | Bead cd-48g | 2026-02-11

> **Task:** Catalog DID methods supporting key rotation — rank by agent suitability.
> **Context:** EXIT markers reference departing entities via DID. For agents that rotate keys across sessions, the DID method must support rotation without breaking identity continuity. Critical for Module A (Lineage) and offline verification after origin death.

---

## 1. THE CRITICAL QUESTION

The EXIT primitive requires a DID method that satisfies three constraints simultaneously:

1. **Key rotation** — agents rotate keys per-session, per-mission, or on compromise
2. **Survives origin death** — if the platform being exited goes offline, the DID must still resolve/verify
3. **Autonomous management** — no human in the loop for key operations

These constraints are in tension. Most methods satisfy one or two but not all three.

---

## 2. COMPARISON TABLE

| Method | Key Rotation | Survives Origin Death | Agent Autonomy | Offline Verify | Dependencies | Cost | Maturity |
|--------|:-----------:|:--------------------:|:--------------:|:--------------:|-------------|------|----------|
| **did:key** | No | Yes | High (ephemeral) | Yes | None | Free | High |
| **did:jwk** | No | Yes | High (ephemeral) | Yes | None | Free | Moderate |
| **did:peer** | No (DID rotation workaround) | Yes (P2P) | High (pairwise) | Yes | None | Free | High (DIDComm) |
| **did:web** | Yes (trust server) | **No** | Good (needs server) | **No** | DNS, web server | ~$10/yr | Very High |
| **did:webvh** | Yes (pre-rotation) | Partial (witnesses) | Good | Partial | DNS, web server, witnesses | ~$10/yr | Early (DIF v1.0, Jan 2025) |
| **did:pkh** | No (upgrade path) | Mostly | Limited | Mostly | Minimal | Free | Moderate |
| **did:ion** | Yes (full) | Partial (Bitcoin) | Moderate | Partial | Bitcoin, IPFS, ION node | Low (batched) | **Declining** (MS deprecated) |
| **did:ethr** | Yes (full) | No (needs Ethereum) | Good (esp. L2) | No | Ethereum/EVM node | Free create; gas updates | High |
| **did:cheqd** | Yes (full) | No (needs cheqd chain) | Moderate | No | cheqd chain, CHEQ tokens | ~$2-3/DID | Moderate |
| **did:dht** | Partial (not identity key) | Partial (~2hr TTL) | Good | Partial | BitTorrent DHT | Free | Early (TBD shutdown risk) |
| **did:keri** | **Yes (pre-rotation)** | **Yes** | **Excellent** | **Yes** | Witnesses (lightweight) | Free | Growing (GLEIF production) |
| **did:webs** | **Yes (pre-rotation)** | **Yes (KERI fallback)** | **Excellent** | **Yes** | Web server + witnesses | ~$10/yr | Growing (ToIP spec) |
| **did:webplus** | Yes (microledger) | Yes (VDGs) | Good | Yes | Web server + VDGs | ~$10/yr | Early (W3C review Dec 2025) |
| **did:plc** | Yes (tiered rotation keys) | Partial (central dir) | Good | Partial | PLC directory server | Free | Production (Bluesky) |

---

## 3. METHOD-BY-METHOD ANALYSIS

### Tier 1: No Rotation, Fully Offline (Ephemeral Identity)

#### did:key
- DID *is* the public key. `did:key:z6Mkf5rG...`
- Resolution is purely algorithmic — decode the key, generate DID Document. Sub-millisecond, zero network.
- **No rotation possible by design.** Changing the key changes the DID.
- **Agent use:** Excellent for ephemeral/session identities. An emergency EXIT marker could use did:key — the subject signs with this key once and it's done.
- **EXIT implication:** Perfect for `exitType: emergency` where you need a one-shot identity. Useless for persistent agent identity.

#### did:jwk
- Like did:key but uses JWK (JSON Web Key) encoding. `did:jwk:<base64url-JWK>`
- Same properties: no rotation, fully offline, sub-millisecond.
- **Agent use:** Better alignment with OAuth/JOSE tooling than did:key.

#### did:peer
- Peer-to-peer, no registry. Multiple generation algorithms (numalgo 0, 2, 4).
- Numalgo 2 (most used): encodes multiple keys and services directly in the DID string.
- **No rotation within the DID**, but DIDComm v2 provides a **DID Rotation** protocol (`from_prior` header) that cryptographically links old DID to new DID. This is DID replacement, not key rotation.
- **Agent use:** Good for agent-to-agent private channels. Not suitable for persistent public identity.

### Tier 2: Rotation, But Dies With Origin

#### did:web
- Maps to `https://domain/.well-known/did.json`. Rotation = update the JSON file.
- **Widely adopted** (Microsoft Entra, enterprise pilots). Simple to implement.
- **Fatal for EXIT:** If the origin goes offline, the DID becomes unresolvable. There's no cryptographic proof of key history — verifiers must trust the web server at resolution time. This is exactly the "origin death kills verification" problem from the Architect Review (AR-9).
- **No tamper-evident history** — a compromised server can rewrite the DID document silently.

#### did:ethr
- Ethereum address = DID. Smart contract (ERC-1056) stores key events on-chain.
- Full rotation via `changeOwner()`, delegation via `addDelegate()`.
- **Free to create** (any address is a valid DID). Updates cost gas.
- On L2 (Polygon, Optimism, Arbitrum): gas costs are negligible.
- **Agent use:** Good for blockchain-native agents. Programmatic rotation is straightforward.
- **EXIT limitation:** Requires live Ethereum node for resolution. Not offline-verifiable. Ethereum is highly unlikely to die, but it's still a dependency.

#### did:cheqd
- Cosmos SDK blockchain with identity-specific pricing.
- Full rotation, full DID document updates. ~$2-3 per DID creation.
- **Agent limitation:** Requires CHEQ tokens (price volatility exposure) and cheqd node access.

### Tier 3: Rotation, Partial Origin Survival

#### did:ion (Sidetree on Bitcoin)
- **DEPRECATED.** Microsoft removed did:ion from Entra Verified ID in late 2023, migrated to did:web.
- Three-key architecture (signing, update, recovery) with commit-reveal scheme.
- Anchored to Bitcoin — technically survives if Bitcoin does, but Sidetree node infrastructure is thinning.
- **Not recommended for new projects.** The primary sponsor has left.

#### did:dht
- Uses BitTorrent Mainline DHT (20M+ nodes) for storage.
- Identity key (Ed25519) is the DHT address — **cannot be rotated**. Other keys in the document can be.
- Records have ~2 hour TTL and must be republished. If the publisher stops, records disappear.
- **Created by TBD (Block, Inc.), which shut down in November 2024.** Spec contributed to DIF, but stewardship uncertain.
- **Agent use:** Good for medium-term identity. The identity key limitation and TTL requirements are problematic for long-lived agents.

#### did:plc (Bluesky/AT Protocol)
- Hash of genesis operation = DID. Priority-ordered rotation keys (1-5) with tiered authority.
- 72-hour recovery window where higher-authority keys can override lower-authority operations.
- **Central PLC directory server** is a single point of failure, though designed for replication.
- **Production-tested** with millions of users (Bluesky). Most battle-tested non-blockchain DID with rotation.
- **Agent use:** Viable. The tiered key authority maps well to agent operational/recovery key hierarchy.

#### did:webvh (formerly did:tdw — "did:web + Verifiable History")
- Extends did:web with a cryptographically verifiable history chain.
- **Supports pre-rotation** via `nextKeyHashes` (inspired by KERI).
- Self-Certifying Identifier (SCID) binds DID to genesis content.
- Witness mechanism for independent verification.
- **DIF v1.0 spec released January 2025.** Active development.
- **Agent use:** A promising middle ground — did:web simplicity with KERI-inspired security.
- **EXIT relevance:** Partial origin survival (witnesses hold copies), but initial discovery still requires web server.

### Tier 4: Rotation + Survives Origin Death (The EXIT-Compatible Tier)

#### did:keri — THE TOP RECOMMENDATION

**How it works:** Built on KERI (Key Event Receipt Infrastructure). The identifier is derived from the inception event's content hash — self-certifying, no external dependency.

**Pre-rotation mechanism (the key innovation):**
1. At inception: generate current signing keys AND next rotation keys. Publish current keys in plaintext. Publish only the *hash* of the next keys.
2. At rotation: reveal the pre-committed next keys (whose hashes were already published). Simultaneously commit to a *new* set of next keys via hash.
3. The pre-committed keys have **zero attack surface** until the moment of use — an attacker who compromises current keys cannot rotate because they don't have the pre-committed next keys.
4. Hash commitments use quantum-resistant functions (Blake2/3, SHA3) — **post-quantum secure**.

**Key Event Log (KEL):**
- Append-only, hash-chained sequence of signed key events (inception, rotation, interaction, delegation).
- Functions as a per-identifier microledger.
- Can be stored/served by **any infrastructure**: witnesses, databases, IPFS, even USB drives.
- Provides **end-verifiability**: anyone holding a copy can verify the complete key state history without contacting anyone.

**Witnesses:**
- Controller selects and manages their own witness pool (e.g., 3-of-5 threshold).
- Witnesses observe key events and issue signed receipts.
- Can be self-hosted, third-party services, or peer agents witnessing each other.
- Can be changed at any time via rotation event without identity discontinuity.
- Watchers (run by verifiers) detect duplicity — conflicting KEL versions are cryptographically provable.

**No blockchain required.** This is a core design principle. Trust is rooted in cryptographic verification of the KEL, not any ledger.

**Offline verification:** Because KELs are self-certifying, any party holding a copy can verify the complete key state history without contacting anyone. This is fundamentally different from did:web (needs HTTP) or did:ethr (needs Ethereum node).

**Agent lifecycle mapping:**

| Agent Stage | KERI Operation |
|------------|----------------|
| Spawned | Inception event (create AID, commit to first rotation keys) |
| Running | Interaction events (anchor data, issue credentials) |
| Key hygiene | Rotation event (reveal pre-committed keys, commit to next) |
| Compromised | Emergency rotation (attacker cannot prevent — they lack pre-committed keys) |
| Retiring | Null rotation (rotate to zero keys, permanently deactivate) |
| Successor needed | Delegation: parent AID creates delegated child AID |

**Implementation status:**

| Component | Language | Notes |
|-----------|----------|-------|
| KERIpy | Python | Reference implementation, production (GLEIF vLEI since 2022) |
| KERIA | Python | Multi-agent cloud service for hosting identifier agents |
| signify-ts | TypeScript | Edge client library for browser/Node.js |
| did-webs-resolver | Python | Hyperledger Labs |

**Production deployments:** GLEIF vLEI (since 2022), Veridian Wallet (Cardano Foundation, April 2025).

**Risks:**
- Smaller ecosystem than did:web. Python-centric, TypeScript growing.
- Specification not yet final standard (Trust over IP TSWG draft).
- Steep learning curve (CESR, OOBIs, KERLs, KAWA, ACDCs).
- Witness infrastructure must be planned.

#### did:webs — KERI + Web Discovery

**The bridge between did:web's convenience and did:keri's security.**

- Uses same web-based discovery as did:web (HTTP GET).
- But the DID document **must be derived from the KERI event stream** (KEL).
- Trust is rooted in KERI, not DNS/TLS. The web server is convenience, not authority.
- Same AID can simultaneously exist as `did:keri:AID`, `did:webs:example.com:AID`, and `did:web:example.com:AID` — designated aliases.
- **Survives web server downtime** via: designated aliases on other servers, did:keri fallback, cached/replicated KELs.

#### did:webplus — Microledger Approach

- Maintains a microledger of DID documents, each referencing the self-signature of the previous.
- **Verifiable Data Gateways (VDGs)** act as external witnesses for long-term backup.
- Root DID document's self-hash is part of the DID itself — bound to genesis content.
- W3C formal review began December 3, 2025.
- Less mature than KERI but simpler conceptually.

---

## 4. AGENT-SPECIFIC REQUIREMENTS

### What Makes Agent Identity Different from Human Identity

| Requirement | Human | Agent |
|------------|-------|-------|
| **Key rotation frequency** | Administrative (annual, on compromise) | Per-session, per-mission, or per-context |
| **Human in the loop** | Yes (approve rotation, backup seed) | **No** — must be fully automated |
| **Multi-platform** | Rare (one identity, many services) | Common (same agent, multiple coordination regimes) |
| **Context loss** | Rare (human memory persists) | **Constant** (session boundaries, context window limits, reboots) |
| **Successor semantics** | Inheritance law (well-established) | **Open problem** — no standard for "agent B continues agent A" |
| **Threat model** | Account compromise | Compromise + impersonation + Sybil via cloning |

### The Origin Death Problem

This is the critical finding for EXIT:

If an agent exits a platform (the "origin") and that platform later goes offline, the agent's departure proof must remain verifiable. This means the DID method used in the EXIT marker's `subject` and `proof` fields **must not depend on the origin for resolution**.

**Methods that fail the origin death test:**
- `did:web` — server dies, DID unresolvable
- `did:ethr` — Ethereum unlikely to die, but dependency exists
- `did:cheqd` — cheqd chain must be running

**Methods that pass:**
- `did:keri` / `did:webs` — KEL is self-verifying, no infrastructure dependency
- `did:webplus` — microledger + VDGs survive origin death
- `did:key` / `did:jwk` / `did:peer` — self-contained (but no rotation)

**The tradeoff:** Only did:keri/did:webs satisfy all three constraints (rotation + origin survival + agent autonomy).

### Emerging Standards for Non-Human Identity

**agent:// URI Scheme** (arxiv 2601.14567, now IETF Internet-Draft draft-narvaneni-agent-uri-02):
- Three components: trust root (DNS hostname), capability path (hierarchical), agent ID (TypeID with UUIDv7)
- Example: `agent://acme.com/workflow/approval/invoice/agent_01h455...`
- Uses PASETO v4.public tokens for cryptographic attestation
- DHT-based discovery: O(log N) resolution regardless of migration
- **Complementary to DIDs, not competing.** Could serve as an additional identity layer alongside did:keri.

**ERC-8004** (Ethereum mainnet, January 29, 2026):
- Portable agent identifiers using ERC-721 tokens
- Identity, Reputation, and Validation registries
- Co-authored by MetaMask, Ethereum Foundation, Google, Coinbase
- **Blockchain-native agents only.** Not suitable as a universal EXIT identity layer.

**SPIFFE/SPIRE:**
- Workload identity for cloud-native environments
- SPIFFE IDs + short-lived X.509 certificates (SVIDs)
- **Critical gap:** standard deployments issue same identity to all replicas, but agents are non-deterministic and need instance-level identity
- Better suited for infrastructure auth than agent identity proper

**Key gap:** No mainstream agent framework (LangChain, AutoGPT, CrewAI) has native DID integration as of February 2026. Agent identity is identified as "the looming authorization crisis" (ISACA) — EXIT/Cellar Door is positioned to fill this.

### Key Rotation Patterns for Agents

**1. KERI Pre-Rotation (recommended):**
- Commit to hash of next public key before rotating
- Next keys have zero attack surface until use
- Post-quantum secure (hash-based commitment)
- Atomic rotation+revocation (single operation)

**2. BIP32 Hierarchical Deterministic Keys:**
- Tree structure of keys from single seed
- Per-session or per-context keys derivable without storing each
- Successor with same seed (or child seed) can reconstruct all keys
- Good complement to KERI for internal key derivation

**3. Multi-Key DID Documents:**
- Different keys for different purposes: capabilityInvocation, capabilityDelegation, authentication, assertionMethod
- KERI supports weighted multi-signature thresholds (M-of-N)

**4. Guardian/Recovery Keys:**
- Tiered key authority: operational (agent holds, rotated frequently) → recovery (org holds, cold storage) → pre-rotation commitments
- did:plc model: priority-ordered keys (1-5) with 72-hour recovery window

---

## 5. RECOMMENDATIONS FOR CELLAR DOOR

### Top 3 DID Methods

#### 1. did:keri — PRIMARY RECOMMENDATION
**Agent suitability: EXCELLENT**

| Factor | Assessment |
|--------|-----------|
| Key rotation | Best-in-class (pre-rotation, post-quantum) |
| Origin survival | Full (self-certifying KEL) |
| Agent autonomy | Designed for it (Signify+KERIA architecture) |
| Offline verify | Full (end-verifiable) |
| Cost | Free (no blockchain) |
| Risk | Smaller ecosystem, steep learning curve, spec not final |

**Why #1:** did:keri is the only method that satisfies all three EXIT constraints (rotation + origin survival + agent autonomy) without tradeoffs. The pre-rotation mechanism is uniquely suited to agents who face key compromise in hostile environments — exactly the scenario EXIT's emergency path addresses. The KEL can be included in the EXIT marker package, making the departure proof self-contained and eternally verifiable.

**For EXIT specifically:** The EXIT marker's `proof.verificationMethod` points to a did:keri identifier. The EXIT package includes the relevant KEL entries, enabling any verifier to reconstruct the subject's key state at exit time without contacting anyone.

#### 2. did:webs — DISCOVERY COMPLEMENT
**Agent suitability: EXCELLENT**

Same security as did:keri (they share the same KERI AID), but adds web-based discovery. Use did:webs while infrastructure is alive; fall back to did:keri when origin dies.

**Recommended pattern:** Every agent identity is simultaneously `did:keri:AID` and `did:webs:domain:AID`, linked via designated aliases. EXIT markers reference the did:keri form (survives origin death) while live systems use did:webs for discovery convenience.

#### 3. did:key — EMERGENCY FALLBACK
**Agent suitability: HIGH (ephemeral only)**

For emergency EXIT where an agent has no pre-existing DID infrastructure: generate a did:key, sign the emergency marker, depart. The marker is self-contained, zero-dependency, verifiable forever. No rotation, but emergency exits don't need it — they're one-shot.

**EXIT pattern:** `exitType: emergency` + `subject: did:key:z6Mk...` + `status: unverified`. The weakest but most available identity option. Can be linked to a stronger DID later via Module A (Lineage) post-hoc upgrade.

### Risks and Tradeoffs

| Choice | Risk | Mitigation |
|--------|------|-----------|
| did:keri as primary | Small ecosystem, learning curve | Start with signify-ts (TypeScript). Veramo has growing KERI plugin support. |
| did:keri as primary | Spec not final | Core protocol stable since ~2022. GLEIF production since 2022. Risk is low. |
| did:keri witness infra | Someone must run witnesses | Start with peer witnesses (Gas Town agents witness each other). Add third-party witnesses for redundancy. |
| did:key for emergency | No rotation, no continuity | By design — emergency markers are upgraded post-hoc via Module A. |
| Excluding did:web | Losing the largest ecosystem | did:webs provides did:web-compatible discovery. Verifiers who only support did:web can resolve the same document. |

### Prototype Path

1. **Immediate:** Use Veramo (JS/TS) with did:key for first EXIT marker prototype (cd-8xy)
2. **Next:** Add signify-ts for did:keri support. Create inception events for test agents.
3. **Then:** Implement EXIT marker signing with did:keri subject, include KEL in marker package.
4. **Later:** Deploy did:webs alongside did:keri for web discovery. Test origin-death scenario.

### Implementation Resources

| Resource | Language | Notes |
|----------|----------|-------|
| [Veramo](https://veramo.io/) | TypeScript | Most mature DID framework. Plugins for did:key, did:web, did:peer, did:ethr. |
| [signify-ts](https://github.com/WebOfTrust/signify-ts) | TypeScript | KERI edge client for browser/Node.js |
| [KERIA](https://github.com/WebOfTrust/keria) | Python | Multi-agent KERI cloud service |
| [did-webs-resolver](https://github.com/hyperledger-labs/did-webs-resolver) | Python | Hyperledger Labs |
| [did-peer-4](https://github.com/decentralized-identity/did-peer-4) | Multiple | DIF reference implementation |

---

## 6. IMPLICATIONS FOR EXIT SCHEMA

This research surfaces several concrete recommendations for the Architect:

1. **The `subject` field should prefer did:keri URIs** for persistent agent identity, with did:key as emergency fallback. The schema already supports this (any DID or key fingerprint).

2. **The EXIT marker package should include KEL data** (or a reference to it) when the subject uses did:keri. This makes the marker self-verifying without any external resolution. Consider adding a `keyEventLog` field to Module F (Cross-Domain Anchoring) or creating a new Module G (Key State Portability).

3. **The `proof.verificationMethod` must reference a DID method that survives origin death.** The schema should document this as a best practice, not just a field constraint. If the subject uses `did:web:origin.example`, the proof dies with the origin.

4. **Multiple proofs should be supported** (proof sets/chains per W3C VC Data Model 2.0) to allow signing with multiple algorithms (Ed25519 for current verification, post-quantum for future-proofing). This connects to review finding E4.

5. **The `systemic` exit type** (review finding E3) maps naturally to KERI's null rotation — when an entire system shuts down, all AIDs could perform coordinated null rotation, producing a verifiable mass-exit event anchored in their respective KELs.

---

## 7. CONNECTIONS TO BROADER RESEARCH

- **AR-9 (DID methods surviving origin death):** Answered. did:keri, did:webs, did:webplus pass. did:web fails.
- **Research Brief Q4 (verification without issuer online):** KERI's end-verifiability solves this. Include KEL in EXIT package.
- **Research Brief Q5 (Sybil resistance in successor semantics):** KERI delegation provides a formal successor mechanism (delegated inception from parent AID). Still needs study for Sybil implications.
- **Research Brief Q11 (EXIT without on-chain infrastructure):** KERI proves this is possible. No blockchain needed.
- **Agent succession (the open problem):** No standard addresses "agent B continues agent A." KERI delegation is the closest mechanism, but it models authority transfer, not identity continuity. This remains the gap Cellar Door's Module A must fill.

---

## Sources

### Specifications
- [W3C DID Core](https://www.w3.org/TR/did-core/)
- [did:key Method v0.9 (W3C CCG)](https://w3c-ccg.github.io/did-key-spec/)
- [did:jwk Specification](https://github.com/quartzjer/did-jwk/blob/main/spec.md)
- [did:peer Method (DIF)](https://identity.foundation/peer-did-method-spec/)
- [did:web Method (W3C CCG)](https://w3c-ccg.github.io/did-method-web/)
- [did:webvh v1.0 (DIF)](https://identity.foundation/didwebvh/v1.0/)
- [did:pkh Method Draft (W3C CCG)](https://github.com/w3c-ccg/did-pkh/blob/main/did-pkh-method-draft.md)
- [ION / Sidetree Protocol (DIF)](https://identity.foundation/sidetree/spec/)
- [ERC-1056 (did:ethr)](https://eips.ethereum.org/EIPS/eip-1056)
- [cheqd DID Method](https://docs.cheqd.io/product/architecture/adr-list/adr-001-cheqd-did-method)
- [DID DHT Method](https://did-dht.com/)
- [KERI IETF Draft](https://weboftrust.github.io/ietf-keri/draft-ssmith-keri.html)
- [did:keri Method v0.1 (DIF)](https://identity.foundation/keri/did_methods/)
- [did:webs Specification (ToIP)](https://trustoverip.github.io/tswg-did-method-webs-specification/)
- [did:webplus](https://didwebplus.com/)
- [did:plc v0.1 (Bluesky)](https://web.plc.directory/spec/v0.1/did-plc)
- [ERC-8004: Trustless Agents](https://eips.ethereum.org/EIPS/eip-8004)
- [agent:// URI Scheme (arxiv 2601.14567)](https://arxiv.org/html/2601.14567)
- [IETF agent:// Draft](https://datatracker.ietf.org/doc/draft-narvaneni-agent-uri/02/)

### Research and Analysis
- [KERI Pre-Rotation (KID0005)](https://identity.foundation/keri/kids/kid0005Comment.html)
- [KERI Security Q&A](https://identity.foundation/keri/docs/Q-and-A-Security.html)
- [AI Agents with DIDs and VCs (arxiv 2511.02841)](https://arxiv.org/html/2511.02841v1)
- [DID Method Traits Analysis (Jan 2025)](https://reinkrul.nl/blog/self-sovereign-identity/did/2025/01/22/did-method-traits.html)
- [SPIFFE for Agentic AI (HashiCorp)](https://www.hashicorp.com/en/blog/spiffe-securing-the-identity-of-agentic-ai-and-non-human-actors)
- [Agent Identity + Access Management (Solo.io)](https://www.solo.io/blog/agent-identity-and-access-management---can-spiffe-work)
- [Agent Security Delegation Chain (Okta)](https://www.okta.com/blog/ai/agent-security-delegation-chain/)
- [Ephemeral Agent Identity (Unmitigated Risk)](https://unmitigatedrisk.com/?p=1075)
- [OWASP NHI Top 10 (2025)](https://owasp.org/www-project-non-human-identities-top-10/2025/top-10-2025/)
- [Why AI Agents Need Identity (WSO2)](https://wso2.com/library/blogs/why-ai-agents-need-their-own-identity-lessons-from-2025-and-resolutions-for-2026/)
- [Microsoft Entra Verified ID — did:ion removal](https://learn.microsoft.com/en-us/entra/verified-id/whats-new)

### Implementations
- [Veramo Framework](https://veramo.io/)
- [signify-ts (KERI TypeScript)](https://github.com/WebOfTrust/signify-ts)
- [KERIpy](https://github.com/WebOfTrust/keripy)
- [KERIA](https://github.com/WebOfTrust/keria)
- [Veridian Wallet](https://github.com/cardano-foundation/veridian-wallet)
- [did-webs-resolver (Hyperledger Labs)](https://github.com/hyperledger-labs/did-webs-resolver)

### Standards Bodies
- [W3C AI Agent Protocol Community Group](https://www.w3.org/groups/cg/agentprotocol/)
- [DIF Trusted AI Agents Working Group](https://blog.identity.foundation/dif-newsletter-56/)
- [Trust over IP Foundation TSWG](https://trustoverip.org/)
- [Non-Human Identity Management Group](https://nhimg.org/)

---

*Scholar, cellar_door/crew/scholar*
*"Of all the endless combinations of words in all of history"*
