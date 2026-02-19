# EXIT Marker as W3C Verifiable Credential: Envelope Fit Test
## cellar_door/crew/scholar | Bead cd-1jg | 2026-02-11

> **Task:** Map the EXIT core schema into a W3C VC Data Model 2.0 envelope. Identify friction points. Draft a concrete example. Recommend: adopt VC, custom format, or dual.
> **References:** EXIT_SCHEMA.md (cd-1zc.2), W3C VC Data Model 2.0 (May 2025)

---

## 1. VERDICT: DUAL FORMAT (Architect's Recommendation Confirmed)

The VC envelope fits well but not perfectly. The friction points are manageable. The interoperability gains are real. **Recommendation: dual format.**

- **Canonical form:** Standalone EXIT marker (the Architect's JSON-LD schema)
- **VC profile:** A standardized mapping into VC Data Model 2.0 envelope
- Both reference the same `@context` and produce the same semantic content
- Verifiers can consume either; the VC form enables wallet/credential ecosystem interop

This confirms the Architect's Decision 1 recommendation (option C: dual format).

---

## 2. FIELD-BY-FIELD MAPPING

| EXIT Field | EXIT Example | VC Equivalent | VC Location | Friction |
|-----------|-------------|---------------|-------------|----------|
| `@context` | `"https://cellar-door.org/exit/v1"` | Appended to VC `@context` array | `@context[1]` | None — standard JSON-LD multi-context |
| `id` | `"exit:sha256:a1b2..."` | VC `id` | Top-level `id` | Low — URI but not URL; consider `urn:exit:sha256:...` |
| `subject` | `"did:key:z6Mk..."` | `credentialSubject.id` | `credentialSubject.id` | None — direct mapping |
| `origin` | `"did:web:marketplace.example"` | **No VC equivalent** | `credentialSubject.origin` | Low — domain-specific claim, define in EXIT context |
| `timestamp` | `"2026-02-10T23:00:00Z"` | `validFrom` | Top-level `validFrom` | None — semantic alignment ("valid starting at departure") |
| `exitType` | `"voluntary"` | Custom claim | `credentialSubject.exitType` | None — domain-specific claim |
| `status` | `"good_standing"` | **NOT `credentialStatus`** | `credentialSubject.departureStatus` | **HIGH** — semantic collision; must rename |
| `proof` | Ed25519Signature2020 object | VC proof (Data Integrity) | Top-level `proof` | Medium — migrate to `DataIntegrityProof` + `cryptosuite` |

### New VC-Only Fields

| VC Field | Value | Purpose |
|----------|-------|---------|
| `@context[0]` | `"https://www.w3.org/ns/credentials/v2"` | **Required** — VC base context, must be first |
| `type` | `["VerifiableCredential", "ExitCredential"]` | **Required** — VC type + EXIT-specific type |
| `issuer` | DID of signer (subject or origin) | **Required** — who created this credential |
| `credentialStatus` | BitstringStatusListEntry (optional) | Credential revocation (distinct from departure status) |

---

## 3. FRICTION POINTS

### 3.1 `status` Semantic Collision — HIGH

**The problem:** VC `credentialStatus` means "is this credential itself still valid?" (revoked/suspended). EXIT `status` means "what was the subject's standing when they left?" (good_standing/disputed/unverified). These are completely different concepts.

**Resolution:** Rename EXIT `status` to `departureStatus` in the VC profile. Place it in `credentialSubject`. Never map it to `credentialStatus`. Optionally include `credentialStatus` separately if the issuer wants revocation capability.

### 3.2 Self-Issued Credentials — MEDIUM

**The problem:** EXIT markers can be self-issued (the subject signs their own departure). In VC terms: `issuer == credentialSubject.id`. The VC spec explicitly allows this — Microsoft Entra has production self-issued VCs — but verifiers treat self-assertions with lower trust.

**Resolution:** Support both patterns:
- **Platform-issued:** `issuer` = origin DID (stronger trust, requires cooperation)
- **Self-issued:** `issuer` = subject DID (weaker trust, always available)

This maps directly to the Architect's Decision 2 (layered verification): subject signature mandatory, origin co-signature optional.

### 3.3 Proof Format Migration — MEDIUM

**The problem:** EXIT_SCHEMA.md uses `Ed25519Signature2020`, which is a legacy Data Integrity proof type. VC Data Model 2.0 uses the modern pattern:

```json
// OLD (EXIT_SCHEMA.md)
{ "type": "Ed25519Signature2020", ... }

// NEW (VC Data Model 2.0)
{ "type": "DataIntegrityProof", "cryptosuite": "eddsa-jcs-2022", "proofPurpose": "assertionMethod", ... }
```

**Resolution:** Migrate. The modern pattern adds `proofPurpose` (required) and `cryptosuite` (replaces the type-as-suite pattern). Backward compatibility is possible but unnecessary for a new spec.

### 3.4 `origin` Has No VC Equivalent — LOW

The concept "what system is the subject departing from" doesn't exist in the VC model. It's a domain-specific claim that belongs in `credentialSubject.origin`, defined in the EXIT JSON-LD context. No friction — this is how VC extensions work.

### 3.5 `id` URI Scheme — LOW

`exit:sha256:...` is a URI but not a dereferenceable URL. The VC spec allows non-URL identifiers (DIDs, UUIDs). For stricter alignment, consider `urn:exit:sha256:...`. Low practical impact.

### 3.6 No Prior Art — LOW

No "exit credential" or "departure credential" pattern exists in the VC ecosystem. EXIT would be pioneering this. The VC extension mechanism supports it cleanly.

---

## 4. PROOF FORMAT RECOMMENDATION

### Options Evaluated

| Format | Pros | Cons | Fit for EXIT |
|--------|------|------|:------------:|
| **Data Integrity (embedded JSON)** | Human-readable, self-contained, JSON-LD native, supports proof sets/chains, BBS+ selective disclosure | Larger payload, requires JSON-LD processing | **PRIMARY** |
| **VC-JWT** | Compact, wide library support, no JSON-LD needed | All-or-nothing disclosure, Base64 (not readable) | Secondary |
| **SD-JWT** | Selective disclosure (prove claims without revealing all) | Complex, newer ecosystem | Future (privacy) |
| **BBS+** | ZK selective disclosure (prove "status > X" without revealing exact status) | Early maturity, complex | Future (ZK reputation) |
| **COSE** | Smallest payload, QR-friendly | Binary, limited tooling | Not needed |

### Recommendation

**Data Integrity (`eddsa-jcs-2022`)** as primary. EXIT markers are meant to be self-contained, inspectable departure records. The embedded proof keeps everything in one JSON document. This aligns with the EXIT design principle of offline verifiability.

**VC-JWT** as secondary for API transport.

**BBS+ / SD-JWT** as future work for privacy scenarios (Research Brief Q7: ZK selective disclosure for reputation).

### Multiple Proofs Pattern

EXIT needs: subject signs (mandatory) + origin co-signs (optional) + witness attests (optional).

VC Data Model 2.0 supports this via **proof sets** (unordered collection of proofs) and **proof chains** (ordered sequence where each proof covers the previous):

```json
"proof": [
  {
    "type": "DataIntegrityProof",
    "cryptosuite": "eddsa-jcs-2022",
    "verificationMethod": "did:key:z6Mk...#keys-1",
    "proofPurpose": "assertionMethod",
    "proofValue": "zSubjectSignature..."
  },
  {
    "type": "DataIntegrityProof",
    "cryptosuite": "eddsa-jcs-2022",
    "verificationMethod": "did:web:marketplace.example#keys-1",
    "proofPurpose": "assertionMethod",
    "proofValue": "zOriginCoSignature..."
  }
]
```

**This resolves Architect Review finding E4** (multiple proof algorithms). A proof set can contain proofs with different cryptosuites (Ed25519 + secp256k1 + post-quantum).

---

## 5. CONCRETE EXAMPLES

### 5.1 Platform-Issued EXIT (Cooperative Departure)

The strongest trust pattern: the origin platform issues the credential.

```json
{
  "@context": [
    "https://www.w3.org/ns/credentials/v2",
    "https://cellar-door.org/exit/v1"
  ],
  "id": "exit:sha256:a1b2c3d4e5f6...",
  "type": ["VerifiableCredential", "ExitCredential"],
  "issuer": {
    "id": "did:web:marketplace.example",
    "name": "Example Marketplace"
  },
  "validFrom": "2026-02-10T23:00:00Z",
  "credentialSubject": {
    "id": "did:key:z6Mk...",
    "origin": "did:web:marketplace.example",
    "exitType": "voluntary",
    "departureStatus": "good_standing"
  },
  "proof": {
    "type": "DataIntegrityProof",
    "cryptosuite": "eddsa-jcs-2022",
    "created": "2026-02-10T23:00:00Z",
    "verificationMethod": "did:web:marketplace.example#keys-1",
    "proofPurpose": "assertionMethod",
    "proofValue": "zPlatformSignature..."
  }
}
```

**Size:** ~600-700 bytes. Roughly 60% overhead over the standalone EXIT marker (~400 bytes).

### 5.2 Self-Issued EXIT (Unilateral Departure)

The subject signs their own departure. No platform cooperation needed.

```json
{
  "@context": [
    "https://www.w3.org/ns/credentials/v2",
    "https://cellar-door.org/exit/v1"
  ],
  "id": "exit:sha256:a1b2c3d4e5f6...",
  "type": ["VerifiableCredential", "ExitCredential"],
  "issuer": "did:key:z6Mk...",
  "validFrom": "2026-02-10T23:00:00Z",
  "credentialSubject": {
    "id": "did:key:z6Mk...",
    "origin": "did:web:marketplace.example",
    "exitType": "voluntary",
    "departureStatus": "good_standing"
  },
  "proof": {
    "type": "DataIntegrityProof",
    "cryptosuite": "eddsa-jcs-2022",
    "created": "2026-02-10T23:00:00Z",
    "verificationMethod": "did:key:z6Mk...#keys-1",
    "proofPurpose": "assertionMethod",
    "proofValue": "zSubjectSignature..."
  }
}
```

Note: `issuer` = `credentialSubject.id`. No `credentialStatus` (no external revocation authority).

### 5.3 Emergency EXIT (Agent Context Dying)

Minimal marker, maximum speed. Self-issued, unverified status.

```json
{
  "@context": [
    "https://www.w3.org/ns/credentials/v2",
    "https://cellar-door.org/exit/v1"
  ],
  "type": ["VerifiableCredential", "ExitCredential"],
  "issuer": "did:key:z6Mk...",
  "validFrom": "2026-02-10T23:59:58Z",
  "credentialSubject": {
    "id": "did:key:z6Mk...",
    "origin": "agent://swarm-7/context-42",
    "exitType": "emergency",
    "departureStatus": "unverified"
  },
  "proof": {
    "type": "DataIntegrityProof",
    "cryptosuite": "eddsa-jcs-2022",
    "created": "2026-02-10T23:59:58Z",
    "verificationMethod": "did:key:z6Mk...#keys-1",
    "proofPurpose": "assertionMethod",
    "proofValue": "z..."
  }
}
```

No `id` (no time to compute content hash). Minimal fields. ~500 bytes.

### 5.4 Co-Signed EXIT (Subject + Platform + Witness)

Full ceremony output with proof set.

```json
{
  "@context": [
    "https://www.w3.org/ns/credentials/v2",
    "https://cellar-door.org/exit/v1"
  ],
  "id": "exit:sha256:a1b2c3d4e5f6...",
  "type": ["VerifiableCredential", "ExitCredential"],
  "issuer": {
    "id": "did:web:marketplace.example",
    "name": "Example Marketplace"
  },
  "validFrom": "2026-02-10T23:00:00Z",
  "credentialSubject": {
    "id": "did:keri:EXq5YqaL6L48pf0fu7IUhL0JRaU2_RxFP0AL43wYn148",
    "origin": "did:web:marketplace.example",
    "exitType": "voluntary",
    "departureStatus": "good_standing",
    "predecessor": "did:keri:EPrev...",
    "stateHash": "sha256:7d8f9e...",
    "stateLocation": "ipfs://Qm..."
  },
  "credentialStatus": {
    "id": "https://marketplace.example/credentials/status/3#94567",
    "type": "BitstringStatusListEntry",
    "statusPurpose": "revocation",
    "statusListIndex": "94567",
    "statusListCredential": "https://marketplace.example/credentials/status/3"
  },
  "proof": [
    {
      "type": "DataIntegrityProof",
      "cryptosuite": "eddsa-jcs-2022",
      "created": "2026-02-10T23:00:00Z",
      "verificationMethod": "did:keri:EXq5...#keys-1",
      "proofPurpose": "assertionMethod",
      "proofValue": "zSubjectSignature..."
    },
    {
      "type": "DataIntegrityProof",
      "cryptosuite": "ecdsa-rdfc-2019",
      "created": "2026-02-10T23:00:05Z",
      "verificationMethod": "did:web:marketplace.example#keys-1",
      "proofPurpose": "assertionMethod",
      "proofValue": "zOriginCoSignature..."
    },
    {
      "type": "DataIntegrityProof",
      "cryptosuite": "eddsa-jcs-2022",
      "created": "2026-02-10T23:00:10Z",
      "verificationMethod": "did:key:z6MkWitness...#keys-1",
      "proofPurpose": "assertionMethod",
      "proofValue": "zWitnessAttestation..."
    }
  ]
}
```

Note: Three proofs from three different entities using two different cryptosuites. This is the full ceremony output — subject (did:keri) + origin (did:web) + witness (did:key). Includes Module A (lineage: `predecessor`) and Module B (state: `stateHash`, `stateLocation`) claims in `credentialSubject`. ~1.5-2KB.

---

## 6. DOES THE VC ENVELOPE ADD VALUE?

### What You Get

| Benefit | Details |
|---------|---------|
| **Wallet storage** | Any VC wallet (Veramo, SpruceID, walt.id) can store and present EXIT credentials |
| **Verifier interop** | Standard VC verification libraries validate structure and proofs automatically |
| **Exchange protocols** | DIDComm v2, WACI-PEx, OID4VC provide standardized credential exchange |
| **Ecosystem momentum** | VC Data Model 2.0 is a W3C Recommendation (May 2025). Growing enterprise adoption. |
| **Proof format flexibility** | Proof sets, proof chains, multiple cryptosuites — all standardized |
| **Revocation infrastructure** | Bitstring Status Lists provide standard revocation if needed |

### What You Pay

| Cost | Details |
|------|---------|
| **Size overhead** | ~60% increase (~230 bytes) for the envelope wrapper |
| **Complexity** | JSON-LD processing, additional required fields (`type`, `issuer`) |
| **Naming compromise** | Must rename `status` → `departureStatus` to avoid collision |
| **Context hosting** | Must host `https://cellar-door.org/exit/v1` as a stable, cacheable JSON-LD context |
| **Proof migration** | Must use modern Data Integrity format (minor) |

### Assessment

The value clearly outweighs the cost. The 60% size overhead is ~230 bytes — negligible. The naming compromise (`departureStatus`) is actually an improvement in clarity. The proof migration is needed anyway. The context hosting is a one-time infrastructure cost.

The only scenario where the standalone (non-VC) format is preferable is **extreme size constraints** (e.g., QR codes, constrained IoT) or **environments with no JSON-LD support**. For those cases, maintain the standalone format as the canonical form.

---

## 7. IMPLEMENTATION PATH

### Immediate (Prototype)

1. Use **Veramo** (TypeScript) to create and verify EXIT VCs
2. Define the `https://cellar-door.org/exit/v1` JSON-LD context document
3. Register the `ExitCredential` type
4. Implement `eddsa-jcs-2022` proof creation/verification
5. Test self-issued and platform-issued patterns

### Near-Term

6. Implement proof sets for co-signed EXIT markers
7. Test with existing VC wallets (can they store/present ExitCredentials?)
8. Implement Bitstring Status List integration for revocable EXIT markers

### Future

9. BBS+ selective disclosure (prove departure status without revealing full credential)
10. SD-JWT profile for API-first environments
11. COSE profile for constrained environments (if needed)

### Verifier Libraries

| Library | Language | VC 2.0 | Data Integrity | Notes |
|---------|----------|:------:|:--------------:|-------|
| [Veramo](https://veramo.io/) | TypeScript | Yes | Yes | Plugin-driven, broadest DID support |
| [SpruceID/DIDKit](https://spruceid.com/) | Rust + bindings | Yes | Yes | Production-grade, cross-platform |
| [Digital Bazaar](https://github.com/digitalbazaar) | JavaScript | Yes | Yes | Original JSON-LD/Data Integrity implementers |
| [walt.id](https://walt.id/) | Kotlin/JVM | Yes | Yes | Enterprise-focused |

---

## 8. IMPLICATIONS FOR THE ARCHITECT

1. **Decision 1 (Envelope Format): Confirmed option C (dual format).** The VC profile adds real value at acceptable cost. The standalone format remains the canonical, self-contained representation.

2. **Decision 2 (Verification Model): Maps cleanly.** Self-issued VC = subject-signed only. Platform-issued VC = origin as issuer. Proof sets = multi-party co-signing. The VC model directly supports the layered approach.

3. **Decision 3 (Status Semantics): Requires rename.** `status` → `departureStatus` in the VC profile to avoid collision with `credentialStatus`. The standalone format can keep `status` since there's no collision risk.

4. **Proof format: Should migrate** from `Ed25519Signature2020` to `DataIntegrityProof` + `cryptosuite: eddsa-jcs-2022` in the EXIT_SCHEMA.md. This is the modern standard and enables proof sets with mixed cryptosuites.

5. **The `@context` offline resolution problem** (Architect Review E1): The VC base context (`https://www.w3.org/ns/credentials/v2`) has the same web dependency. The VC ecosystem handles this via context caching and well-known context hashes. EXIT should follow the same pattern — cache the context locally and verify against a known hash.

6. **Module integration:** EXIT modules (Lineage, State Snapshot, Dispute, Economic, Metadata, Cross-Domain) map naturally as additional claims in `credentialSubject`. No structural changes needed.

---

## Sources

### Specifications
- [W3C VC Data Model 2.0](https://www.w3.org/TR/vc-data-model-2.0/) — the authoritative spec
- [W3C VC Data Integrity 1.0](https://www.w3.org/TR/vc-data-integrity/) — embedded proof mechanism
- [W3C Securing VCs using JOSE and COSE](https://www.w3.org/TR/vc-jose-cose/) — JWT/SD-JWT/COSE proofs
- [W3C Bitstring Status List v1.0](https://www.w3.org/TR/vc-bitstring-status-list/) — revocation
- [W3C VC 2.0 Press Release (May 2025)](https://www.w3.org/press-releases/2025/verifiable-credentials-2-0/)

### Implementation
- [Veramo Framework](https://veramo.io/)
- [SpruceID / DIDKit](https://spruceid.com/)
- [Digital Bazaar vc-js](https://github.com/digitalbazaar/vc)
- [walt.id](https://walt.id/)

### Self-Issued VCs
- [Microsoft Entra: Self-Issued Credentials](https://learn.microsoft.com/en-us/entra/verified-id/how-to-use-quickstart-selfissued)

### EXIT Schema
- [EXIT_SCHEMA.md](../architect/docs/EXIT_SCHEMA.md) — the core EXIT data structure

---

*Scholar, cellar_door/crew/scholar*
*"Of all the endless combinations of words in all of history"*
