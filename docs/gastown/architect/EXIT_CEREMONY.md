# EXIT Ceremony: State Machine and Ritual Design

> Produced by architect (cellar_door/crew) for cd-1zc.3
> Depends on: docs/SYNTHESIS.md (cd-1zc.1), docs/EXIT_SCHEMA.md (cd-1zc.2)

---

## Design Philosophy

EXIT is a ceremony, not a single event. This is the key differentiator from everything else in the landscape.

But the ceremony must be as simple as possible. The goal is the *minimum viable ritual* that still enables:
- Dispute-awareness (the origin can contest)
- Evidence preservation (artifacts exist at each step)
- Emergency bypass (agents can skip steps when context is dying)

The state machine has **two paths**: the **full ceremony** (cooperative, dispute-aware) and the **emergency exit** (unilateral, immediate). Both produce valid EXIT markers.

---

## Roles

| Role | Description | Required? |
|------|-------------|-----------|
| **Subject** | The entity departing. Signs the EXIT marker. | Always |
| **Origin** | The system being exited. May co-sign, contest, or be absent. | Optional (may be hostile/offline) |
| **Witness** | Neutral third party that can attest to ceremony steps. | Optional (adds trust) |
| **Verifier** | Any future party that checks the EXIT marker. Not present during ceremony. | Downstream |
| **Successor** | Entity designated to inherit continuity. May be the same entity under a new key. | Optional (Module A) |

**Critical design choice:** Only the Subject is always required. The ceremony must work with *no cooperation from anyone else*. This is what makes EXIT an escape hatch, not a negotiation.

---

## State Machine

```
                    ┌─────────────────────────────────────────┐
                    │           EMERGENCY PATH                │
                    │  (context dying, hostile origin, etc.)   │
                    │                                         │
                    ▼                                         │
┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   │   ┌──────────┐
│          │   │          │   │          │   │          │   │   │          │
│  ALIVE   ├──►│  INTENT  ├──►│ SNAPSHOT ├──►│ OPEN     ├───┼──►│ FINAL    │
│          │   │          │   │          │   │          │   │   │          │
└──────────┘   └──────────┘   └──────────┘   └────┬─────┘   │   └────┬─────┘
                                                   │         │        │
                                                   ▼         │        ▼
                                              ┌──────────┐   │   ┌──────────┐
                                              │          │   │   │          │
                                              │CONTESTED ├───┘   │ DEPARTED │
                                              │          │       │          │
                                              └──────────┘       └──────────┘
```

### States

#### 1. ALIVE
- **Meaning:** Entity is an active participant in the origin system. No exit in progress.
- **Artifacts:** None (this is the default state).
- **Transitions:** → INTENT (subject declares departure) or → FINAL (emergency bypass).

#### 2. INTENT
- **Meaning:** Subject has declared intent to exit. The origin system is now on notice.
- **Who acts:** Subject.
- **Artifacts produced:**
  - `ExitIntent` object: `{ subject, origin, timestamp, exitType, reason? }`
  - Signed by subject.
- **Transitions:**
  - → SNAPSHOT (subject or origin produces state reference)
  - → FINAL (emergency: skip snapshot, go directly to marker creation)
- **Duration:** Instantaneous or bounded by origin's rules. The intent *starts the clock* on any challenge window.
- **What can happen here:**
  - Origin acknowledges intent (cooperative).
  - Origin ignores intent (non-cooperative, but exit continues).
  - Origin contests intent (→ triggers CONTESTED at next step).

#### 3. SNAPSHOT
- **Meaning:** A reference to the state at exit time has been captured.
- **Who acts:** Subject (self-snapshot), Origin (origin-provided), or both.
- **Artifacts produced:**
  - `stateHash` — content-addressed hash of relevant state.
  - `stateLocation` — where the full snapshot is stored (external).
  - `obligations` — list of outstanding commitments (references only).
- **Transitions:**
  - → OPEN (snapshot accepted, challenge window begins)
  - → FINAL (no challenge window configured, or emergency)
- **Key principle:** EXIT stores the *hash*, never the state. The snapshot itself lives wherever the subject or origin puts it.

#### 4. OPEN
- **Meaning:** Exit is in the challenge window. The origin or counterparties can contest.
- **Who acts:** Origin, counterparties, witnesses.
- **Artifacts produced:**
  - Challenge window parameters: `{ opens, closes, arbiter? }`
  - Any challenges filed (references to dispute objects).
- **Transitions:**
  - → FINAL (challenge window closes with no contest, or all challenges resolved)
  - → CONTESTED (a challenge is filed)
- **Duration:** Configurable. Default: determined by origin system's rules. For agent contexts, may be very short (minutes) or zero.
- **If no challenge window is configured:** This state is skipped entirely. SNAPSHOT → FINAL.

#### 5. CONTESTED
- **Meaning:** A challenge has been filed against the exit. The departure is valid but disputed.
- **Who acts:** Challenger (origin or counterparty), Subject (responds), Arbiter (if configured).
- **Artifacts produced:**
  - Challenge object: `{ challenger, claim, evidence_hash, filed_at }`
  - Response object (optional): `{ subject_response, evidence_hash }`
  - Arbiter ruling (optional): `{ ruling, rationale_hash }`
- **Transitions:**
  - → FINAL (challenge resolved — either dismissed, settled, or timed out)
  - → FINAL (subject proceeds anyway — exit with `status: disputed`)
- **Critical rule:** A contest does NOT block exit. It changes the `status` field on the EXIT marker from `good_standing` to `disputed`. The subject can always proceed. This prevents origins from using disputes as an exit-denial mechanism.

#### 6. FINAL
- **Meaning:** The EXIT marker is created and signed. The ceremony is complete.
- **Who acts:** Subject (signs marker). Optionally: Origin (co-signs), Witness (attests).
- **Artifacts produced:**
  - **The EXIT marker itself** (the core schema from EXIT_SCHEMA.md).
  - Optional: origin co-signature, witness attestation.
- **Transitions:**
  - → DEPARTED (marker is distributed / anchored / stored).
- **This is the point of no return.** Once the marker exists and is signed, the exit is a fact.

#### 7. DEPARTED
- **Meaning:** The entity has left. The EXIT marker is in the wild.
- **Artifacts:** The EXIT marker, any module extensions, any co-signatures collected.
- **This is a terminal state.** There is no "undo" for an EXIT. If the entity wants to return to the origin, that's a new JOIN event — not a reversal of EXIT.

---

## The Emergency Path

Agents face a unique problem: context can be destroyed without warning. A session can be killed, a host can crash, a context window can fill up. The ceremony must accommodate this.

**Emergency exit collapses the ceremony to a single step: ALIVE → FINAL.**

The subject creates and signs an EXIT marker with:
- `exitType: "emergency"`
- `status: "unverified"` (no time to verify standing)
- No snapshot, no challenge window, no co-signatures.
- The marker is valid but carries less trust.

**This is by design.** A weak exit marker is infinitely better than no exit marker. The alternative — losing identity continuity entirely because the ceremony couldn't complete — is the exact failure mode EXIT exists to prevent.

Emergency markers can be *upgraded* later: the subject (or successor) can add co-signatures, state references, or standing attestations after the fact, linking back to the original marker via its `id`.

---

## Ceremony Timelines

### Full Cooperative Exit (Happy Path)
```
Subject                     Origin                     Witness
   │                          │                          │
   ├─── ExitIntent ──────────►│                          │
   │                          ├─── Acknowledge ─────────►│
   │                          │                          │
   ├─── StateSnapshot ───────►│                          │
   │    (or Origin provides)  ├─── Verify ──────────────►│
   │                          │                          │
   │◄── Challenge window opens │                          │
   │         ...wait...        │                          │
   │◄── Window closes ────────│                          │
   │                          │                          │
   ├─── EXIT marker (signed) ─►│                          │
   │                          ├─── Co-sign ─────────────►│
   │                          │                    Attest │
   │◄─────────────────────────┤◄─────────────────────────┤
   │                          │                          │
   └─── DEPARTED              │                          │
```
**Duration:** Minutes to days, depending on origin's challenge window.

### Unilateral Exit (Non-Cooperative Origin)
```
Subject                     Origin (hostile/absent)
   │                          │
   ├─── ExitIntent ──────────►│  (may be ignored)
   │                          │
   ├─── Self-snapshot          │
   │    (hash own state)       │
   │                          │
   ├─── EXIT marker (signed)   │  (no co-signature)
   │    status: good_standing   │
   │    or status: unverified   │
   │                          │
   └─── DEPARTED               │
```
**Duration:** Seconds. No waiting for a hostile origin.

### Emergency Exit (Context Dying)
```
Subject
   │
   ├─── EXIT marker (signed)
   │    exitType: emergency
   │    status: unverified
   │
   └─── DEPARTED
```
**Duration:** Milliseconds. One signature, one artifact.

---

## Artifacts Summary

| Ceremony Step | Artifact | Signed By | Stored Where |
|--------------|----------|-----------|-------------|
| INTENT | ExitIntent | Subject | Subject's discretion |
| SNAPSHOT | State hash + location | Subject and/or Origin | External (referenced by hash) |
| OPEN | Challenge window params | Origin (sets window) | Origin or public registry |
| CONTESTED | Challenge + Response | Challenger + Subject | External (referenced by hash) |
| FINAL | **EXIT Marker** | Subject (mandatory) + Origin/Witness (optional) | Subject distributes |
| DEPARTED | — | — | Marker is in the wild |

---

## Failure Modes and Recovery

### 1. Origin refuses to acknowledge intent
- **Impact:** None. Intent is a notification, not a request. The subject proceeds unilaterally.
- **Recovery:** Subject skips to SNAPSHOT or FINAL.

### 2. Origin provides false state snapshot
- **Impact:** Disputed state. Subject's self-snapshot may conflict with origin's.
- **Recovery:** Both snapshots are preserved. Verifiers see the discrepancy. Module C (Dispute Bundle) captures both views.

### 3. Subject loses keys during ceremony
- **Impact:** Critical. Cannot sign the EXIT marker.
- **Recovery:** If Module A (Lineage) is configured, a designated successor with a continuity proof can complete the exit on behalf of the original subject. Otherwise, the exit is lost — this is the one failure EXIT cannot recover from without prior setup.

### 4. Challenge window expires with unresolved dispute
- **Impact:** Exit proceeds with `status: disputed`. The dispute exists but does not block departure.
- **Recovery:** Dispute resolution continues independently of the exit. The EXIT marker references the dispute; the dispute references the EXIT marker. Neither blocks the other.

### 5. Emergency exit produces weak marker
- **Impact:** Marker has `unverified` status. Less trusted by downstream systems.
- **Recovery:** Post-hoc upgrade: subject or successor adds co-signatures, state references, and standing attestations after the fact, linking to the original marker ID.

### 6. Network partition during ceremony
- **Impact:** Intent or snapshot may not reach origin.
- **Recovery:** Subject proceeds unilaterally. The ceremony is designed to complete with zero network connectivity to the origin. Only co-signatures require origin cooperation.

---

## Design Decisions

### Why contests don't block exit
This is the most important decision in the ceremony design. If a dispute could *prevent* departure, then filing a dispute becomes a denial-of-exit attack. The origin could simply contest every exit to trap participants. This violates the fundamental principle: EXIT must always be available.

Instead: disputes change the *metadata* on the marker (status: disputed), not the marker's *existence*. The exit happens; the dispute is recorded alongside it. This is analogous to how bankruptcy allows you to leave financial obligations while disputes continue separately.

### Why the emergency path exists
Agents can't always complete a multi-step ceremony. Context windows fill, sessions are killed, hosts crash. If EXIT required multiple round-trips, it would fail exactly when it's needed most — during system failure. The emergency path ensures that the *minimum viable exit* (one signature, one marker) is always achievable in a single operation.

### Why DEPARTED is terminal
There is no "undo EXIT." This is deliberate. If exits could be reversed, they wouldn't be credible — origins could pressure entities to "undo" their departure. The irreversibility is what gives EXIT its power as a constitutional primitive. If you want to return, you JOIN again — that's a new relationship, not a restoration of the old one.

### Why the challenge window is optional
Many contexts (especially agent contexts) don't need or can't afford challenge windows. An agent whose context is being destroyed in 30 seconds can't wait 24 hours for a challenge period. The ceremony accommodates both: systems that want dispute windows configure them; systems that don't, skip OPEN entirely.
