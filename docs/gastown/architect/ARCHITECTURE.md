# EXIT Primitive: Architecture Specification v1

> Cellar Door — Phase 1 Architectural Specification
> Produced by architect (cellar_door/crew) for epic cd-1zc
> Date: 2026-02-11

---

## 1. What This Document Is

This is the living architectural specification for the EXIT primitive. It synthesizes four detailed design documents into a single reference that crew and polecats can build against.

**Detailed specs (read these for depth):**
- `docs/SYNTHESIS.md` — What EXIT is, what it isn't, the competitive gap
- `docs/EXIT_SCHEMA.md` — The data structure (7 core fields, 6 optional modules)
- `docs/EXIT_CEREMONY.md` — The state machine (7 states, 3 ceremony paths)
- `docs/AGENT_LINEAGE.md` — Agent identity, successor appointment, continuity proofs

**This document is the map. Those are the territory.**

---

## 2. The One-Sentence Version

EXIT is a **verifiable transition marker** — the authenticated declaration of departure that preserves continuity across contexts, without custody of assets, identity, or content.

---

## 3. Architectural Principles

These are non-negotiable. Every design decision must pass through them.

| # | Principle | Implication |
|---|-----------|-------------|
| 1 | **Non-custodial** | EXIT references external state but never contains it. No data, no funds, no identity stored. |
| 2 | **Always available** | EXIT must work with zero cooperation from the origin system. A hostile or dead origin cannot prevent exit. |
| 3 | **Offline-verifiable** | A third party can verify an EXIT marker years later without the origin being live. |
| 4 | **Agent-native** | Designed for autonomous agents first. Works for humans too, but agents are the hard case. |
| 5 | **Minimal core, optional extensions** | 7 mandatory fields. Everything else is a module. Valid markers are small (~300 bytes). |
| 6 | **Envelope-agnostic** | EXIT defines semantics. Packaging (VC, JWT, CBOR) is a separate concern. |
| 7 | **Irreversible** | There is no undo. EXIT markers are facts. Returning is a new JOIN, not a reversal. |

---

## 4. The EXIT Marker (Data Structure)

### Core Fields

Every valid EXIT marker contains exactly these 7 fields:

```json
{
  "@context": "https://cellar-door.org/exit/v1",
  "id":        "exit:sha256:...",
  "subject":   "did:key:z6Mk...",
  "origin":    "did:web:marketplace.example",
  "timestamp": "2026-02-11T01:00:00Z",
  "exitType":  "voluntary",
  "status":    "good_standing",
  "proof": {
    "type": "Ed25519Signature2020",
    "created": "2026-02-11T01:00:00Z",
    "verificationMethod": "did:key:z6Mk...#keys-1",
    "proofValue": "z3FXQqF..."
  }
}
```

| Field | Type | Purpose |
|-------|------|---------|
| `id` | URI / content-addressed hash | Globally unique. Allows reference, deduplication, revocation. |
| `subject` | DID / agent URI / key fingerprint | Who is leaving. A pointer to identity, not identity itself. |
| `origin` | URI / domain identifier | What is being left. Scopes the departure. |
| `timestamp` | ISO 8601 UTC | When. Ordering, replay detection, validity windows. |
| `exitType` | `voluntary` \| `forced` \| `emergency` | Nature of departure. Changes downstream interpretation. |
| `status` | `good_standing` \| `disputed` \| `unverified` | Standing at departure. Minimal reputation portability. |
| `proof` | Signature object | Cryptographic authentication. Always subject-signed. |

**Remove any one field and the marker breaks.** See EXIT_SCHEMA.md for field-by-field rationale.

### Optional Modules

| Module | Purpose | When to Include |
|--------|---------|-----------------|
| **A: Lineage** | Successor, predecessor, continuity proof, lineage chain | Agent migration, session handoff, forking |
| **B: State Snapshot** | State hash, location, schema, obligations | Financial exits, DAO departures, dispute-prone contexts |
| **C: Dispute Bundle** | Active disputes, evidence hash, challenge window, counterparty acks | Contested departures, arbitration contexts |
| **D: Economic** | Asset manifest, settled/pending obligations, exit fee | Financial platforms, treasuries, escrow |
| **E: Metadata** | Reason, narrative, tags, locale | Human-readable contexts, UIs |
| **F: Cross-Domain** | On-chain anchors, registry entries | High-assurance, regulatory, cross-chain |

### Size Profile

| Configuration | Size | Use Case |
|--------------|------|----------|
| Core only | 300–500 B | Agent session exit, quick departure |
| Core + Lineage | 500–800 B | Agent migration with successor |
| Core + State + Dispute | 800–1500 B | DAO exit with evidence |
| Core + all modules | 2–4 KB | Full ceremony with economic settlement |

---

## 5. The EXIT Ceremony (State Machine)

EXIT is a ceremony — a structured multi-step ritual — not a single event. This is the key differentiator.

### States

```
                    ┌─────────────────────────────────────────┐
                    │           EMERGENCY PATH                │
                    │                                         │
                    ▼                                         │
┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   │   ┌──────────┐
│  ALIVE   ├──►│  INTENT  ├──►│ SNAPSHOT ├──►│  OPEN    ├───┼──►│  FINAL   │
└──────────┘   └──────────┘   └──────────┘   └────┬─────┘   │   └────┬─────┘
                                                   │         │        │
                                                   ▼         │        ▼
                                              ┌──────────┐   │   ┌──────────┐
                                              │CONTESTED ├───┘   │ DEPARTED │
                                              └──────────┘       └──────────┘
```

| State | Who Acts | Artifact Produced |
|-------|----------|-------------------|
| ALIVE | — | None (default state) |
| INTENT | Subject | Signed ExitIntent declaration |
| SNAPSHOT | Subject and/or Origin | State hash + location reference |
| OPEN | Origin, counterparties | Challenge window parameters |
| CONTESTED | Challenger, Subject, Arbiter | Challenge, response, ruling (all by reference) |
| FINAL | Subject (+ optional co-signers) | **The EXIT marker** |
| DEPARTED | — | Terminal. Marker is in the wild. |

### Three Ceremony Paths

| Path | Steps | Duration | When |
|------|-------|----------|------|
| **Full cooperative** | ALIVE → INTENT → SNAPSHOT → OPEN → FINAL → DEPARTED | Minutes to days | Planned departure, cooperative origin |
| **Unilateral** | ALIVE → INTENT → SNAPSHOT → FINAL → DEPARTED | Seconds | Non-cooperative or absent origin |
| **Emergency** | ALIVE → FINAL → DEPARTED | Milliseconds | Context dying, host crash, hostile termination |

### Critical Design Rules

1. **Contests don't block exit.** A dispute changes the `status` field to `disputed`. It does not prevent the marker from being created. This prevents denial-of-exit attacks.
2. **Only the Subject is always required.** The ceremony completes with zero cooperation from anyone else.
3. **DEPARTED is terminal.** No undo. Return is a new JOIN.
4. **Emergency markers can be upgraded later.** Add co-signatures, state references, or standing attestations after the fact by linking to the original marker ID.

---

## 6. Agent Lineage

Agents face the hardest version of the exit problem: no physical body, no persistent memory, keys tied to ephemeral contexts. EXIT provides the crossing event that makes identity, lineage, and reputation portable.

### Identity Reference

The `subject` field accepts (in order of preference): `did:key`, `did:peer`, `agent://` URI, `did:web`, or raw key fingerprint. EXIT references identity — it does not issue or manage it.

### Successor Appointment

Three trust levels:

| Level | Mechanism | Trust | When |
|-------|-----------|-------|------|
| Self-appointed | Subject designates successor in Module A | Low | Emergency, hostile origin |
| Cross-signed | Successor co-signs an acceptance attestation | Medium | Planned migration |
| Witnessed | Third party attests to the transition | High | High-stakes, reputation-carrying exits |

Deferred appointment is supported: create the EXIT marker without a successor, then publish a **SuccessorAmendment** later (must be signed by the original key).

### Continuity Proofs

Four proof types, from strongest to weakest:

| Proof Type | Mechanism | Strength |
|-----------|-----------|----------|
| **Key Rotation Binding** | Old key signs "new key is my successor" | Strongest — cryptographic, unforgeable |
| **Lineage Hash Chain** | Merkle chain of all rotation bindings from genesis | Strong — accumulated trust, auditable |
| **Delegation Token** | Scoped capability transfer (not same-entity, but authorized) | Medium — for forking, organizational succession |
| **Behavioral Attestation** | Other agents vouch for behavioral consistency | Weakest — last resort after key loss |

### Emergency Preparedness

Agents in volatile environments should maintain a **rotation escrow**: a pre-signed key rotation binding stored on a root-of-trust machine separate from the operational context. This ensures the emergency exit path can include a successor designation even when the operational context is destroyed.

---

## 7. Verification Model

**Layered trust, subject-signed floor.**

| Layer | Signer | Required? | What It Proves |
|-------|--------|-----------|----------------|
| Core | Subject | **Yes** | "I left. Here is my signed proof." |
| Origin co-signature | Origin system | No | "We confirm this departure and the stated standing." |
| Witness attestation | Neutral third party | No | "We observed this ceremony and attest to its integrity." |
| Successor acceptance | Designated successor | No | "I acknowledge and accept this appointment." |

Verifiers decide what trust level they require. Some will accept self-signed markers. Others will demand origin co-signatures. The protocol accommodates both without making the higher bar mandatory.

**Self-attestation of status:** The `status` field is always subject-attested in core. An optional `originStatus` in Module C carries the origin's view. Verifiers see both perspectives. This prevents both gaming (subject inflates standing) and retaliation (origin downgrades standing as punishment).

---

## 8. What EXIT Does NOT Do

This section is as important as the rest. Scope discipline is what makes EXIT a primitive rather than a platform.

| Not This | That's This | EXIT's Relationship |
|----------|-------------|-------------------|
| Identity management | NAME / DID systems | EXIT *references* identity via `subject` |
| Reputation scoring | MARK / VC systems | EXIT *enables portability* via `status` + receipts |
| Memory persistence | RECORD / agent memory | EXIT *anchors transitions* that memory systems reference |
| Asset transfer | Migration tooling, bridges | EXIT *produces the marker* that authorizes transfer |
| Dispute resolution | Arbitration, courts | EXIT *preserves evidence* and records disputes |
| Enforcement | Slashing, penalties | EXIT is the *escape hatch* that bounds enforcement |
| Transport | Data migration, APIs | EXIT *marks the crossing*; transport happens elsewhere |

---

## 9. Open Decisions (For Mayor)

Three architectural decisions were flagged in EXIT_SCHEMA.md. They need Mayor input before implementation.

### Decision 1: Envelope Format
| Option | Description | Architect Recommendation |
|--------|-------------|------------------------|
| A | W3C Verifiable Credential envelope | — |
| B | Custom JSON-LD context | — |
| **C** | **Dual format** (standalone JSON-LD + VC wrapper profile) | **Recommended** |

Rationale: EXIT should be self-contained and not depend on VC ecosystem. But a VC profile from day one enables interop.
**Research spike cd-1jg (Scholar) will test VC envelope fit and inform this decision.**

### Decision 2: Verification Model
| Option | Description | Architect Recommendation |
|--------|-------------|------------------------|
| A | Subject-signed only | — |
| B | Co-signed (subject + origin) | — |
| **C** | **Layered** (subject mandatory, co-signatures optional) | **Recommended** |

Rationale: EXIT must work in the worst case (hostile origin, emergency). Co-signatures add trust but can never be required.

### Decision 3: Status Field Semantics
| Option | Description | Architect Recommendation |
|--------|-------------|------------------------|
| A | Self-attested status only | — |
| B | Origin-attested status only | — |
| **C** | **Multi-source** (self-attested core + optional origin perspective) | **Recommended** |

Rationale: Prevents both gaming and retaliation. Downstream systems set their own trust policies.

---

## 10. Implementation Roadmap

### Phase 1: Architecture (this document) — COMPLETE
- [x] Synthesis of source materials (cd-1zc.1)
- [x] Core schema design (cd-1zc.2)
- [x] Ceremony state machine (cd-1zc.3)
- [x] Agent lineage spec (cd-1zc.4)
- [x] Research spike identification (cd-1zc.5)
- [x] Architecture capstone (this document)

### Phase 1b: Research Spikes — IN PROGRESS (Scholar + Polecats)
- [ ] cd-1zc.7 (P1) Agent memory persistence + EXIT user story
- [ ] cd-1zc.6 (P2) Partial exit scoping
- [ ] cd-t9n (P2) Moloch ragequit state machine extraction
- [ ] cd-1jg (P2) EXIT as W3C VC envelope fit test
- [ ] cd-48g (P2) DID method catalog for agent key rotation
- [ ] cd-8xy (P2) TypeScript prototype: sign/verify EXIT marker

### Phase 2: Prototype (pending Mayor decisions + spike results)
- [ ] Resolve envelope format decision (informed by cd-1jg)
- [ ] Resolve DID method recommendation (informed by cd-48g)
- [ ] Build minimal EXIT marker library (informed by cd-8xy)
- [ ] Implement emergency exit path (the simplest path — sign and go)
- [ ] Implement full ceremony path with state machine
- [ ] Test: voluntary exit, emergency exit, successor handoff, tamper rejection

### Phase 3: Integration
- [ ] Agent-native bindings (agent:// resolver, DID resolution)
- [ ] VC wrapper profile (if Decision 1 → C)
- [ ] On-chain anchoring (Module F, for high-assurance contexts)
- [ ] Cross-system verification (offline-first, no origin dependency)

---

## 11. Dependency Map

```
                    ┌───────────────────────────┐
                    │  EXIT Primitive (this spec) │
                    └─────┬────────────┬────────┘
                          │            │
              ┌───────────▼──┐   ┌─────▼──────────┐
              │ DID / agent://│   │ Crypto Libraries │
              │ (identity     │   │ (Ed25519, secp)  │
              │  resolution)  │   │                  │
              └──────────────┘   └─────────────────┘

  EXIT depends on:
  - A DID method (or agent:// resolver) for subject identification
  - A signing algorithm for proof creation/verification
  - A content-addressing scheme for marker IDs (SHA-256)

  EXIT does NOT depend on:
  - Any specific blockchain or ledger
  - Any specific VC library or wallet
  - Any specific storage or transport layer
  - Any specific platform or origin system
  - The origin system being online or cooperative
```

---

## 12. Glossary

| Term | Definition |
|------|-----------|
| **EXIT marker** | The signed artifact produced by the ceremony. The primary output of the EXIT primitive. |
| **Subject** | The entity departing. Always the primary signer. |
| **Origin** | The system being exited. May cooperate, be hostile, or be offline. |
| **Successor** | Entity designated to inherit continuity from the subject. |
| **Ceremony** | The multi-step ritual of departure (INTENT → SNAPSHOT → OPEN → FINAL). |
| **Emergency exit** | Single-step ceremony bypass: sign marker and go. Produces a valid but lower-trust marker. |
| **Continuity proof** | Cryptographic evidence binding a successor to its predecessor. |
| **Lineage chain** | Accumulated history of key rotations from genesis to current identity. |
| **Root-of-trust** | A machine or storage location separate from the operational context, holding emergency keys. |
| **SuccessorAmendment** | Post-hoc designation of a successor, linked to an existing EXIT marker. |
| **Challenge window** | Optional period during which the origin or counterparties can contest the exit. |

---

*This is a living document. It will be updated as research spikes complete and Mayor decisions are made.*
