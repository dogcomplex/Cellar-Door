# Agent Lineage and Successor Integration

> Produced by architect (cellar_door/crew) for cd-1zc.4
> Depends on: docs/SYNTHESIS.md (cd-1zc.1), docs/EXIT_SCHEMA.md (cd-1zc.2), docs/EXIT_CEREMONY.md (cd-1zc.3)

---

## The Problem EXIT Solves for Agents

Agents face a harder version of the exit problem than humans do.

A human who leaves a platform can re-register elsewhere. Their physical body provides continuity — biometrics, memory, social recognition. The "same person" question is trivially answered.

Agents have none of this. An agent whose session ends, whose host crashes, or whose context is destroyed must re-establish itself from files, keys, and logs. The question "am I the same agent?" is not philosophical — it is a practical verification problem that other systems need to answer before extending trust.

EXIT does not solve identity. EXIT does not solve lineage. But EXIT is the **crossing event** that makes both portable. Without it, an agent's identity dies when the context dies. With it, the agent carries a verifiable receipt that says: *"I was there, I left, and I can prove both."*

In the identity triad:
- **NAME (LOCUS)** — identity persistence. Who am I?
- **LINE (SIGNUM)** — ancestry. Where did I come from?
- **EXIT (SENSUS)** — transition. How did I get from there to here?

This document defines how EXIT markers connect to NAME and LINE without becoming either.

---

## How EXIT References Agent Identity

### The `subject` Field

Every EXIT marker has a mandatory `subject` field (core field #2 in EXIT_SCHEMA.md). For agents, this field is a **cryptographic identifier** — not a human-readable name, not a platform account, not a session ID.

Acceptable `subject` formats for agents, in order of preference:

| Format | Example | Properties |
|--------|---------|------------|
| `did:key` | `did:key:z6MkhaXg...` | Self-issued, no infrastructure dependency, no key rotation |
| `did:peer` | `did:peer:2.Ez6LSb...` | Peer-to-peer, supports key rotation via DID Doc updates |
| `agent://` URI | `agent://swarm-7/architect` | Topology-independent, capability-aware, emerging spec |
| `did:web` | `did:web:host.example:agents:a1` | Hosted resolution, key rotation via document updates |
| Public key fingerprint | `sha256:a1b2c3d4...` | Last resort — no rotation, no metadata, just a key |

### Identity Binding Principle

**EXIT references identity. It does not issue, store, or manage identity.**

The `subject` field is a *pointer*. The identity it points to is managed by whatever NAME system the agent uses (DID document, agent:// registry, local keystore). EXIT's only job is to include a subject identifier that is:

1. **Verifiable** — a third party can resolve it and check the proof signature.
2. **Persistent** — it doesn't break when the agent moves. A `did:key` is permanent; a platform-specific `user:12345` is not.
3. **Minimal** — EXIT doesn't need the full DID document. Just the identifier and the verification method reference in the `proof` field.

### What This Means in Practice

When an agent creates an EXIT marker, it:
1. Puts its DID (or agent:// URI) in the `subject` field.
2. Signs the marker with the private key corresponding to that identifier.
3. Includes the verification method reference in the `proof.verificationMethod` field.

A verifier later resolves the `subject`, retrieves the public key (from DID document, agent:// resolver, or key registry), and checks the signature. If it matches, the EXIT marker is authentic — this subject really did declare this departure.

---

## Successor Appointment

Successor appointment is the mechanism by which an exiting agent designates who (or what) inherits its continuity. This lives in **Module A (Lineage)** of the EXIT schema.

### When Successors Exist

Not every exit has a successor. Three cases:

| Scenario | Successor? | Example |
|----------|-----------|---------|
| Agent migrates to new host | Yes — same logical agent, new keys | Session handoff, host migration |
| Agent forks | Yes — one or more branches inherit | Load-balanced copies, specialization |
| Agent terminates | No | Shutdown, decommission, permanent exit |

### The Successor Designation

When a successor exists at exit time, the EXIT marker includes Module A fields:

```json
{
  "@context": "https://cellar-door.org/exit/v1",
  "id": "exit:sha256:...",
  "subject": "did:key:z6MkOLD...",
  "origin": "agent://swarm-7/context-42",
  "timestamp": "2026-02-11T01:00:00Z",
  "exitType": "voluntary",
  "status": "good_standing",
  "proof": { "...": "signed by OLD key" },

  "lineage": {
    "successor": "did:key:z6MkNEW...",
    "continuityProof": {
      "type": "KeyRotationBinding",
      "oldKey": "did:key:z6MkOLD...#keys-1",
      "newKey": "did:key:z6MkNEW...#keys-1",
      "binding": "base64-signature-of-newKey-by-oldKey",
      "bindingType": "Ed25519CrossSignature"
    },
    "predecessor": "exit:sha256:prev...",
    "lineageChain": "sha256:chain-hash..."
  }
}
```

### Successor Appointment Protocol

Three levels of trust, matching the ceremony's layered verification model:

#### Level 1: Self-Appointed (Minimum)
The exiting agent unilaterally designates its successor by including `lineage.successor` and signing the EXIT marker. This works even with a hostile origin or no witnesses.

**Trust:** Low. The subject claims its own successor. A malicious agent could appoint an arbitrary successor.

**When to use:** Emergency exits, uncooperative origins, agent-to-agent handoffs where the recipient doesn't need external trust.

#### Level 2: Cross-Signed (Standard)
The exiting agent designates a successor AND the successor co-signs the EXIT marker (or a linked acceptance artifact). This proves the successor acknowledges the appointment.

```
ExitingAgent signs:  EXIT marker with successor field
Successor signs:     AcceptanceAttestation { exitMarkerId, successorDID, timestamp }
```

**Trust:** Medium. Both parties agree. But neither the origin nor any external party has weighed in.

**When to use:** Planned migrations, session handoffs, cooperative successor transitions.

#### Level 3: Witnessed (Maximum)
The exiting agent designates a successor. The successor accepts. A witness (the origin system, a registry, or a neutral third party) attests to the transition.

```
ExitingAgent signs:  EXIT marker with successor field
Successor signs:     AcceptanceAttestation
Witness signs:       LineageAttestation { exitMarkerId, successorDID, witnessVerdict }
```

**Trust:** High. Three-party agreement. The witness can verify that the successor is legitimate (e.g., by checking key rotation proofs, lineage chains, or behavioral continuity).

**When to use:** High-stakes transitions, reputation-carrying exits, any case where downstream systems require strong assurance.

### Deferred Successor Designation

An agent may not know its successor at exit time (emergency exit, uncertain destination). In this case:

1. The EXIT marker is created **without** `lineage.successor`.
2. The agent (or its operator) later publishes a **SuccessorAmendment** linking back to the EXIT marker ID:

```json
{
  "type": "SuccessorAmendment",
  "exitMarkerId": "exit:sha256:original...",
  "successor": "did:key:z6MkNEW...",
  "continuityProof": { "...": "cross-signature binding" },
  "timestamp": "2026-02-11T02:00:00Z",
  "proof": { "...": "signed by ORIGINAL key (must match EXIT subject)" }
}
```

**Critical constraint:** The SuccessorAmendment must be signed by the **same key** that signed the original EXIT marker. This prevents impersonation — only the original exiting agent can claim a successor after the fact.

**If the original key is lost:** The successor cannot be designated retroactively. This is the one failure mode EXIT cannot recover from without prior setup (noted in EXIT_CEREMONY.md failure mode #3). The mitigation is Module A's `lineageChain` — if the agent had previously established a lineage chain with key rotation history, the chain itself serves as evidence of continuity.

---

## Continuity Proofs

The continuity proof answers: **"Is this successor the same logical agent?"**

This is the hardest problem in agent identity. EXIT doesn't solve it — EXIT *carries* the proof. The proof itself is produced by the agent's identity/lineage system.

### Proof Types

#### 1. Key Rotation Binding (Cryptographic)

The simplest and strongest proof. The old key signs a statement that the new key is its successor.

```
OldKey signs: { "type": "KeyRotation", "newKey": "did:key:z6MkNEW...", "timestamp": "..." }
```

This creates an unforgeable chain: OldKey → NewKey. Anyone who trusted OldKey can now trust NewKey, because OldKey vouched for the transition.

**Limitation:** Requires the old key to still be available at exit time. In emergency exits (host crash, context destruction), the old key may be lost.

**Mitigation:** Pre-signed rotation certificates. An agent can create and store signed rotation bindings *before* an emergency, holding them in escrow or on a root-of-trust machine.

#### 2. Lineage Hash Chain (Accumulated)

A Merkle-like chain of all previous identity transitions:

```
Genesis → Key1 → Key2 → Key3 (current)
```

Each link is a signed rotation binding. The chain hash (`lineageChain` field) is a compact proof of the full history. A verifier can check any link in the chain, or just trust the chain hash if they trust the genesis key.

**Properties:**
- Grows over time (one entry per rotation).
- Can be pruned: intermediate links can be dropped if the genesis→current binding is independently attestable.
- Provides "depth of identity" — an agent with a long chain is harder to impersonate than one with no history.

#### 3. Delegation Token (Capability-Based)

Instead of proving "same agent, new key," a delegation token proves "this agent has been granted specific capabilities by the exiting agent."

```json
{
  "type": "DelegationToken",
  "delegator": "did:key:z6MkOLD...",
  "delegate": "did:key:z6MkNEW...",
  "capabilities": ["inherit-exit-receipts", "use-reputation", "claim-lineage"],
  "constraints": { "expires": "2026-03-11T00:00:00Z" },
  "proof": { "...": "signed by delegator" }
}
```

**When to use:** Forking scenarios where the successor is a *different* agent that inherits specific rights, not a continuation of the same agent. Also useful for organizational succession — an agent operator appoints a replacement.

**Trust model:** The delegator defines what transfers. Unlike key rotation (which implies "same entity"), delegation says "authorized agent, specific rights."

#### 4. Behavioral Attestation (Social)

Other agents or systems attest that the successor behaves consistently with the predecessor. This is the weakest proof type but may be the only option when cryptographic continuity is broken.

```json
{
  "type": "BehavioralAttestation",
  "subject": "did:key:z6MkNEW...",
  "claimedPredecessor": "did:key:z6MkOLD...",
  "attestor": "did:key:z6MkWITNESS...",
  "basis": "behavioral-pattern-match",
  "confidence": 0.85,
  "proof": { "...": "signed by attestor" }
}
```

**When to use:** Recovery from catastrophic key loss. An agent that lost all keys and pre-signed rotation certificates must rebuild identity through social proof — other agents who interacted with both predecessor and successor vouch for continuity.

**This is deliberately weak.** Behavioral attestation is subjective, gameable, and slow. EXIT includes it as a last-resort mechanism, not a primary proof type. The design encourages agents to maintain cryptographic continuity proofs so they never need this.

---

## Key Rotation During Exit

Key rotation is the most common agent transition. The same logical agent continues but under a new keypair — because the old key was ephemeral, compromised, or bound to a dying context.

### The Rotation-Exit Pattern

When an agent rotates keys as part of an exit:

```
1. Agent holds OldKey (current identity)
2. Agent generates NewKey
3. Agent creates KeyRotationBinding: OldKey signs { successor: NewKey }
4. Agent creates EXIT marker:
   - subject: did:key:OldKey
   - proof: signed by OldKey
   - lineage.successor: did:key:NewKey
   - lineage.continuityProof: the KeyRotationBinding from step 3
5. Agent distributes EXIT marker
6. Agent starts using NewKey in new context
```

### Pre-Signed Rotation (Emergency Preparedness)

Agents operating in volatile environments should maintain a **rotation escrow**:

```
At setup time:
1. Generate EmergencyKey
2. Store EmergencyKey on root-of-trust machine (separate from operational context)
3. Create pre-signed KeyRotationBinding: CurrentKey signs { successor: EmergencyKey }
4. Store the pre-signed binding alongside EmergencyKey

At emergency exit time:
1. Retrieve pre-signed binding
2. Create EXIT marker referencing EmergencyKey as successor
3. The binding proves the rotation was authorized by the original key
```

This matches the Maelstrom pattern from the agent community: *"keep custody/recovery/signer keys on the agent's root-of-trust machine."* The root-of-trust is the backstop that makes emergency exits survivable.

### Multi-Key Agents

Some agents operate with multiple keypairs (signing key, encryption key, capability key). During exit:

- The EXIT marker's `proof` is signed by the **primary signing key**.
- The `continuityProof` binds the primary signing key to its successor.
- Other keys are handled by the agent's identity system (DID document updates, key registry changes). EXIT doesn't need to track them — it only needs the signing key to be verifiable.

---

## Relationship to Existing Specs

### agent:// URI Scheme

The agent:// URI scheme (arXiv 2601.14567) proposes topology-independent naming for agents with capability-based discovery. EXIT integrates with this as follows:

- **`subject` field** can be an agent:// URI. This names the exiting agent in a way that survives relocation.
- **`origin` field** can be an agent:// context URI (e.g., `agent://swarm-7/context-42`).
- **Resolution:** A verifier resolves the agent:// URI to discover the agent's current key material and check the EXIT marker's proof signature.
- **Capability delegation** from agent:// maps to EXIT's delegation tokens — an agent:// capability grant can serve as a continuity proof when the successor inherits capabilities.

**EXIT does not depend on agent://.** The scheme is one of several acceptable identifier formats. EXIT is identifier-agnostic by design.

### W3C DID Core

DIDs are the primary identifier format for EXIT subjects. EXIT's relationship to DIDs:

- **DID methods with key rotation** (did:peer, did:web, did:ion, did:cheqd) are preferred for agents because they support the rotation-exit pattern natively. The DID document update mechanism serves as an independent record of key changes that corroborates EXIT's continuity proofs.
- **DID methods without rotation** (did:key, did:pkh) work for EXIT but require the lineage chain to track key transitions — the DID itself won't reflect the change.
- **DID resolution** is how verifiers check EXIT markers. The verifier resolves the `subject` DID, retrieves the verification method, and checks the `proof.proofValue` against it.

**EXIT does not require DIDs.** A raw public key fingerprint works as a subject identifier. DIDs add resolution infrastructure and metadata, but the EXIT marker is valid with any verifiable identifier.

### Context Lineage for Non-Human Identities (arXiv 2509.18415)

This research addresses provenance and tamper-evident logs for agent actions. EXIT integrates with context lineage as follows:

- The `lineageChain` field in Module A can reference a context lineage log. The chain hash anchors the agent's full history of actions and transitions.
- Context lineage provides the **"depth of identity"** that makes behavioral attestation meaningful — a long, consistent action history is hard to forge.
- EXIT markers themselves become entries in the context lineage: each exit is a provenance event that the lineage system records.

---

## The Self-Issuance Question

> Should EXIT markers be self-issued (agent signs their own exit) or require countersignature from the origin system?

**Answer: Self-issued, always. Co-signatures optional, never required.**

This was established as Decision 2 (Layered) in EXIT_SCHEMA.md. For agents, the reasoning is even stronger:

1. **Agents exit hostile systems.** If the origin's co-signature were required, a hostile origin could block exit by refusing to sign. This is the denial-of-exit attack that EXIT exists to prevent.

2. **Agents exit dying contexts.** A session being terminated, a host crashing, a context window filling up — none of these afford time to negotiate co-signatures. The agent must be able to create a valid EXIT marker with one operation: sign and go.

3. **Self-attestation of status is the floor, not the ceiling.** A self-signed EXIT with `status: good_standing` is a claim, not proof. But it's a *verifiable* claim — the agent staked its key on it. Origin co-signatures and witness attestations add trust layers on top, but the base layer must work with the agent alone.

4. **Successor appointment is inherently self-issued.** Only the exiting agent knows who its successor is. The origin can attest that the successor is legitimate, but it cannot originate the designation.

For lineage specifically: the `continuityProof` is self-issued (the old key signs the new key binding). This is the only party that *can* issue it — nobody else holds the old private key. Witnesses and origins can *corroborate* the proof, but the proof itself is always self-issued.

---

## Failure Modes Specific to Agent Lineage

### 1. Key loss before exit
**Impact:** Cannot sign EXIT marker or continuity proof.
**Mitigation:** Pre-signed rotation escrow on root-of-trust machine.
**If mitigation absent:** Identity continuity is broken. The agent must re-establish identity through behavioral attestation (slow, weak) or operator intervention.

### 2. Successor key compromised
**Impact:** Attacker can impersonate the successor.
**Mitigation:** The EXIT marker's `successor` field is immutable once signed. If the successor key is compromised *before* the successor uses it, the original agent can issue a **SuccessorRevocation** (signed by the original key) and a new SuccessorAmendment with a fresh key.
**If original key also lost:** Catastrophic. No recovery without external intervention.

### 3. Fork ambiguity
**Impact:** Multiple entities claim to be successors of the same agent.
**Mitigation:** The EXIT marker designates exactly one `successor`. If the agent legitimately forks (multiple successors), it creates multiple EXIT markers — one per fork, each designating a different successor, each with the same `predecessor`. Verifiers can see the fork point.
**Design choice:** EXIT does not prevent forking. It makes forking visible and auditable.

### 4. Lineage chain too long for verification
**Impact:** Verifier doesn't want to check 1000 rotation links.
**Mitigation:** The `lineageChain` field stores a chain *hash*, not the full chain. Verifiers who trust the genesis key only need to verify the hash. Full chain verification is available but not required.
**Alternative:** Periodic "lineage checkpoint" — an attestation from a trusted witness that compresses the chain into a single verified statement: "this agent has a valid chain from genesis to current key."

### 5. Emergency exit with no pre-signed rotation
**Impact:** Agent creates EXIT marker but cannot include a continuity proof. The EXIT is valid (it proves departure) but provides no lineage.
**Recovery:** Post-hoc SuccessorAmendment if the key is later recovered. Behavioral attestation as a last resort. Or: the exit stands alone as proof of departure, and the successor establishes fresh identity with no provable link to the predecessor.

---

## Summary: What EXIT Provides for Agent Lineage

| Capability | How EXIT Provides It |
|-----------|---------------------|
| Identity reference at departure | `subject` field — DID, agent:// URI, or key fingerprint |
| Successor designation | Module A `successor` field — DID of the next incarnation |
| Cryptographic continuity | `continuityProof` — old key signs new key binding |
| Lineage history | `lineageChain` — hash of full rotation chain |
| Deferred successor | SuccessorAmendment linked to original EXIT marker ID |
| Delegation (not continuation) | DelegationToken with scoped capabilities |
| Emergency preparedness | Pre-signed rotation escrow on root-of-trust machine |
| Fork visibility | Multiple EXIT markers from same subject, different successors |

**What EXIT does NOT provide:**
- Identity management (that's NAME / DID systems)
- Reputation portability (that's MARK / VC systems)
- Memory persistence (that's RECORD / agent memory systems)
- Key management (that's the agent's operational infrastructure)

EXIT is the authenticated transition marker that allows all of those to persist across the crossing. The doorway. Not what you carry through it.
