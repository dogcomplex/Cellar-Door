# EXIT Schema: The Irreducible EXIT Data Structure

> Produced by architect (cellar_door/crew) for cd-1zc.2
> Depends on: docs/SYNTHESIS.md (cd-1zc.1)

---

## Design Principles

1. **Non-custodial.** The EXIT marker references external state but never contains it.
2. **Offline-verifiable.** A third party can verify the marker without the origin system being live.
3. **Agent-native.** Works for autonomous agents, not just human accounts.
4. **Minimal core, optional extensions.** The core is what every EXIT must have. Everything else is a module.
5. **Envelope-agnostic.** The core schema defines semantics; the envelope (VC, JWT, CBOR, protobuf) is a packaging decision made separately.

---

## Core Fields (Mandatory)

These are the absolute minimum for a valid EXIT marker. Remove any one and you no longer have a verifiable departure proof.

### 1. `id` — EXIT Marker Identifier
- **Type:** URI / content-addressed hash
- **What:** Globally unique identifier for this specific exit event.
- **Why mandatory:** Without an ID, the marker can't be referenced, deduplicated, or revoked. Other systems need to point at "this specific departure."
- **Example:** `exit:sha256:a1b2c3...` or `urn:exit:cd:a1b2c3`

### 2. `subject` — Who Is Exiting
- **Type:** DID, agent URI, or public key fingerprint
- **What:** The identity of the departing entity.
- **Why mandatory:** An exit with no subject is meaningless. This is the "I" in "I am no longer here." Does not need to be a human-readable name — a cryptographic identifier suffices.
- **Note:** This is a *reference* to identity, not identity itself. EXIT does not issue or store identity.

### 3. `origin` — What Is Being Exited
- **Type:** URI / domain identifier
- **What:** The coordination regime, platform, DAO, marketplace, or context being departed.
- **Why mandatory:** An exit with no origin is just a statement about the subject, not a transition. The origin scopes the exit — you leave *something specific*.
- **Example:** `did:web:marketplace.example`, `dao:moloch:0xabc...`, `agent://swarm-7/context-42`

### 4. `timestamp` — When The Exit Occurred
- **Type:** ISO 8601 datetime (UTC)
- **What:** The moment the exit was declared.
- **Why mandatory:** Exit is a temporal event. Without a timestamp, you can't establish ordering, detect replays, or reason about validity windows. In agent contexts, this anchors the exit to a specific point in the entity's timeline.

### 5. `exitType` — Nature of Departure
- **Type:** Enum: `voluntary` | `forced` | `emergency`
- **What:** Whether the entity chose to leave, was expelled, or departed under duress/system-failure.
- **Why mandatory:** The exit type changes how downstream systems interpret the marker. A voluntary exit in good standing carries different weight than a forced expulsion. Omitting this would make all exits ambiguous.
- **Definitions:**
  - `voluntary` — Subject initiated departure. No dispute, no coercion.
  - `forced` — Origin system expelled the subject (slashing, ban, contract termination).
  - `emergency` — Departure under abnormal conditions (system shutdown, context destruction, imminent loss). Signals that normal ceremony steps may have been truncated.

### 6. `proof` — Cryptographic Signature
- **Type:** Object containing signature algorithm, value, and verification key reference
- **What:** The cryptographic proof that authenticates this marker.
- **Why mandatory:** Without proof, the marker is an unsigned claim. The *entire point* of EXIT is that the departure is *verifiable*. This is what makes it more than a log entry.
- **Structure:**
  ```
  {
    "type": "Ed25519Signature2020" | "EcdsaSecp256k1Signature2019" | ...,
    "created": "<datetime>",
    "verificationMethod": "<DID or key URI>",
    "proofValue": "<base64-encoded signature>"
  }
  ```
- **Who signs:** At minimum, the *subject* signs (self-attestation of departure). Co-signatures from the origin or witnesses are optional extensions.

### 7. `status` — Standing At Departure
- **Type:** Enum: `good_standing` | `disputed` | `unverified`
- **What:** The subject's relationship status with the origin at the moment of exit.
- **Why mandatory:** This is the minimal reputation portability that EXIT enables. Without it, the marker says "I left" but not "how things stood when I left." This is the difference between a passport stamp and a warrant.
- **Definitions:**
  - `good_standing` — No open disputes, obligations met. The "clean exit certificate."
  - `disputed` — Active disputes exist at time of departure. The exit is valid but the relationship is contested.
  - `unverified` — Standing could not be determined (e.g., emergency exit, origin unresponsive).

---

## Core Schema (JSON)

```json
{
  "@context": "https://cellar-door.org/exit/v1",
  "id": "exit:sha256:...",
  "subject": "did:key:z6Mk...",
  "origin": "did:web:marketplace.example",
  "timestamp": "2026-02-10T23:00:00Z",
  "exitType": "voluntary",
  "status": "good_standing",
  "proof": {
    "type": "Ed25519Signature2020",
    "created": "2026-02-10T23:00:00Z",
    "verificationMethod": "did:key:z6Mk...#keys-1",
    "proofValue": "z3FXQqF..."
  }
}
```

**Size:** ~300-500 bytes. This is intentionally small — an EXIT marker should be cheap to create, store, and transmit.

---

## Optional Modules

Each module extends the core with additional fields. A valid EXIT marker does NOT require any of these. They exist for systems that need richer semantics.

### Module A: Lineage (Agent Continuity)
For agents that need to establish successor relationships.

| Field | Type | Purpose |
|-------|------|---------|
| `predecessor` | URI/DID | Previous incarnation of this entity |
| `successor` | URI/DID | Designated successor (if known at exit time) |
| `lineageChain` | Array of URIs | Full ancestry chain (compact form: hash of chain) |
| `continuityProof` | Object | Cryptographic proof binding predecessor to successor |

**When to include:** Agent migrations, context cycling, session handoffs, instance forking.

### Module B: State Snapshot Reference
For exits that need to anchor to a specific state.

| Field | Type | Purpose |
|-------|------|---------|
| `stateHash` | Hash | Content-addressed hash of state at exit time |
| `stateLocation` | URI | Where the full state snapshot can be retrieved |
| `stateSchema` | URI | Schema describing the state format |
| `obligations` | Array | Outstanding commitments at exit (hashes/references only) |

**When to include:** Financial exits, DAO departures, any context where post-exit disputes may reference pre-exit state. Note: EXIT stores the *hash*, not the state itself.

### Module C: Dispute Bundle
For exits where evidence preservation matters.

| Field | Type | Purpose |
|-------|------|---------|
| `disputes` | Array of objects | Active disputes at exit time (IDs + status only) |
| `evidenceHash` | Hash | Hash of evidence bundle (stored externally) |
| `challengeWindow` | Object | `{ opens, closes, arbiter }` if a challenge period applies |
| `counterpartyAcks` | Array | Co-signatures from counterparties acknowledging exit |

**When to include:** Contested departures, exits from systems with arbitration, any exit where "the story of what happened" matters later.

### Module D: Economic
For exits involving assets or financial obligations.

| Field | Type | Purpose |
|-------|------|---------|
| `assetManifest` | Array | Assets being ported (type + amount + destination, as references) |
| `settledObligations` | Array | Obligations resolved at exit |
| `pendingObligations` | Array | Obligations NOT resolved (with escrow references) |
| `exitFee` | Object | Any cost of exit (amount + recipient) |

**When to include:** Financial platforms, DAOs with treasuries, marketplaces with escrow.

### Module E: Metadata / Narrative
For human-readable context.

| Field | Type | Purpose |
|-------|------|---------|
| `reason` | String | Free-text reason for departure |
| `narrative` | String | Human-readable summary of exit circumstances |
| `tags` | Array of strings | Domain-specific labels |
| `locale` | String | Language of narrative fields |

**When to include:** When the exit receipt will be read by humans or presented in UIs.

### Module F: Cross-Domain Anchoring
For exits that need to be verifiable across chains or systems.

| Field | Type | Purpose |
|-------|------|---------|
| `anchors` | Array of objects | `{ chain, txHash, blockHeight }` — on-chain anchoring points |
| `registryEntries` | Array of URIs | Entries in external registries or certificate transparency logs |

**When to include:** High-assurance exits, regulatory contexts, cross-chain migrations.

---

## Field-by-Field Rationale: Why These 7 Core Fields

| Field | Included Because | Would Removal Break |
|-------|-----------------|-------------------|
| `id` | Deduplication, reference, revocation | Can't point at a specific exit |
| `subject` | "Who left" — the fundamental question | Exit becomes anonymous noise |
| `origin` | "Left what" — scopes the departure | Exit has no context |
| `timestamp` | Temporal ordering, replay detection | Can't reason about when or sequence |
| `exitType` | Interpretation depends on voluntary vs forced | All exits look the same — dangerous ambiguity |
| `proof` | Verifiability is the entire value proposition | It's just an unsigned log entry |
| `status` | Minimal reputation portability | "I left" says nothing about how things stood |

### Why NOT these (considered and excluded from core):

| Candidate | Why Excluded |
|-----------|-------------|
| `destination` | EXIT marks departure, not arrival. Where you go is not EXIT's concern. Including it would couple EXIT to migration. |
| `stateSnapshot` | Violates non-custodial principle. State lives elsewhere; EXIT can reference it via Module B. |
| `successor` | Not all exits have successors. Critical for agents but belongs in Module A. |
| `reason` | Subjective, optional, and not cryptographically meaningful. Module E. |
| `challengeWindow` | Only relevant for dispute-aware exits. Module C. |
| `assets` | EXIT is not a financial instrument. Module D. |
| `counterpartySignatures` | Requires origin system liveness. Co-signatures strengthen trust but can't be mandatory or EXIT only works with cooperative origins. |

---

## Decisions Flagged for Mayor

### Decision 1: Envelope Format
**Options:**
- **(A) W3C Verifiable Credential envelope.** EXIT markers are VCs. Immediate interop with VC ecosystem (wallets, verifiers, registries). Constrains field naming and structure to VC Data Model 2.0. Risk: coupling to a specific standard early.
- **(B) Custom JSON-LD context.** Define `https://cellar-door.org/exit/v1` as a standalone context. Can be wrapped in a VC later but isn't one by default. More flexibility, less immediate interop.
- **(C) Dual format.** Define the canonical schema as JSON-LD; provide a VC "wrapper" profile that maps it into VC envelope. Best of both, but more spec surface.

**Architect recommendation:** **(C) Dual format.** The core schema should be self-contained and not depend on the VC ecosystem. But a VC profile should exist from day one so the two worlds can interoperate. This matches the "envelope-agnostic" principle — EXIT defines semantics, not packaging.

### Decision 2: Verification Model
**Options:**
- **(A) Subject-signed only.** The exiting entity is the sole signer. Simplest. Works even when the origin is hostile or offline. But: self-attestation of "good standing" is weak.
- **(B) Co-signed (subject + origin).** Both parties sign. Stronger trust. But: requires origin cooperation, which may not exist (the whole point of exit is that you might be leaving a hostile system).
- **(C) Layered.** Subject signature is mandatory (core). Origin co-signature and witness attestations are optional (modules). Verifiers decide what trust level they require.

**Architect recommendation:** **(C) Layered.** The core proof is always subject-signed. This ensures EXIT works in the worst case (hostile origin, emergency departure). Co-signatures from origin or witnesses add trust but are never required for a valid marker. This matches the non-custodial, exit-always-available principle.

### Decision 3: Status Field Semantics
**Options:**
- **(A) Self-attested status.** The subject declares their own standing. Simple but easily gamed ("I declare I left in good standing" when you were actually expelled).
- **(B) Origin-attested status.** The origin system sets the status. Accurate but gives the origin veto power over the narrative ("we say you were disputed" as retaliation).
- **(C) Multi-source status.** The core `status` field is subject-attested. An optional `originStatus` field (Module C) carries the origin's view. Verifiers see both perspectives.

**Architect recommendation:** **(C) Multi-source.** Self-attestation in core (because EXIT must work without origin cooperation), with optional origin perspective in the dispute module. This prevents both gaming and retaliation. Downstream systems can set their own policies for which attestations they trust.

---

## Schema Versioning

The `@context` URI includes a version path (`/v1`). The core fields are the v1 contract. Modules can be added without breaking v1 compatibility. A future v2 would only be needed if core fields change — which should be extremely rare given how minimal the core is.

---

## Size and Performance Characteristics

| Configuration | Approximate Size | Use Case |
|--------------|-----------------|----------|
| Core only | 300-500 bytes | Agent session exit, quick departure |
| Core + Lineage | 500-800 bytes | Agent migration with successor |
| Core + State + Dispute | 800-1500 bytes | DAO exit with evidence |
| Core + All modules | 2-4 KB | Full ceremony with economic settlement |

EXIT markers are small by design. They are proofs, not payloads.
