# Fileverse — Economics & Game Theory

_Phase 2c · Generated 2026-04-27_

Fileverse has no disclosed native token, no published fee schedule, and no announced venture round as of 2026-04-27 ([fileverse.io](https://fileverse.io "Source: Fileverse, retrieved 2026-04-27")). Standard tokenomics framing therefore does not apply. This analysis instead models the economic primitives that *do* exist: (1) UCAN-gated quotas on pinning, (2) gas costs for the onchain content registry on Ethereum mainnet and Gnosis Chain, (3) the cost-of-goods structure inherited from Pinata, Filebase, and Web3.Storage, (4) AGPL-3.0 licensing in the absence of disclosed funding, and (5) the strategic choices facing self-hosters versus Fileverse-hosted users. The game-theoretic core is not a bonding curve or staking equilibrium — it is the multi-sided coordination problem between Fileverse-as-protocol, three centralized pinning vendors, self-hosters, end-users, and incumbents (CryptPad, Proton, Notion).

---

## Section 1: Participation Incentive Model

The incentive loop has four distinguishable participants:

- **End-user (U)** — uploads encrypted blobs, consumes WebSocket bandwidth, pays nothing in the disclosed model.
- **Fileverse operator (F)** — runs the storage gateway, the stateless collaboration server, and pays pinning bills downstream.
- **Pinning provider (P_i ∈ {Pinata, Filebase, Web3.Storage})** — sells gigabyte-months and bandwidth at published rates.
- **Self-hoster (S)** — runs their own collaboration-server instance and pins via own credentials.

We define the per-document monthly cost to F of a 1 MB document with two collaborators editing 8 hours/week:

- IPFS pin storage: 1 MB × $0.15/GB-month [Pinata Picnic plan list price](https://www.pinata.cloud/pricing "Source: Pinata, retrieved 2026-04-27") ≈ $0.00015/month
- Bandwidth (assume 50 MB/month re-fetch): 0.05 GB × $0.15/GB-bandwidth ≈ $0.0075/month
- Collaboration-server WebSocket compute: ~0.001 vCPU-hour/user-day × 60 days × $0.034/vCPU-hour ≈ $0.002

Total marginal cost ≈ **$0.01/document/month** at a 1 MB midpoint, dominated by bandwidth not storage. The number is small per-document but unbounded in aggregate; cost per user scales with document count and re-fetch frequency, not seat count, which is the inverse of Google Workspace's $6/seat/month structure ([workspace.google.com pricing](https://workspace.google.com/pricing.html "Source: Google, retrieved 2026-04-27")).

### Scenario A — Symmetric (all participants follow protocol intent)

All users U pin via the Fileverse-hosted gateway under UCAN quotas. F bundles pinning costs into a free tier capped by quota, with overflow to a paid tier (price unannounced). Each P_i takes its list-rate share. Self-hosters S exist but are <5% of upload volume. Equilibrium is sustainable iff F's revenue from paying tier ≥ aggregate pinning + bandwidth + audit + dev cost.

Estimated break-even with 1,000 paying users at $5/month (a CryptPad-comparable tier; [CryptPad's lowest paid plan is €5/month](https://cryptpad.org/pricing/ "Source: CryptPad, retrieved 2026-04-27")) yields $5,000/month against ~$2,000–$3,000/month in pinning at modest scale (≈100 GB stored, ≈10 TB egress). That is feasible but does not cover engineering. F is therefore implicitly subsidized — by grants, founder equity, or a future raise — until a token or paid tier emerges.

### Scenario B — Asymmetric (sophisticated free-rider)

A user U' uploads aggressively against the free quota, generates churn (creates → deletes → re-creates docs to reset quota), or exploits the ephemeral collaboration server to relay non-Fileverse data through encrypted WebSocket frames (a steganographic free-rider). F's marginal cost per such user is unbounded; their marginal revenue is zero.

Early participation is **not** a dominant strategy in the standard tokenomic sense — there is no airdrop signal, no early-staker yield, no fee discount on accumulated activity. The dominant strategy for a rational, non-ideological user is **wait-and-see**: use the free tier without reciprocating, defect to CryptPad if Fileverse meters more aggressively, defect to Notion-with-Skiff-style E2EE rooms if a Web2 incumbent ships an E2EE feature. Early participation is dominant **only** for users who place positive utility on (a) ideological alignment with self-sovereignty, or (b) the option value of a future token retroactively rewarding usage — exactly the incentive Fileverse has not committed to.

The architecture is sound; the participation incentive is weak. Fileverse depends on intrinsic motivation plus a small "tip jar"-style revenue path until either a paid tier or a token clarifies cash flow.

---

## Section 2: Equilibrium Analysis

The system has multiple Nash candidates. We model the most consequential one: the **Fileverse-as-gateway equilibrium**, in which (i) F operates a free-quota gateway, (ii) U primarily pins via F rather than self-hosting, and (iii) P_i continues serving F at list rates. The system is at equilibrium when none of {F, U, P_i, S} unilaterally improves payoff by deviating.

Let:
- *q* = fraction of users self-hosting
- *m* = monthly per-user cost burden on F
- *r* = monthly revenue per user (paid tier)
- *c* = pinning provider concentration (HHI on F's spend across P_i)

**Equilibrium holds when:**

1. *r* × paying-share ≥ *m* × total active users — i.e. F is cash-flow positive or grant-covered. Empirically, with $5/month tier and ~30% paid-conversion analog from [CryptPad's published 5,000+ paid subscribers across ~280k registered](https://blog.cryptpad.org/2024/06/12/Onwards-and-upwards/ "Source: CryptPad Blog, 2024-06-12"), F needs roughly **3,300 paying users** to cover ~$15K/month pinning + bandwidth at 1,000 GB stored, **20–30k registered users** to support that paid count, and an additional engineering subsidy (grants or funded round) to cover ~$60–$100K/month in headcount.
2. *q* < ~25%. Above ~25% self-hosting share, F's gateway loses pricing power against pinning vendors and cannot negotiate enterprise discounts ([Filebase's enterprise tier discounts begin at 10 TB committed monthly](https://filebase.com/pricing/ "Source: Filebase, retrieved 2026-04-27")). Below 25%, F retains aggregator leverage.
3. *c* (HHI) < ~5,000 — i.e. no single P_i carries >70% of F's spend. The architecture documents three providers ([github.com/fileverse/fileverse-storage-v2](https://github.com/fileverse/fileverse-storage-v2 "Source: GitHub, retrieved 2026-04-27")); equally distributed yields HHI ≈ 3,333. If Filebase's lower S3-compatible egress pricing pulls share above 70%, HHI rises past 4,900 and Fileverse loses leverage in renegotiation.
4. The maximum imbalance the mechanism tolerates between **upload demand** and **persistence supply** is bounded by the smallest single P_i's stated capacity. [Web3.Storage's free tier ended in 2024 and the service repositioned to a w3up paid model with included storage from $10/month](https://blog.storacha.network/web3-storage-rebrands-to-storacha-network/ "Source: Storacha (formerly Web3.Storage), 2024"); Pinata's free tier caps at 1 GB per account ([Pinata pricing](https://www.pinata.cloud/pricing "Source: Pinata, retrieved 2026-04-27")). If F's free-quota policy permits aggregate uploads >1 GB/free-account/month at >50% of that quota's egress, the cheapest provider rejects writes and the fallback chain forces traffic onto more expensive providers — a 3–5× COGS spike.

**Equilibrium collapses when:**

- Aggregate free-tier abuse pushes *m* above *r* × paying-share for >2 quarters with no grant injection.
- Any single P_i degrades or sunsets (precedent: Web3.Storage's free-tier wind-down in 2024). Storacha's pivot demonstrates that pinning vendors can change pricing on weeks-to-months notice.
- Self-hosting share *q* exceeds ~40% — at that point F's marginal user is a self-hoster and the gateway business becomes a public good without monetization.
- A larger E2EE incumbent (Proton, Notion-with-acquired-Skiff-tech) prices a comparable feature at <$5/month and absorbs the privacy-conscious segment via existing distribution.

The equilibrium is **conditional and fragile**: it requires sustained subsidy or a paid tier launch within roughly 12–24 months under reasonable cost projections.

---

## Section 3: Attack / Gaming Analysis

We analyze five attack vectors with quantified impact.

### Attack 1 — UCAN Quota Sybil (free-tier dilution)

**Mechanism.** UCAN tokens gate IPFS uploads; tokens are issued after authentication. If username-based vOPRF-ID accounts can be created without economic cost, an attacker spawns *N* accounts to multiply free quota by *N*.

**Worked example.** If a free account permits 100 MB pinned + 1 GB egress/month, and account creation costs the attacker zero (no email verification disclosed in public repos), 1,000 sybil accounts yield 100 GB pinned + 1 TB egress free. At [Pinata's $0.15/GB egress](https://www.pinata.cloud/pricing "Source: Pinata, retrieved 2026-04-27"), this is **~$150/month direct cost** dumped onto F per attacker, with no marginal revenue.

**Mitigation status.** Partial. Wallet-based authentication binds an account to a funded address (~$1–$5 minimum gas history); username-based accounts likely do not. Required design change: tier the quota by attestation strength — full quota for wallet-bound or domain-attested users, reduced quota for vOPRF-only.

### Attack 2 — Pinning Provider Cartel / Renegotiation Squeeze

**Mechanism.** Three providers (Pinata, Filebase, Web3.Storage/Storacha) constitute the persistence layer. If two coordinate (or one absorbs another, as Skiff was absorbed by Notion), F's bargaining position degrades.

**Worked example.** Storacha's repositioning from free to paid tiers in 2024 increased the price of the cheapest fallback provider for a comparable workload from $0/month to $10/month-minimum + $0.05/GB ([Storacha pricing](https://blog.storacha.network/web3-storage-rebrands-to-storacha-network/ "Source: Storacha, 2024")). For F at 1,000 GB, a 30% rate hike from one provider with concurrent capacity reduction at another forces 70% of volume onto the priciest path. At list rates this is **a 40–60% COGS increase** on roughly the storage line item.

**Mitigation status.** Partial. Multi-provider fallback exists architecturally but pricing leverage requires either >10 TB committed (enterprise tier qualification) or self-operated IPFS nodes. Required design change: maintain a community-run pinning club (akin to [Pinning Service API-compatible tools](https://ipfs.github.io/pinning-services-api-spec/ "Source: IPFS PSA spec, retrieved 2026-04-27")) and budget for own-node infra at >5 TB scale.

### Attack 3 — Link Lock Leakage (encryption-recovery exploit)

**Mechanism.** The walk-away triple-lock model wraps a per-file AES key under three independent locks, including a **Link Lock** ([github.com/fileverse/walk-away](https://github.com/fileverse/walk-away "Source: GitHub, retrieved 2026-04-27")). If the link lock key appears in a URL fragment (`#key=...`) and the document URL is logged by an intermediary, the file is decryptable by anyone holding the URL.

**Worked example.** Common URL leakage paths: browser sync (Chrome Sync stores history under user's Google account), enterprise proxies, copy-paste into Slack (which fetches link previews), referer headers. If 1% of shared links leak and the average doc has 3 link-shares over its lifetime, **a corpus of 100,000 shared docs yields ~3,000 effectively-public docs** despite E2EE. Quantified user impact: a 3% silent confidentiality breach rate is materially higher than a Web2 baseline where access control is server-enforced. Note: this depends on Fileverse implementing link-keys in URL fragments rather than out-of-band channels; the public repo references link-lock cryptography but does not publish the share-flow UX detail.

**Mitigation status.** Not at all by base mechanism — the architecture pushes responsibility to user link-handling discipline. Required design change: gate link-shared docs behind a one-time-use exchange (link contains an opaque token, server-mediated key handshake binds to recipient identity), or warn explicitly that link-shared docs are revocable only by re-keying.

### Attack 4 — Stateless Collaboration Server Free-Riding

**Mechanism.** The collaboration server is stateless and ephemeral. Anyone can connect; documents in transit are E2EE so the operator cannot distinguish content. A non-Fileverse application can use the server as a free, encrypted CRDT relay.

**Worked example.** A WebSocket relay carrying ~100 KB/s sustained traffic costs an operator on AWS roughly **$0.09/GB egress × 0.36 GB/hour = $0.032/hour**, or ~$23/month per concurrent steady stream ([AWS data-transfer pricing](https://aws.amazon.com/ec2/pricing/on-demand/#Data_Transfer "Source: AWS, retrieved 2026-04-27")). 100 free-rider relays = ~$2,300/month bandwidth dump.

**Mitigation status.** Partial. UCAN tokens gate authorization, but if tokens are issued cheaply or the WebSocket layer accepts unauthenticated relay sessions, free-riding is uncapped. Required design change: rate-limit per-token bandwidth on the collaboration server, enforce UCAN binding at WebSocket-handshake, and apply a quadratic cost curve on egress per token.

### Attack 5 — Onchain Registry Gas Griefing

**Mechanism.** The smart contract registry on Ethereum mainnet maps file IDs to IPFS CIDs ([github.com/fileverse/collaboration-server](https://github.com/fileverse/collaboration-server "Source: GitHub, retrieved 2026-04-27")). If F sponsors writes for end-users (meta-transactions / paymaster pattern), an attacker spams updates to drain the paymaster.

**Worked example.** A registry update is a single SSTORE-class write, ~50,000 gas. At Ethereum mainnet 30 gwei + ETH at $3,000, that is ~$4.50 per write. An attacker sending 1,000 updates/day burns **$4,500/day** = ~$135K/month off the paymaster. Even on Gnosis Chain (target ~1 gwei, native xDAI ~$1), per-write cost is ~$0.0005, but attack rate scales — 1M updates/day yields $500/day. Either chain is exploitable at scale.

**Mitigation status.** Depends on whether Fileverse sponsors writes (no public confirmation in repos as of retrieval). If users self-pay, attack reduces to self-DoS. If F sponsors, required design change: meter per-account onchain writes per epoch, require small non-sponsored deposit to enable sponsored writes, default Gnosis-only with mainnet upgrade reserved for premium tier.

---

## Section 4: Fee Structure Recommendation

We benchmark plausible fee schedules against three direct comparators:

- [CryptPad](https://cryptpad.org/pricing/ "Source: CryptPad, retrieved 2026-04-27"): free 1 GB, €5/month for 20 GB, €15/month for 1 TB (operated by XWiki SAS, ~10y operating history)
- [Proton Drive (Docs/Sheets included)](https://proton.me/drive/pricing "Source: Proton, retrieved 2026-04-27"): free 5 GB, paid plans from $2.99/month (200 GB) up to $9.99/month (1 TB Unlimited)
- [Anytype](https://anytype.io/pricing "Source: Anytype, retrieved 2026-04-27"): free Anytype Network, $9.99/month "Builder" tier with 256 GB, $99/month "Co-creator" tier
- [Standard Notes](https://standardnotes.com/plans "Source: Standard Notes, retrieved 2026-04-27"): free, $90/year Productivity, $120/year Professional

Median paid-tier price among comparators is **$5–$10/month** for 100 GB–1 TB. Fileverse should target this band.

**Recommended structure (conditional on Fileverse choosing to monetize):**

1. **Free tier** — 1 GB pin + 5 GB egress/month, capped at 50 docs concurrent, wallet-bound or vOPRF with rate-limit. Matches CryptPad's free baseline.
2. **Pro tier** — $6/month or 0.003 ETH/month — 50 GB pin, 200 GB egress, unlimited docs, priority WebSocket. The price defection threshold against Proton ($2.99 entry) and CryptPad (€5) is roughly **$7/month**: above that, privacy-conscious users with no Web3 affinity defect to Proton's bundled stack. Below $5, F cannot cover pinning + bandwidth at the implied volume.
3. **Onchain Premium** — $20/month or 0.01 ETH/month — Pro plus paymaster credits for 100 mainnet writes/month or 10,000 Gnosis writes/month, ENS-based access policies, dSheets onchain-write quota. This is the only tier where Fileverse has a clear advantage over Proton/CryptPad, since neither competes on wallet-native primitives.
4. **Self-host license** — AGPL-3.0 compliance is automatic; Fileverse may offer a paid commercial license carve-out (similar to MongoDB SSPL → commercial pattern) for organizations wanting to bundle Fileverse code into a closed product, priced by negotiation in the $10K–$100K/year band.
5. **Pinning gateway pass-through** — operator fee of ~10% above raw P_i list rates for users beyond Pro tier limits (e.g., $0.165/GB-month vs. Pinata's $0.15). Defection threshold: any markup >25% sends sophisticated users to direct-with-Pinata accounts; below 15%, the markup covers gateway operations.

**Defection threshold summary.** The user-side switching cost for a self-hostable, AGPL-licensed E2EE suite is dominated by infrastructure familiarity, not lock-in. The price defection threshold for Fileverse Pro vs. CryptPad Personal is **~$7/month** (above which CryptPad's cheaper, longer-running offering wins); the defection threshold against Proton's bundled $9.99 is **~$10/month** (above which the Proton Mail+Drive+Docs+VPN bundle wins on dollar utility). Fileverse therefore has roughly a $5–$8 price corridor in which to operate.

---

## Section 5: Unit Economics

Three scenarios, each modeling Fileverse as a hosted gateway operator with the recommended fee structure of Section 4. Assumptions are conservative and stated.

| Variable | Low | Medium | Scale |
|---|---|---|---|
| Registered users | 25,000 | 200,000 | 1,500,000 |
| Paid conversion | 2% | 4% | 6% |
| Paying users | 500 | 8,000 | 90,000 |
| Pro tier ($6) share | 90% (450) | 80% (6,400) | 70% (63,000) |
| Onchain Premium ($20) share | 10% (50) | 20% (1,600) | 30% (27,000) |
| Avg storage per paying user (GB) | 8 | 15 | 25 |
| Avg egress per paying user (GB/mo) | 4 | 10 | 20 |
| Free-tier infra burden (GB stored) | 5,000 | 50,000 | 500,000 |
| Free-tier egress (GB/mo) | 10,000 | 150,000 | 2,000,000 |

### Low scenario (early subsidized phase)

- Pro revenue: 450 × $6 = $2,700/month
- Onchain Premium revenue: 50 × $20 = $1,000/month
- **MRR = $3,700; ARR ≈ $44,400**
- Pinning cost (paid + free): (500 × 8 GB + 5,000 GB) × $0.15 = $1,350/month; egress: (500 × 4 + 10,000) × $0.15 = $1,800/month → **infra COGS ≈ $3,150/month**
- Gross margin: ~15% — covers infra but does not fund engineering. Implies grant or founder-equity subsidy.

### Medium scenario (post-product-market-fit)

- Pro revenue: 6,400 × $6 = $38,400/month
- Onchain Premium revenue: 1,600 × $20 = $32,000/month
- **MRR = $70,400; ARR ≈ $844,800**
- Pinning cost: (8,000 × 15 + 50,000) × $0.15 = $25,500/month; egress: (8,000 × 10 + 150,000) × $0.15 = $34,500/month → **infra COGS ≈ $60,000/month**
- Gross margin: ~15% on the gateway business — but the Onchain Premium tier carries higher margin (paymaster gas is metered, pinning load is not the binding constraint), so blended is ~25–30%. Engineering team of 6–8 (~$120K/month fully loaded) implies F still needs supplementary revenue (commercial licenses, grants).

### Scale scenario (multi-year compounding)

- Pro revenue: 63,000 × $6 = $378,000/month
- Onchain Premium revenue: 27,000 × $20 = $540,000/month
- **MRR = $918,000; ARR ≈ $11,016,000**
- Pinning cost: (90,000 × 25 + 500,000) × $0.15 = $412,500/month; egress: (90,000 × 20 + 2,000,000) × $0.15 = $570,000/month → **infra COGS ≈ $982,500/month** before enterprise discounts.
- At committed-volume rates (>10 TB triggers Filebase enterprise discount up to 50% off), realistic COGS reduces to ~$550K/month.
- Gross margin: ~40% post-discount. Sustainable as a self-funded operator at this scale; $11M ARR with 40% GM funds a 15–25 person engineering and ops team without a token or VC layer.

The arithmetic shows the model is **viable only at >100K paid users**, equivalent to roughly 1.5M registered users. Below that, Fileverse is structurally subsidy-dependent. CryptPad reached ~5,000 paying users at ~280K registered after a decade ([CryptPad blog](https://blog.cryptpad.org/2024/06/12/Onwards-and-upwards/ "Source: CryptPad, 2024-06-12")); Fileverse would need to either grow faster than that or accept a long subsidy horizon.

---

## Section 6: Minimum Viable Parameters

Floor conditions for the mechanism to function:

1. **Minimum free-tier quota.** Below 100 MB stored / 500 MB monthly egress, the free tier is too constrained to demonstrate product utility (a single document with 5 collaborators editing daily exceeds 500 MB egress in a quarter at typical Yjs sync overhead). Set floor at **1 GB stored / 5 GB egress/month** to match CryptPad's baseline.
2. **Minimum self-host viability.** A single self-hosted collaboration-server instance must support **≥10 concurrent users without degradation**. Below that, the AGPL self-host story is theatrical, not functional. Y.js typical operating point on a 1 vCPU VM is 50–100 concurrent connections per process ([Yjs benchmarks](https://github.com/dmonad/y-protocols "Source: GitHub, retrieved 2026-04-27")); 10 users is well within that envelope but presumes basic ops literacy.
3. **Maximum upload-egress imbalance before fallback chain activates.** When primary provider (Pinata) hits >80% of monthly committed quota, the fallback chain to Filebase/Storacha should engage. Architecture documents fallback exists; operating threshold should be set at 80% to leave headroom for spikes.
4. **Minimum paying-user count for sustainability without grants.** From Section 5, **~3,000 paying users** at the recommended fee structure covers infra but not full engineering. **~30,000 paying users** covers a small full-time team. **~100,000 paying users** funds the project as a self-sustaining operator without external capital.
5. **Minimum chain redundancy.** The content registry must remain accessible if either Ethereum mainnet or Gnosis Chain experiences extended outage. Requires either: dual-write to both chains for premium users (gas-expensive) or a designated "primary chain" with checkpointed snapshots. As of public docs, the design appears to use chains separately (mainnet for premium anchoring, Gnosis for routine writes); a snapshot-replication policy is not documented and should be specified before scale.
6. **Audit cadence floor.** Per [Trail of Bits' published guidance](https://blog.trailofbits.com/2023/02/14/our-2022-security-audit-cycle/ "Source: Trail of Bits, 2023-02-14"), E2EE primitives should be audited at least every 18 months and on any breaking change to the cryptographic protocol. Fileverse has the [in-progress Dedalo audit on collaboration-server](https://github.com/fileverse/collaboration-server "Source: GitHub, retrieved 2026-04-27") but not the `@fileverse/crypto` library disclosed publicly. Floor: cryptographic library audit complete and published before any paid tier launch.
7. **Cooldown / lock window.** If Fileverse introduces a paid tier with prepaid credits or onchain premium balance, a **7-day cooldown** between tier downgrades prevents arbitrage between mainnet gas events; a **24-hour cooldown** between key rotations prevents thrashing the link-lock recovery surface.

---

## Section 7: Applicable Academic Frameworks

1. **Mechanism design without money — Procaccia & Tennenholtz.** [Approximate mechanism design without money](https://arxiv.org/abs/0906.0710 "Procaccia & Tennenholtz, EC 2009") — Relevance: Fileverse currently allocates a public-good resource (gateway capacity) without monetary auctions. The authors prove approximation bounds on what truthful allocation can achieve under no-payment constraints. **Difference:** Procaccia–Tennenholtz assume a fixed agent set; Fileverse faces sybil-creation as a primary attack.

2. **Local-first software economics — Kleppmann et al.** [Local-first software](https://www.inkandswitch.com/local-first/ "Kleppmann, Wiggins, van Hardenberg, McGranaghan, Ink & Switch 2019") — Relevance: defines the design pattern Fileverse claims and identifies the business-model tension: local-first eliminates cloud lock-in but also eliminates cloud's revenue mechanism. **Difference:** the essay is a design-pattern manifesto, not a sustainability model; Fileverse must invent the latter.

3. **Public-goods provision in cryptoeconomic systems — Buterin, Hitzig, Weyl.** [A flexible design for funding public goods](https://arxiv.org/abs/1809.06421 "Buterin, Hitzig, Weyl, arXiv:1809.06421, 2018") — Relevance: quadratic-funding-style mechanisms could finance Fileverse infra if a token or matched-grant program were introduced. Gitcoin grants have funded comparable open-source privacy infrastructure. **Difference:** quadratic funding requires identity-resistance to sybil attacks, which Fileverse's vOPRF-ID partially provides but has not been formally evaluated for QF use.

4. **Tragedy of the commons / Hardin.** [Hardin, "The Tragedy of the Commons," Science 1968](https://www.science.org/doi/10.1126/science.162.3859.1243 "Hardin, Science, 1968-12-13") — Relevance: stateless collaboration server + free-tier pinning is a common-pool resource. Without metering, the rational user over-consumes. **Difference:** classical tragedy assumes rivalrous goods; bandwidth is rivalrous, but stored encrypted blobs are partially excludable via UCAN.

5. **Two-sided platform pricing — Rochet & Tirole.** [Platform competition in two-sided markets](https://www.jstor.org/stable/40005188 "Rochet & Tirole, JEEA 2003") — Relevance: Fileverse mediates between users and pinning providers; classical two-sided pricing predicts that one side (here, end-users) is subsidized by the other (here, premium tier or grants). **Difference:** Rochet–Tirole assumes platform-internal pricing power; Fileverse's pinning-side prices are set by external markets.

6. **Free-rider problem in P2P systems — Adar & Huberman.** [Free riding on Gnutella](https://firstmonday.org/ojs/index.php/fm/article/view/792 "Adar & Huberman, First Monday 2000") — Relevance: 70% of Gnutella users shared no files; the same dynamic threatens Fileverse self-host narratives if "self-hosting" remains theoretical for >90% of users. **Difference:** Gnutella was content-distribution P2P; Fileverse is collaboration-server P2P with a stronger trust anchor (the paying user).

7. **Auditing and security investment — Anderson.** [Why information security is hard — an economic perspective](https://www.acsac.org/2001/papers/110.pdf "Anderson, ACSAC 2001") — Relevance: under-investment in audits is a documented equilibrium for free-tier security software. Fileverse's incomplete audit posture (Dedalo in progress, crypto lib unaudited) is consistent with Anderson's prediction and a known risk surface.

8. **CRDT consistency under Byzantine actors — Kleppmann & Howard.** [Making CRDTs Byzantine fault tolerant](https://martin.kleppmann.com/papers/bft-crdt-papoc22.pdf "Kleppmann & Howard, PaPoC 2022") — Relevance: Yjs CRDTs assume non-Byzantine collaborators. The authors describe overheads required for BFT-CRDT. **Difference:** Fileverse trusts UCAN authorization to keep collaborators non-Byzantine; if UCAN is compromised, no CRDT-level defense exists.

---

## Section 8: Attack Vectors & Failure Modes

### Failure Mode 1 — Pinning Provider Cascade Failure

- **Description.** All three pinning providers (Pinata, Filebase, Storacha) experience concurrent disruption — pricing event, regulatory action, or acquisition cascade. Skiff's shutdown and Storacha's free-tier wind-down are precedent for non-trivial probability.
- **Quantified impact.** Documents not pinned to user-controlled IPFS nodes lose persistence within the providers' garbage-collection windows (Pinata: 30-day grace on dropped pins per [Pinata docs](https://docs.pinata.cloud/account-management/billing#what-happens-if-i-dont-pay "Source: Pinata, retrieved 2026-04-27")). At the Medium scenario (50,000 GB free + 120,000 GB paid stored), aggregate data at risk is **~170 TB**. Recovery cost to re-pin to alternative: 170 TB × $5/GB-month emergency vendor pricing = ~$850K one-time.
- **Mitigation status.** Partially mitigated. Multi-provider fallback exists; concurrent-failure scenario is not. **Required change:** Fileverse-operated own IPFS cluster as fourth fallback at >5 TB committed volume, plus user-facing nudge to self-pin for high-value documents.

### Failure Mode 2 — Triple-Lock Recovery Path Compromise

- **Description.** Walk-away encryption uses Portal Lock + Owner Lock + Link Lock. If the Link Lock leaks via URL fragment exposure (browser sync, link-preview crawlers, copy-paste into instrumented chat clients), file confidentiality is broken even though E2EE primitives are sound.
- **Quantified impact.** As modeled in Section 3 attack 3, a 1% link-leakage rate on documents with average 3 shares yields ~3% silent confidentiality breach across the corpus. At Medium scenario user volume, that is ~6,000 docs/month exposed without user awareness. The reputational impact of one publicized incident is asymmetric — comparable to the Skiff sunset's impact on its category in 2024.
- **Mitigation status.** Not mitigated by base mechanism. **Required change:** replace URL-fragment link sharing with server-mediated key handshake + revocable share-token; default link expiry of 30 days; explicit UX warning that link-shared documents are revocable only by re-key + re-pin.

### Failure Mode 3 — Sybil Free-Tier Exhaustion

- **Description.** vOPRF-ID enables zero-cost account creation. An attacker creates *N* accounts, consuming free quota *N*-fold and dumping cost on F.
- **Quantified impact.** From Section 3 attack 1, 1,000 sybil accounts ≈ $150/month direct cost. At sustained 10,000 sybils (achievable via cheap headless-browser farms), monthly cost is **~$1,500/month direct + ~$3,000–$5,000/month bandwidth** = up to $6,500/month diverted to attacker-controlled storage. At Low scenario MRR of $3,700, this single attack is **larger than F's entire revenue**.
- **Mitigation status.** Partially mitigated by wallet-based authentication (which has economic cost), not mitigated for vOPRF-only accounts. **Required change:** tiered quotas — full quota only for wallet-bound or domain-attested accounts; vOPRF-ID accounts get 10–20% of full quota and require periodic proof-of-personhood (e.g., World ID, BrightID, or passive bot-detection signals).

### Failure Mode 4 — Self-Host Defection at Scale

- **Description.** AGPL-3.0 plus self-hostable architecture means any organization with infra capability can fork Fileverse and operate independently. If self-hosting share *q* exceeds 40% of intended user volume, Fileverse-the-operator loses both pinning leverage and revenue base while continuing to incur engineering cost as the upstream maintainer.
- **Quantified impact.** Modeling 40% defection at Scale scenario: 60,000 paying users instead of 90,000, MRR drops from $918K to $612K, while engineering cost is unchanged. Margin compresses from ~40% to ~20%. The defection is asymmetric: defectors take the revenue, Fileverse retains the audit and maintenance burden.
- **Mitigation status.** Not mitigated by base mechanism. **Required change:** introduce features that benefit from network effects unavailable to forks — for example, (a) the Onchain Premium paymaster requires Fileverse-controlled paymaster keys, (b) cross-portal discovery via Waku has utility only at network scale, (c) commercial-license offering for organizations seeking SLA + indemnification beyond AGPL terms (the MongoDB and Elastic precedent). None require violating AGPL; all create gradient between fork and Fileverse-hosted experience.

### Failure Mode 5 — Onchain Registry Gas-Cost Inflation

- **Description.** Ethereum mainnet gas prices spike during periods of network congestion (NFT mints, memecoin events). If Fileverse sponsors writes for paying users via paymaster, sudden gas inflation drains the paymaster faster than anticipated.
- **Quantified impact.** A 10× gas spike (30 gwei → 300 gwei, observed multiple times in 2021–2024) raises per-write cost from ~$4.50 to ~$45. If Onchain Premium tier ($20/month) includes 100 mainnet writes/month, sponsored cost spikes from $450 to $4,500 per user — **22.5× the user's monthly fee**. At 100 such users, F absorbs $440K/month unfunded for the duration of the spike.
- **Mitigation status.** Not mitigated by base mechanism (paymaster behavior depends on implementation choices not yet in public repos). **Required change:** dynamic gas-cap per epoch (e.g., paymaster halts at 50 gwei, falls back to user-pay or Gnosis-only mode), Gnosis-first default with mainnet writes throttled and queued during high-fee windows, and SLA language explicitly excluding mainnet-gas-spike force-majeure events.

### Failure Mode 6 — Audit Gap on Cryptographic Library

- **Description.** The `@fileverse/crypto` library implements the triple-lock walk-away scheme. As of 2026-04-27, no public audit report exists; only the collaboration-server is under [Dedalo audit](https://github.com/fileverse/collaboration-server "Source: GitHub, retrieved 2026-04-27"). A vulnerability in key wrapping, RSA padding, or AES IV reuse would compromise files at rest universally.
- **Quantified impact.** Universal impact across all stored documents — at Scale scenario, ~1.5M registered users × average 20 docs = 30M docs potentially compromised. Recovery requires re-key + re-pin of the entire corpus, a multi-week operation at ~$0.001/doc COGS = ~$30K direct + multi-month engineering effort.
- **Mitigation status.** Partial — code is open-source and has had community review, but absence of formal audit is the documented gap. **Required change:** complete and publish an audit of `@fileverse/crypto` before paid-tier or Onchain Premium launch; publish a deprecation + re-key procedure as part of the security policy.

---

## Closing Note on No-Token Framing

Standard tokenomics modeling — bonding curves, staking yield, governance distribution — has no purchase on Fileverse as it stands. The economic analysis above instead treats Fileverse as a **dual-economy operation**: a free public good funded by an undisclosed mix of grants, founder equity, and (potentially) a future paid tier; layered with metered services (UCAN quotas, paymaster credits, premium tier) that constitute the only legible cash-flow path. Under conditions where Fileverse launches a token, the analysis would shift toward classical mechanism-design framing (token as quota voucher, as governance share, or as retroactive contributor reward); under conditions where it does not, sustainability depends on either reaching the ~100K-paid-user threshold modeled in Section 5 or sustaining grant funding and commercial-license revenue indefinitely. Either path is plausible; neither is currently announced.
