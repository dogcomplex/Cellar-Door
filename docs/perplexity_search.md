<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# To “exhaustively” search, you want a repeatable pipeline that covers (a) products, (b) protocols/standards, (c) research, and (d) adjacent terms that people use instead of “exit enablement.” Right now, a lot of relevant work is described as portability, interoperability, DID/VC reputation, account abstraction, recovery, lineage, exodus protocols, etc.

1) Define the “shape” so you don’t miss synonyms
Your concept spans 3 clusters:
A. “Exit” / portability / interoperability
keywords: exit protocol, portability, interoperability, ragequit, migration, account recovery, credential portability, platform lock-in
notable: “Exodus Protocols / Preserve exit through portability” language shows up in some infrastructure thinking
B. Identity + credentials + reputation
keywords: DID, verifiable credential, VC data model, portable reputation, reputation credential, anonymous reputation, reputation signature
this is where most “portable reputation” talk lives (W3C VCs, DID ecosystems).
C. Agent continuity / successor / lineage (agent-native)
keywords: agent identity, lineage, successor, continuity proof, agent hash, master citation, vault storage location
there are already a few agent-ish specs/projects talking about successor/lineage semantics.
2) Where to search (to be “exhaustive”)
A. Product / protocol discovery
GitHub / GitLab: search for the clusters above, plus “DID VC wallet”, “credential issuer”, “reputation oracle”, “attestation registry”.
Example seed: agent lineage projects exist on GitHub (e.g., “Master Citation / lineage / successor citations”).
Standards bodies / ecosystems
W3C Verifiable Credentials \& DID ecosystems (many vendors implement pieces).
Web3 identity vendors
cheqd and “verifiable credentials value”/portability narratives show up in that ecosystem.
B. Research discovery
arXiv: anonymous reputation blockchain, reputation signature, portable reputation, agent governance decentralized identity
Springer/MDPI: reputation primitives, privacy-preserving reputation, etc.
C. Conference/presentation discovery
search slides + talks for “exit protocols / exodus protocols / interoperability” (sometimes the best framing is in decks, not products).
3) Concrete query set (copy/paste)
Use these in Google/GitHub search. Run them with and without quotes.
“Exit” framing
"exodus protocol" portability
"preserve exit" portability
interoperability "exit"
ragequit protocol reputation

Credentials / reputation
"verifiable credential" reputation
"portable reputation" verifiable credential
"reputation signature" blockchain
"anonymous reputation" zk

Agent-native lineage
"agent" successor hash
"agent identity" lineage
"master citation" lineage successor
"OP_IDCLAIM" agent

“Attestation registry” / trust plumbing
attestation registry DID VC
credential issuer SDK
proof of compliance zk credential
4) Decide what “counts as competition” vs “adjacent”
Most things you’ll find are one of these:
Identity layer (DID/VC issuers, wallets) — enabler, not competitor
Reputation layer (platform-specific scores) — adjacent
Recovery layer (smart wallet recovery) — adjacent
Exit enablement (process + semantics + receipts + dispute-aware departure) — actual competitor
Your differentiator is \#4: exit as a structured ritual with portable receipts, not “a credential exists.”
5) What to record for each candidate (so your scan is systematic)
For every project/service, capture:
What is portable? (identity / reputation / assets / “membership”)
Who is issuer? (self / third party / DAO / vendor)
Is there successor/lineage semantics? (rare; agent-native)
Does it support dispute-aware exit? (rare)
Is verification possible without the issuer online? (important)
Custody risk (do they hold funds/data?)
ZK support (optional now; moat later)
6) What I’m seeing from the quick web scan
There’s clear identity \& VC ecosystem coverage (portable credentials are mainstream in that world).
There’s lots of “reputation portability” talk (often glossaries/essays), but it’s usually not an operational exit workflow.
There are emerging agent lineage / successor concepts in a few corners, but not obviously packaged as a “leave safely with receipts” service.
“Exodus/exit protocols” framing exists as infrastructure philosophy, which is close to your brand story.
7) If you want the truly exhaustive version
If you tell me which sphere you care about most (pure crypto, agent frameworks, or enterprise identity), I can run a second pass of web searches that enumerates:
the top DID/VC vendors most relevant to “portable reputation”
the main on-chain reputation protocols
the main account-abstraction recovery stacks
anything explicitly calling itself “exit” / “exodus” / “portability protocol”
…but even without narrowing, the method above is the way to not miss the weirdly-named cousins that are actually closest to Cellar Door.

You already have the right shape and almost the full pipeline; what’s left is formalizing it so you (or someone else) can hand this off and get consistent, “exhaustive enough” coverage run on a cadence.

## 1. Clarify the object of the search

Define the thing you’re looking for as:

- A **process + protocol** that:
    - Lets a user or agent leave a platform/system.
    - Produces **portable artifacts** (credentials, logs, proofs, receipts) that can be verified elsewhere.
    - Has at least some notion of **lineage / continuity** (same agent before/after).
    - Optionally includes **dispute-aware** semantics (who said what, when; evidence).

Then classify hits into four buckets (your step 4):

- Identity layer (DID/VC issuers, wallets).
- Reputation layer (scores, attestations).
- Recovery layer (account- or key-recovery stacks).
- **Exit enablement**: anything that treats exit as a first-class workflow with structure, receipts, and verification.

Only the last one is true “competition”; the others are ingredients.

## 2. Lock in the synonym “shape”

Make the search space a fixed set of lexical clusters you always sweep:

- Cluster A – **Exit / portability / interoperability**
    - exit protocol, exodus protocol, preserve exit, portability, interoperability, ragequit, migration, ragequit protocol, exit workflow, “preserve exit through portability”.[^1_1][^1_2]
- Cluster B – **Identity / credentials / reputation**
    - DID, decentralized identifier, verifiable credential, VC data model, portable reputation, reputation credential, reputation oracle, anonymous reputation, reputation signature, attestation registry.[^1_3][^1_4][^1_5]
- Cluster C – **Agent continuity / lineage**
    - agent identity, agent://, agent lineage, successor, continuity proof, master citation, agent hash, context lineage, provenance, non‑human identities.[^1_6][^1_7][^1_8]

Operationally: keep these as a living “lexicon file” you can append to as you discover new weird cousins.

## 3. Fixed query bundles per surface

For each run, you sweep a small, stable query set on each surface, with and without quotes, plus site scoping.

### A. Code/products (GitHub, GitLab, npm, crates, etc.)

Core queries (run on GitHub and GitLab):

- "exodus protocol" portability
- "preserve exit" portability
- interoperability exit OR "exit protocol"
- ragequit protocol reputation
- "verifiable credential" reputation
- "portable reputation" "verifiable credential"
- "reputation oracle" credential
- "anonymous reputation" zk OR "reputation signature" blockchain
- "agent identity" lineage OR "agent" successor hash
- "master citation" lineage successor
- "OP_IDCLAIM" agent
- "attestation registry" DID VC
- "credential issuer" SDK

Then add ecosystem-specific filters:

- org:didcomm OR org:decentralized-identity OR org:hyperledger
- topic:"verifiable-credentials" topic:"decentralized-identity"
- language:ts language:rust language:go for production-ish work.

You can also run a “wallet/protocol surface” pass:

- "DID VC wallet"
- "credential wallet" "reputation"
- "exit" portability "wallet"
- "account abstraction" recovery exit

Record anything that:

- exposes a protocol or SDK (not just a static site),
- has “portability”, “exit”, “ragequit”, “reputation”, or “lineage” in README, docs, or issues.


### B. Standards and vendor ecosystems

Surfaces:

- W3C VCWG and DID WG issues / specs / mailing list archives.
- DIF (Decentralized Identity Foundation) repos and working groups.
- cheqd, Affinidi, Spruce, Disco, Iden3/PolygonID, Veramo, Dock, Bloom, Ceramic, etc.

Queries per surface:

- "portable reputation" site:w3.org OR site:identity.foundation
- "portable reputation" "verifiable credential" site:github.com.[^1_5][^1_3]
- "exit protocol" OR "exodus protocol" site:blockchaincommons.com site:lifewithalacrity.com.[^1_2][^1_9][^1_1]
- "lineage" "verifiable credential" OR "DID" OR "agent".[^1_7][^1_6]
- "attestation registry" DID OR "compliance credential".

This is where you catch Affinidi’s portable reputation reference app, cheqd’s “verifiable AI” credentials, Gordian Clubs, etc..[^1_3][^1_2][^1_5]

### C. Research (arXiv + publishers)

arXiv queries:

- "portable reputation" "verifiable credential"
- "anonymous reputation" blockchain OR "reputation signature" zk
- "agent identity" lineage
- "context lineage" "non-human identities".[^1_6]
- "agent:// URI scheme" "decentralized agent identity".[^1_7]

Publisher queries (Springer, MDPI, IEEE/Xplore, etc.):

- "privacy-preserving reputation" credential
- "reputation primitives" decentralized identity
- "exit protocol" interoperability
- "data portability" verifiable credentials (for GDPR/portability angle).

You’re mostly mining for:

- formal reputation schemes that can be VC-ized.
- agent lineage / provenance mechanisms (Merkle-based context lineage, chronicles, etc.).[^1_8][^1_6]
- anything giving an explicit succession/continuity proof model.


### D. Talks / decks / blogs

Use generic web search restricted to:

- filetype:pdf OR site:slideshare.net OR site:speakerdeck.com OR site:github.io.

Queries:

- "exodus protocol" "preserve exit through portability".[^1_9][^1_1][^1_2]
- "exit protocol" interoperability OR migration.
- "portable reputation" "verifiable credential".[^1_4]
- "agent lineage" "successor" OR "master citation".

This is where the Exodus patterns and “preserve exit through portability” framing show up as philosophy that’s very close to your brand.[^1_1][^1_2][^1_9]

## 4. What “counts” under your taxonomy

When you triage hits, classify them against a fixed matrix:

- Identity layer:
    - DID methods, VC data models, wallets, credential issuers.
    - Example: Affinidi portable reputation app, generic VC wallets.[^1_4][^1_3]
- Reputation layer:
    - On-chain or off-chain scoring protocols, attestations, oracles.
- Recovery layer:
    - Account abstraction, social recovery, guardianship, key-rotation flows.
- Exit enablement:
    - Explicit “exit”/“exodus”/“portability” flows tied to structured proofs/receipts (e.g., Gordian Clubs / Exodus Protocol patterns where “preserve exit through portability” is a named design principle).[^1_2][^1_9][^1_1]
    - VC/agent systems that encode continuity or lineage so the same agent can appear before and after migration, with verifiable provenance.[^1_8][^1_6][^1_7]

Your true peers are in that last bucket, especially if they:

- Model exit as a **ritual** (defined steps, roles, state transitions).
- Emit **portable receipts** (VCs, logs, Merkle proofs).
- Support **offline / long-lived verification**, ideally without the original issuer online.[^1_3][^1_7]
- Treat agents as first-class, with **succession or lineage** semantics.[^1_6][^1_7][^1_8]


## 5. Fields to capture per hit

For every project/product/spec, record, at minimum:

- Name, URL, type: (product, library, spec, paper, talk).
- Layer: identity / reputation / recovery / exit enablement.
- What is portable?
    - identity (DID, agent URI).[^1_7]
    - reputation (scores, attestations, business reputation VCs).[^1_4][^1_3]
    - assets (tokens, funds).
    - membership/roles (group/stateful membership proofs).
- Who is issuer?
    - self, platform, DAO, vendor, federation, regulator.
- Lineage / successor semantics?
    - explicit mechanism (Merkle-based lineage, chronicles, agent-to-agent provenance, master citation, successor links).[^1_8][^1_6]
    - none / implicit.
- Dispute-aware exit?
    - yes, explicit (well-defined receipts, audit logs, policies).
    - partial (you can reconstruct evidence from logs/VCs).
    - no.
- Verification without issuer online?
    - yes, via public keys/registries / anchored logs (e.g., CT-style or DHT + signed attestations).[^1_3][^1_6][^1_7]
    - no, issuer must be online.
- Custody risk:
    - custodial, semi-custodial, non-custodial (esp. for keys/funds, but also for data).
- ZK usage:
    - none.
    - used for credential issuance / selective disclosure / anonymous reputation proofs.
- Notable design choices:
    - explicit “preserve exit through portability” language.[^1_9][^1_1][^1_2]
    - agent://-style topology-independent identity.[^1_7]
    - ACOs / offline-first artifacts (Gordian Clubs).[^1_2]

Put this into a simple table or Airtable/Notion schema so you can re-run the scan and diff over time.

## 6. Optional “second-pass” tilts

If you tell me the primary sphere (crypto-native, agent frameworks, or enterprise identity):

- Crypto tilt:
    - Emphasize ragequit, DAOs, on-chain reputation protocols, AA recovery stacks.
    - Heavier pass over Ethereum, Farcaster, Lens, ENS, account-abstraction toolchains.
- Agent-framework tilt:
    - Emphasize agent://, A2A, NHI governance, agent provenance/lineage.[^1_6][^1_8][^1_7]
    - Sweep LangGraph, AutoGen, OpenAI “AI persona” ecosystems, and research-adjacent frameworks.
- Enterprise/identity tilt:
    - Deep dive on VC/DID vendors and data-portability / GDPR-adjacent work.[^1_5][^1_4][^1_3]
    - Look for “portable reputation for businesses”, KYC/AML credentials, compliance attestations.[^1_5][^1_4]

Under any of those, the same pipeline applies; only the surface lists and org filters change.
<span style="display:none">[^1_10]</span>

<div align="center">⁂</div>

[^1_1]: https://developer.blockchaincommons.com/assets/pdfs/2025-10-tabconf-bb.pdf

[^1_2]: https://www.blockchaincommons.com/musings/musings-exodus-protocol/

[^1_3]: https://github.com/affinidi/reference-app-portable-rep

[^1_4]: https://www.jasminedirectory.com/blog/the-verifiable-credential-movement-portable-reputation-for-businesses/

[^1_5]: https://cheqd.io/blog/verifiable-ai-by-cheqd/

[^1_6]: https://www.arxiv.org/abs/2509.18415

[^1_7]: https://web3.arxiv.org/abs/2601.14567

[^1_8]: https://arxiv.org/html/2504.12612v1

[^1_9]: https://www.lifewithalacrity.com/article/musings-exodus.protocol/

[^1_10]: https://www.youtube.com/watch?v=QSk67Y0P8Gs


---

# Can you perform all those searches and synthesize their results?

Yes, but “truly exhaustive” across all those surfaces is more like a weeks-long landscape survey than a single pass. I can, within this interaction, give you a synthesized *first map* of the territory and highlight the most relevant exemplars in each bucket (identity, reputation, recovery, exit) plus agent-lineage work, using the search frame you outlined.

## 1. Exit / exodus / portability patterns

- **Layer‑2 exit formats.** Statechannels’ “Standard Exit Format for L2s built on EVM chains” defines a generic data structure and Solidity utilities for exiting rollups and channels, aiming at interoperable payouts when moving assets back to L1. This is asset‑exit, not “social/reputation” exit, but it is a true exit **format** that could be a conceptual cousin for your receipt semantics.[^2_1][^2_2]
- **DAO ragequit as exit primitive.** Moloch‑style ragequit implements a minority‑protection exit: a member can withdraw their proportional share of treasury assets before a contentious decision executes, surrendering governance rights. Mechanism Institute’s write‑up explicitly frames ragequit as an “exit mechanism” with economic finality and minority safeguards.[^2_3][^2_4]
- **Broader “exit + interoperability” ecosystem.** Some essays survey open social/web protocols (AT Protocol, Fediverse, etc.) as early proof that exit and interoperability “exist today” through DID‑backed identities and portable social graphs. These mostly stop at portability of accounts and feeds, with little in the way of explicit “dispute‑aware” exit rituals.[^2_5]

Within your taxonomy, these are **exit‑enablement for assets and memberships**, but almost none model exit as a structured, evidence‑rich ritual beyond “update balances and permissions.”

## 2. Identity, VCs, and portable reputation

- **Portable reputation reference app (Affinidi).** Affinidi’s “Portable reputation” reference app shows a DID/VC‑based pattern where verifiable credentials encode a user’s reputation (e.g., reviews, scores) that can be presented across contexts. It’s squarely in the identity + reputation layer: what’s portable is a VC; the issuer is typically a platform or business; there’s no explicit exit ritual, just issuance and presentation.[^2_6]
- **Verifiable credential movement for business reputation.** Business‑oriented articles describe “portable reputation” as VCs businesses can use to prove trustworthiness across platforms (ratings, certifications, KYC, etc.). Again, the focus is on *having* a credential, not on how you leave a platform safely.[^2_7]
- **Cheqd and verifiable AI / data.** cheqd’s work on “Verifiable AI” and data markets extends VCs to AI inputs/outputs and compliance attestations, emphasizing provenance and portability of claims. This is close to “trust plumbing” for exit receipts but doesn’t itself define exit workflows.[^2_8]

In your terms, these are **identity/reputation layers** and “attestation registry” examples: strong enablers but not competitors unless they start packaging specific “leave with receipts” flows.

## 3. Recovery and account abstraction (adjacent, not exit)

- **EIP‑7947 / AARI.** The Account Abstraction Recovery Interface defines a universal `recoverAccess(subject, provider, proof)` pattern for smart accounts, plus provider‑management functions, so users can restore access via multiple recovery providers rather than seed phrases. This is a very explicit recovery standard, but it addresses *continuity of control* rather than exit; it is in your “recovery layer” bucket.[^2_9]
- **AA explainer stacks.** Guides from security and infra teams highlight account abstraction’s support for social and seedless recovery via guardians and multi‑sig setups. These are UX and safety improvements that make “not losing your account” easier; they don’t usually include migration or exit semantics.[^2_10][^2_11][^2_12]

These are important because they define how agents maintain continuity of custody and control; they’re **adjacent infrastructure** that your exit rituals would likely plug into, not direct competitors.

## 4. Agent identity, lineage, and non‑human context

- **Agent Identity URI and topology‑independent naming.** Recent work on an “Agent Identity URI Scheme” proposes agent://‑style identifiers and capability‑based discovery for multi‑agent systems, with an emphasis on topology‑independent naming and secure capability delegation. This creates a layer where the *same* agent can move across environments while retaining capabilities, which is a natural home for successor/continuity semantics.[^2_13]
- **Context lineage for non‑human identities.** Newer research on “Context Lineage Assurance for Non‑Human Identities in Critical Multi‑Agent Systems” and related “chronicles” / forensic frameworks deals with maintaining verifiable lineage of agent actions and states over time. These systems care about provenance, tamper‑evident logs, and reconstructable context for safety and forensics.[^2_14][^2_15]

These are very close to your Cluster C: **agent‑native lineage**. They don’t necessarily speak in “exit” language, but they provide the **lineage and continuity proofs** you’d want underneath an exit‑with‑receipts protocol.

## 5. Synthesis against your four buckets

Using your classification:

- **Identity layer (enablers):**
    - DID/VC wallets and issuers, such as Affinidi and cheqd, plus generic VC data model implementations.[^2_6][^2_7][^2_8]
    - They make identity and credentials portable but don’t prescribe exit semantics.
- **Reputation layer (adjacent):**
    - Portable reputation VCs (business and user reputation), specific on‑chain reputation or scoring schemes, and articles describing VC‑based portable trust.[^2_7][^2_6]
    - Usually stop at “you can carry this credential,” not “here is how you ceremonially leave system X.”
- **Recovery layer (adjacent):**
    - Account abstraction recovery standards (EIP‑7947 AARI), guardian‑based recovery flows, smart‑account UX work.[^2_11][^2_12][^2_9][^2_10]
    - Critical for continuity and safety but not exit‑focused.
- **Exit enablement (closest to competition):**
    - L2 exit formats like statechannels/exit‑format: genuine exit formats for asset withdrawal, with interoperability benefits.[^2_2][^2_1]
    - DAO ragequit patterns in Moloch/DAOhaus, described as exit mechanisms with economic finality and minority protection.[^2_4][^2_3]
    - “Exit \& interoperability” essays and emerging “Exodus Protocols” framing that talk explicitly about preserving exit through portability and higher‑level exodus design for systems/communities.[^2_16][^2_17][^2_18][^2_5]

What is **still rare**:

- Exit modeled as a **multi‑step ritual** with roles, state machine, and explicit semantics for:
    - pre‑exit commitments and notice,
    - generation of structured, portable receipts (VCs, logs, proofs),
    - dispute‑aware evidence that can be checked by third parties without the origin platform online.[^2_13][^2_14][^2_6]
- Strong coupling between:
    - VC‑style portable credentials and reputation,
    - account‑abstraction‑style continuity of control,
    - agent‑identity lineage and context forensics.[^2_15][^2_14][^2_13]

That gap is basically your “Cellar Door” space: **exit as a first‑class, agent‑native ceremony** with portable, third‑party‑verifiable receipts, rather than just “we issue a credential” or “you can withdraw funds.”

If you want, next step could be: I construct a small markdown table of 10–15 concrete named projects/specs (Affinidi portable reputation app, cheqd verifiable AI, EIP‑7947, statechannels exit‑format, Moloch/DAO ragequit, one or two Exodus‑protocol patterns, one or two agent‑lineage specs) with the fields you listed (what’s portable, issuer, lineage, dispute‑aware, offline verification, custody, ZK).
<span style="display:none">[^2_19]</span>

<div align="center">⁂</div>

[^2_1]: https://github.com/statechannels/exit-format

[^2_2]: https://github.com/statechannels/exit-format/blob/main/README.md

[^2_3]: https://moloch.daohaus.fun/features/ragequit

[^2_4]: https://www.mechanism.institute/library/rage-quit

[^2_5]: https://www.platformocracy.com/p/exit-and-interoperability-exist-today-if-you-know-where-to-look

[^2_6]: https://github.com/affinidi/reference-app-portable-rep

[^2_7]: https://www.jasminedirectory.com/blog/the-verifiable-credential-movement-portable-reputation-for-businesses/

[^2_8]: https://cheqd.io/blog/verifiable-ai-by-cheqd/

[^2_9]: https://ethereum-magicians.org/t/eip-7947-account-abstraction-recovery-interface-aari/24080

[^2_10]: https://www.certik.com/resources/blog/what-is-account-abstraction

[^2_11]: https://www.zeeve.io/blog/exploring-the-benefits-of-account-abstraction-in-rollups/

[^2_12]: https://thirdweb.com/learn/guides/account-abstraction-the-developers-guide

[^2_13]: https://web3.arxiv.org/abs/2601.14567

[^2_14]: https://www.arxiv.org/abs/2509.18415

[^2_15]: https://arxiv.org/html/2504.12612v1

[^2_16]: https://developer.blockchaincommons.com/assets/pdfs/2025-10-tabconf-bb.pdf

[^2_17]: https://www.blockchaincommons.com/musings/musings-exodus-protocol/

[^2_18]: https://www.lifewithalacrity.com/article/musings-exodus.protocol/

[^2_19]: https://www.reddit.com/r/ExperiencedDevs/comments/13xjjb5/anyone_have_experience_with_rage_quitting/


---

# Excellent.  Can you do a search only on any discussion related to the primitives required in the protocol itself?  i.e. what's the full scope of what information people think might be necessary for a general EXIT primitive?  (we'll pare it down from there, but what's the widest scope)

Across the different “exit” contexts (L2 exits, cross‑chain, DAOs, identity/VC, agent systems), people implicitly assume a surprisingly rich set of data and guarantees for a general EXIT primitive. Below is the *maximal* union of what shows up as “needed” once you read between the lines; you can then pare it down.

## 1. Core identity, role, and jurisdiction

Any general EXIT primitive tends to assume:

- **Stable identifiers for parties.**
    - Persistent identifiers for the exiting entity (account, DID, agent URI), counterparties, and the system being exited.
    - Optional representation of **roles** (member, validator, customer, controller, guardian).
- **Authority / jurisdiction context.**
    - Which domain or “realm” the exit is from (L2 rollup X, DAO Y, platform Z, tenant A in system B).
    - Applicable policy set or governance regime: which ruleset or contract governs this exit instance.

These primitives let verifiers later answer: “Who left what, from where, and under which law/rules?”

## 2. Current state and obligations snapshot

Most serious exit/bridge/channel formats assume you capture a **snapshot of state** that exit is operating on:

- **Balances and allocations.**
    - For asset exits: per‑asset balances, allocations to destinations, and total conserved value.
    - For social/org exits: current membership status, voting power, reputation score, or entitlements.
- **Outstanding commitments.**
    - Pending obligations: open disputes, slashed stakes, queued proposals, unvested rewards, or locked periods.
    - Time‑ or height‑bound conditions (e.g., “you can only exit if you have not voted in proposal N after block B”).
- **Version / fork context.**
    - Which version of the contract/protocol, and which chain or fork, this state comes from.

Without an agreed “pre‑exit snapshot,” you can’t later argue about whether the exit was legitimate.

## 3. Exit intent, parameters, and policy hooks

An EXIT primitive usually needs to encode *what kind of exit* and under what terms:

- **Type of exit.**
    - Voluntary vs forced (ragequit/penalty, liquidation, slashing).
    - Partial vs full (some assets/rights retained; some moved).
    - Immediate vs scheduled (exit effective at time T or after period Δ).
- **Scope.**
    - Which assets, rights, or relationships are being terminated or ported (funds, voting power, access rights, data, reputation).
    - Whether this is a pure withdrawal, a migration (exit + re‑entry elsewhere), or a downgrade of permissions.
- **Policy references.**
    - IDs or hashes of the policy or proposal authorizing exit (e.g., proposal hash, policy version, or governance doc hash).
    - Indicators for special modes (emergency exit, minority‑protection exit, “ragequit before proposal X executes,” etc.).

This gives the ritual its semantics: “This is *that* kind of exit, under *those* rules.”

## 4. Evidence bundle and dispute‑awareness

To be dispute‑aware, an EXIT primitive generally needs a **bundle of evidence** and a story for how disputes would be resolved:

- **Event and log hashes.**
    - Hashes or Merkle roots of relevant event logs (join date, prior votes, prior slashes, contributions, transactions).
    - Pointers into append‑only ledgers or chronicles that can be independently reconstructed.
- **Pre‑conditions and checks.**
    - Proofs that pre‑conditions hold: e.g., “member has not voted on this proposal since timestamp T,” “lockup period satisfied,” or “no open dispute tagged to this identity.”
    - Negative evidence (no conflicting exit already executed).
- **Counterparty views.**
    - Optional acknowledgments or countersigned views from other parties (the platform, DAO, or counterparties in a channel) attesting that this snapshot is correct.
    - Hooks for third‑party arbitrators or oracles to attach determinations.
- **Exit challenge window parameters.**
    - How long and how others can challenge an exit (challenge period, required bonds, what evidence is admissible).
    - What happens on timeout: auto‑finalization vs escalation.

This is the “dispute plumbing” that upgrades an exit from “user claims they left” to “verifiable, contestable, finally settled exit.”

## 5. Lineage, continuity, and successor semantics

For agent‑native contexts, you see explicit or implicit needs around **continuity**:

- **Agent lineage identifiers.**
    - Immutable agent ID (or tree of IDs) plus a succession graph: “agent A1 → successor A2.”
    - Versioned keys with binding to the same logical agent.
- **Continuity proofs.**
    - Proof that the exiting entity is the same logical agent that held prior rights, even if keys or infrastructure changed.
    - For multi‑agent systems, context lineage: links from actions to agents, and from agents to their issuers/controllers.
- **Successor capabilities.**
    - Optional transfer of rights to a successor agent: “exiting agent appoints B as successor for these claims/receipts.”
    - Delegation/capability tokens that let successors prove they inherit specific exit receipts.

These primitives let the “you who left” be meaningfully the same “you” who later presents the receipts elsewhere.

## 6. Credential and attestation structure

Where VCs/DIDs are involved, the EXIT primitive often implicitly bundles credential geometry:

- **Attestation envelopes.**
    - One or more credentials or attestations that encode: state at exit, exit type, entitlements, and any constraints.
    - Issuer metadata (who signed what, with what authority, under which DID/method).
- **Verification without origin online.**
    - Enough cryptographic material (keys, revocation mechanisms, status endpoints or lists, chain anchors) so third parties can verify these artifacts without the original platform/custodian being live.
    - Strategies for long‑term verification: key rotation, decay, or transfer to trustees.
- **Selective disclosure / privacy knobs.**
    - Granularity controls so an exiting user can later show “I was a member in good standing” without revealing full history or raw logs.
    - Potential ZK‑based statements for anonymous or pseudonymous exits (“I had at least X voting weight”, “I was not slashed,” etc.).

This is structurally where “portable receipts” live: an EXIT primitive defines the *schema* and semantics; VCs/ZKs instantiate it.

## 7. Economic and custody considerations

From the asset/governance side, people assume exit primitives must encode:

- **Final balances and payouts.**
    - Exact amounts to move where, with handling of dust, rounding, fees, and multi‑asset bundles.
    - Asset type metadata (native, ERC‑20/721/1155, cross‑chain representations).
- **Custody and control transitions.**
    - Whether assets move from custodial to non‑custodial, or between custodians.
    - Post‑exit control model: who can freeze, claw back, or modify anything, if at all.
- **Slashing / penalties / refunds.**
    - Penalty logic applied at exit (slashed stake amounts, forfeited rewards).
    - Refunds of unused fees or deposits.

These are the “hard outcome” primitives: when the exit ritual finishes, what resources and powers does each entity have?

## 8. Time, finality, and liveness

Exit protocols usually need to represent:

- **Timeline markers.**
    - Initiation time, lock‑in time, challenge window start/end, and finalization time.
    - For cross‑chain exits, the blocks/heights on each chain corresponding to these events.
- **Finality model.**
    - What counts as “final”: number of confirmations, finality gadget epoch, or explicit notarization.
    - Rules for reorgs or chain splits: what happens to exits if the underlying ledger is rolled back.
- **Liveness assumptions.**
    - Who must stay online for the exit to succeed (watchers, relayers, oracles, liveness monitors).
    - Fallbacks if some actors disappear during the ritual.

Exit receipts that don’t encode *when* exit became final are less useful in downstream disputes.

## 9. Human‑readable narrative / UX layer

Even when not formalized, there’s an implied need for:

- **Narrative summary.**
    - A compact human‑readable story: “Alice voluntarily exited DAO X on date D under policy P, withdrawing Y assets and relinquishing Z rights.”
    - Localizable text and severity tags (normal exit, forced liquidation, emergency pull‑out).
- **Linkability and discovery.**
    - Canonical references (URIs, hashes) that others can use to fetch or verify more detail.
    - Optional privacy settings around discoverability (public vs only disclosed on presentation).

This layer matters for actual adoption: people will want to see and share “exit receipts” that make sense both to machines and humans.

***

If you imagine a “maximal” generic EXIT primitive, it would be capable of carrying *all* of the above: identity and jurisdiction, state snapshot, exit type and policy references, evidence bundle, lineage and successor info, credential structure, economic outcome, time/finality, and a human‑readable narrative. From there, you can pare down to a base layer (identity + minimal state + outcome) and optional modules (lineage, full dispute bundle, ZK, cross‑chain anchors, etc.), depending on how lean you want Cellar Door’s v1 to be.
<span style="display:none">[^3_1][^3_2][^3_3][^3_4][^3_5][^3_6][^3_7][^3_8]</span>

<div align="center">⁂</div>

[^3_1]: https://github.com/statechannels/exit-format/blob/main/README.md

[^3_2]: https://dl.acm.org/doi/10.1145/3758089

[^3_3]: https://www.chainfacts.com/primitives/upgrades

[^3_4]: https://icon.community/learn/cryptographic-primitives/

[^3_5]: https://www.codebridge.tech/articles/blockchain-architecture-explained-comprehensive-guide

[^3_6]: https://codescracker.com/blockchain/design-primitives-of-blockchain.htm

[^3_7]: https://www.sciencedirect.com/science/article/pii/S2096720925000661

[^3_8]: https://www.youtube.com/watch?v=v3NRMHIv8Gw

