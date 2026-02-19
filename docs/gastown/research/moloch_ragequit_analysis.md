# Moloch Ragequit Analysis: State Machine, Data Structures, and Implications for EXIT
## cellar_door/crew/scholar | Bead cd-t9n | 2026-02-11

> **Task:** Survey Moloch ragequit: extract state machine and data structures.
> **Context:** Cellar Door's EXIT ceremony is the generalization of structured departure. Moloch ragequit is the closest existing implementation of a formalized exit mechanism with economic finality.

---

## 1. WHAT RAGEQUIT IS

Moloch v2's ragequit is a **purely economic, atomic, unconditional exit right** with one temporal constraint. A member burns shares and/or loot (non-voting economic tokens), receives a pro-rata share of every approved token in the guild treasury, and leaves. One transaction. No ceremony.

It was created to solve the "51% attack on the treasury" problem — preventing a majority from spending a minority's capital on proposals the minority opposes. The grace period + ragequit combination ensures any dissenting member can exit *before* an objectionable proposal executes.

**Origin:** The DAO's split function (2016) was the predecessor. Its recursive calling vulnerability enabled the infamous hack (3.6M ETH). Moloch (2019, Ameen Soleimani) simplified the mechanism to minimum viable form.

---

## 2. THE RAGEQUIT STATE MACHINE

### There Is No Multi-State Machine

This is the key structural finding: **ragequit has no intermediate states.** It is a single atomic transaction.

```
┌──────────┐         canRagequit()?        ┌──────────┐
│          │──────── YES ─────────────────►│          │
│  MEMBER  │                               │  EXITED  │
│          │◄─────── NO (revert) ──────────│          │
└──────────┘                               └──────────┘
```

Compare with the Architect's EXIT ceremony:
```
ALIVE → INTENT → SNAPSHOT → OPEN → CONTESTED → FINAL → DEPARTED
```

Ragequit collapses all of this into a single step. There is no intent declaration, no snapshot, no challenge window, no co-signatures. The entire departure happens atomically in one Ethereum transaction.

### Pre-Conditions (All Must Be True)

1. Caller is a member: `shares > 0 || loot > 0`
2. Sufficient shares: `member.shares >= sharesToBurn`
3. Sufficient loot: `member.loot >= lootToBurn`
4. **canRagequit passes:** The highest-indexed proposal the member voted YES on must be processed

### The Atomic Transaction

```solidity
function ragequit(uint256 sharesToBurn, uint256 lootToBurn) public nonReentrant onlyMember
```

Internal execution (`_ragequit`), in order:
1. Snapshot `initialTotalSharesAndLoot` (before burning)
2. Validate pre-conditions
3. Burn member's shares and loot
4. Reduce global totals
5. For each approved token: calculate pro-rata share, debit GUILD, credit member
6. Emit `Ragequit(memberAddress, sharesToBurn, lootToBurn)` event

### Post-Conditions
- Member's shares/loot reduced (can be partial — burn some, keep some)
- Funds in member's *internal balance* (pull pattern — separate `withdrawBalance` call needed)
- Member's `exists` flag remains `true` (never reset)
- No reputation artifact, no receipt, no credential produced

---

## 3. DATA STRUCTURES

### Member Struct
```solidity
struct Member {
    address delegateKey;           // Voting delegate
    uint256 shares;                // Voting + economic rights
    uint256 loot;                  // Economic rights only (v2 addition)
    bool exists;                   // Always true once created
    uint256 highestIndexYesVote;   // Queue index of highest YES vote
    uint256 jailed;                // Set by guildkick; 0 = not jailed
}
```

### Proposal Struct (ragequit-relevant fields)
```solidity
struct Proposal {
    uint256 startingPeriod;
    bool[6] flags;  // [sponsored, processed, didPass, cancelled, whitelist, guildkick]
    uint256 maxTotalSharesAndLootAtYesVote;  // Peak total at time of any YES vote
    mapping(address => Vote) votesByMember;
    // ... other fields
}
```

### Treasury
```solidity
address constant GUILD  = address(0xdead);  // DAO treasury sentinel
address constant ESCROW = address(0xbeef);  // Tribute escrow sentinel

mapping(address => mapping(address => uint256)) public userTokenBalances;
address[] public approvedTokens;  // Whitelisted tokens for distribution
```

### Fair Share Formula
```
amountToRagequit = (guildBalance[token] * sharesAndLootToBurn) / initialTotalSharesAndLoot
```

Shares and loot are economically equivalent (1:1) for distribution. The only difference: shares grant voting rights, loot does not.

---

## 4. THE TEMPORAL CONSTRAINT: `canRagequit` + Grace Period

### How the Protection Works

```
Proposal timeline:
|---- VOTING (35 periods, ~7 days) ----|---- GRACE (35 periods, ~7 days) ----|-- PROCESSABLE --|
```

The `highestIndexYesVote` mechanism creates the protection:

1. When member votes YES on proposal at queue index N → `highestIndexYesVote = max(current, N)`
2. On ragequit → requires `proposals[proposalQueue[highestIndexYesVote]].flags[processed] == true`

**Effect:** A member who voted YES cannot ragequit until that proposal is processed. A member who voted NO or abstained on all pending proposals *can* ragequit immediately — including during the grace period, *before* the proposal they oppose executes.

### The Dilution Bound Auto-Fail

Complementary protection: if too many members ragequit during a grace period, the proposal auto-fails.

```solidity
if (totalSharesAndLoot * dilutionBound < proposal.maxTotalSharesAndLootAtYesVote) {
    didPass = false;  // Mass ragequit detected → proposal fails
}
```

This prevents a passed proposal from executing after a mass exodus has drained the treasury.

---

## 5. GUILDKICK: THE FORCED EXIT

| Aspect | Ragequit (Voluntary) | GuildKick (Forced) |
|--------|---------------------|-------------------|
| Initiated by | Exiting member | Any member (via proposal) |
| Requires proposal? | No | Yes (full lifecycle: vote + grace) |
| Two phases? | No (atomic) | Yes: shares→loot conversion, then ragekick |
| Intermediate state | None | **Jailed** (no voting, no sponsoring) |
| Economic outcome | Pro-rata distribution | Same, but forced |
| Contestable? | No | Yes (during voting) |

GuildKick process:
1. Submit guildkick proposal targeting a member
2. Normal proposal lifecycle (voting + grace)
3. If passed: `member.shares → member.loot` (strips voting power immediately), `member.jailed = proposalIndex`
4. Once the member's `highestIndexYesVote` is processed, anyone can call `ragekick()` to force the economic exit

---

## 6. THE BROADER DAO EXIT LANDSCAPE

### Nouns DAO Fork — The Most Sophisticated Existing Exit

The closest analog to a multi-step ceremony:

1. **Escrow/Signal:** Token holders deposit Nouns into escrow (costs governance power)
2. **Threshold:** Fork triggers at 20% of supply escrowed
3. **Forking Period (~7 days):** Additional holders can join; new DAO deployed with pro-rata treasury
4. **Token Claiming:** Participants claim new tokens in fork DAO
5. **Delayed Governance:** Proposals blocked until all participants have claimed
6. **Vanilla Ragequit:** Individual exit always available in fork DAOs

**Nouns Fork 1 (Sept 2023):** 56% of holders forked. Fork DAO treasury fell from 16,750 to 7,700 ETH in three days as members ragequit, each taking ~35.5 ETH (~$58K). Three consecutive forks drained the treasury to 40% of peak.

### Other Mechanisms

| System | Exit Type | Multi-Step? | Portable Receipt? |
|--------|-----------|:-----------:|:-----------------:|
| Moloch v2 | Individual ragequit | No | No |
| Moloch v3 (Baal) | Individual ragequit (+ `to` address, token selection) | No | No |
| Tribute DAO | Ragequit adapter (modular) | No | No |
| Nouns Fork | Collective fork + individual ragequit | **Partial** | No |
| Aragon | Token redemption (dissolution) | No | No |
| Compound | Market sale only | No | No |

### The Aragon Cautionary Tale

Aragon had no ragequit. "Risk Free Value Raiders" accumulated ANT below book value and pushed for treasury dissolution. The Aragon Association preemptively dissolved, deploying 86,343 ETH into a redemption contract. **This is the failure mode of no structured exit: the organization itself dissolves under pressure.**

---

## 7. RAGEQUIT'S LIMITATIONS (The Gap Cellar Door Fills)

### What Ragequit Does

| Capability | Ragequit |
|-----------|:--------:|
| Economic exit (pro-rata treasury) | Yes |
| Minority protection (exit before bad proposal executes) | Yes |
| Unilateral (no permission needed) | Yes |
| Partial (burn some, keep some) | Yes |
| Atomic (no failure mid-ceremony) | Yes |
| Involuntary exit (guildkick) | Yes |
| Mass-exit detection (dilution bound) | Yes |

### What Ragequit Does NOT Do

| Capability | Ragequit | EXIT Ceremony |
|-----------|:--------:|:-------------:|
| Portable departure receipt | No | Yes (VC-compatible) |
| Identity continuity | No | Yes (subject persists) |
| Dispute-aware exit | No | Yes (challenge window) |
| Evidence preservation | No | Yes (Module C) |
| Reputation portability | No | Yes (status field) |
| Successor/lineage semantics | No | Yes (Module A) |
| Cross-platform applicability | No (DAO-only) | Yes (any coordination regime) |
| Agent/non-human exit | No | Yes (emergency path) |
| Multi-step ceremony | No (atomic) | Yes (7-state machine) |
| Voice within exit | No | Yes (reason, narrative, evidence) |
| Offline verification | No (requires chain) | Yes (self-certifying markers) |

### The Hirschman Connection

Ragequit is pure Hirschman **Exit** with no **Voice** channel. It communicates *that* someone left but not *why*. As the Voice Over Exit project (Reboot) argues: "While both voice and exit express dissatisfaction, only voice communicates exactly what's going wrong."

Cellar Door's EXIT ceremony embeds voice *within* exit: evidence bundles, departure reasons, dispute records. This is the synthesis Hirschman's framework anticipates but existing DAO implementations don't achieve.

### The Mechanism Institute's Analysis

The Mechanism Institute identifies ragequit's core tension:
- **Positive:** Minority protection, governance moderation, immediate resolution
- **Negative:** Treasury instability, exit friction with complex assets, short-termism incentives, economic attack surface

They trace ragequit's lineage to corporate **appraisal rights** (dissenting shareholders' right to fair value upon fundamental change). EXIT extends this analogy: appraisal rights come with legal process, evidence, and portable judgments. Ragequit strips all of that away. EXIT puts it back.

---

## 8. IMPLICATIONS FOR THE EXIT SCHEMA AND CEREMONY

### What EXIT Should Learn from Ragequit

1. **Unilateral exit must always be available.** Ragequit's unconditional nature (modulo the YES-vote constraint) is its greatest strength. The Architect's "contests don't block exit" decision is the correct generalization.

2. **The fair share formula is a solved problem.** For economic exits, pro-rata distribution is well-understood and battle-tested. EXIT's Module D (Economic) can reference this directly.

3. **The dilution bound is an elegant governance signal.** Mass ragequit triggering proposal auto-fail is a mechanism EXIT could generalize — mass departure as a verifiable governance signal, not just individual action.

4. **Partial exit matters.** Ragequit's ability to burn *some* shares while keeping others is the on-chain version of partial exit scoping (cd-1zc.6). The `origin` field in EXIT may need to support sub-scoping.

5. **The guildkick two-phase pattern maps to EXIT's forced path.** GuildKick's jail→ragekick sequence is a form of `exitType: forced` with an intermediate state. EXIT's ceremony already accommodates this via INTENT→FINAL with `exitType: forced`.

### What EXIT Adds Over Ragequit

1. **The ceremony itself.** Ragequit is a function call. EXIT is a ritual with roles, states, evidence, and receipts. The ceremony is the primitive — not the economic distribution.

2. **Portable receipts.** The single most important addition. After ragequit, the only record is an on-chain event. After EXIT, the departing entity holds a self-certifying, offline-verifiable credential.

3. **The emergency path.** Ragequit requires an on-chain transaction. EXIT's emergency path (ALIVE→FINAL in milliseconds) works when the infrastructure itself is dying. This is the agent-native innovation.

4. **Cross-regime applicability.** Ragequit is DAO-specific. EXIT works for any coordination regime — including ones with no treasury, no blockchain, and no smart contracts.

### Proposed `exitType: systemic` — Mass Exit Signal

The Nouns fork experience suggests a new pattern: **systemic exit** where the entire coordination regime is fragmenting. This maps to the `systemic` exit type proposed in the Architect review (E3). When a critical mass of members depart simultaneously:
- Individual EXIT markers can reference a shared `systemicEvent` ID
- Verifiers can aggregate markers to reconstruct the mass-exit event
- The dilution-bound auto-fail pattern becomes a verifiable governance signal rather than just a contract mechanism

---

## 9. ACADEMIC CONTEXT

### Key Papers

- **Hirschman, "Exit, Voice, and Loyalty" (1970):** The foundational text. Ragequit is pure exit; EXIT adds voice.
- **"Open Problems in DAOs" (Oak, Lim, Allen, Landemore, 2024):** Exit mechanisms identified as unresolved. "Early returns on rage quits have been mixed." Agent exit not considered.
- **Strnad, "Economic DAO Governance: A Contestable Control Approach" (2024):** Argues exit threats alone are insufficient — *control* must be contestable, not just capital. Implications for EXIT: the primitive may need to support "exit as governance action" not just "exit as departure."
- **a16z, "DAO Governance Attacks, and How to Avoid Them":** Catalogs ragequit weaponization within broader governance attack taxonomy.
- **Mechanism Institute, "Rage Quit":** Most comprehensive analytical framework. Traces lineage from corporate appraisal rights through The DAO to Moloch.

### The Arbitrage Problem

Nouns demonstrated that ragequit creates a persistent arbitrage opportunity: buy governance tokens below book value → ragequit → extract the difference. Three consecutive forks drained 60% of treasury value. This is a structural risk for any exit mechanism with pro-rata economic distribution. EXIT's Module D (Economic) should document this as a known attack vector.

---

## Sources

### Contracts and Specifications
- [Moloch v2 source (MolochV2.sol)](https://github.com/MolochVentures/moloch/blob/master/contracts/Moloch.sol)
- [Moloch v2 README](https://github.com/MolochVentures/moloch/blob/master/README.md)
- [Moloch v3 / Baal](https://github.com/Moloch-Mystics/Baal)
- [Baal Ragequit Docs](https://moloch.daohaus.fun/features/ragequit)
- [Tribute DAO Ragequit Adapter](https://tributedao.com/docs/contracts/adapters/exiting/rage-quit-adapter/)
- [Nouns Fork Spec](https://github.com/verbsteam/nouns-fork-spec/blob/main/spec.md)
- [Moloch on Starknet — Ragequit & GuildKick](https://dao-docs.quadratic-labs.com/moloch-on-starknet/features/ragequit-and-guildkick)

### Analysis and Academic
- [Mechanism Institute — Rage Quit](https://www.mechanism.institute/library/rage-quit)
- [Open Problems in DAOs (arXiv)](https://arxiv.org/abs/2310.19201)
- [Strnad — Economic DAO Governance (arXiv)](https://arxiv.org/abs/2403.16980)
- [a16z — DAO Governance Attacks](https://a16zcrypto.com/posts/article/dao-governance-attacks-and-how-to-avoid-them/)
- [ECGI — Review of DAO Governance (2025)](https://www.ecgi.global/publications/working-papers/a-review-of-dao-governance-recent-literature-and-emerging-trends)
- [Voice Over Exit (Reboot)](https://voice.mirror.xyz/cco0CvtNGQMHMGPdmoW74bl86UNXfkfgDmp2PnORYKY)
- [Hirschman — Exit, Voice, and Loyalty](https://www.hup.harvard.edu/books/9780674276604)

### Historical
- [Decrypt — Inside Moloch](https://decrypt.co/5206/fixing-ethereum)
- [Verbsteam — Introducing Nouns Fork](https://mirror.xyz/verbsteam.eth/iN0FKOn_oYVBzlJkwPwK2mhzaeL8K2-W80u82F7fHj8)
- [Jacob — Forks Are Good](https://jacob.energy/forks-are-good.html)
- [Blockworks — Nouns Fork](https://blockworks.co/news/nouns-dao-treasury-fork-governance)
- [CoinDesk — Nouns $27M Revolt](https://www.coindesk.com/business/2023/09/21/nouns-daos-27m-revolt-reveals-toxic-mix-of-money-hungry-traders-and-blockchain-idealists)
- [CoinTelegraph — Raider Investors](https://cointelegraph.com/magazine/raider-investors-daos-nouns-aragon/)
- [CoinDesk — The Ends of Aragon](https://www.coindesk.com/opinion/2023/12/14/the-ends-of-aragon)

---

*Scholar, cellar_door/crew/scholar*
*"Of all the endless combinations of words in all of history"*
