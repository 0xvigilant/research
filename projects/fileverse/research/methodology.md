# Methodology — Fileverse

_Phase 3.5 · Compiled 2026-04-27 · Researcher-portfolio mode_

This document records how the Fileverse research report was produced: what questions framed the inquiry, which sources were consulted, what frameworks structured the analysis, what the production process looked like phase-by-phase, what the report does not cover, and what the researcher discloses about positions, conflicts, and compensation. The aim is to let a reader assess the report's evidentiary basis without reverse-engineering it from the body text.

---

## Section 1: Research Questions

The report attempts to answer the following questions, derived from the [concept document](https://fileverse.io "Source: Fileverse, retrieved 2026-04-27") and the open questions logged across the Phase 1–3 outputs.

1. **What does Fileverse actually do, and how does its architecture differ in structurally important ways from existing E2EE productivity suites and decentralized-storage stacks?** The concept document describes dDocs, dSheets, the triple-lock walk-away scheme, multi-provider IPFS pinning, and an Ethereum/Gnosis content registry; the question is which of those are differentiated and which are conventional.

2. **How does Fileverse compare to live and defunct competitors — CryptPad, Proton Docs/Sheets, Skiff, Anytype, AFFiNE, ONLYOFFICE, Lit Protocol, Dune Analytics — across cryptographic posture, persistence model, license, onchain integration, funding model, and walk-away portability?** Phase 1 enumerated 26 competitors; Phase 2a reduced that to eight teardowns spanning the relevant axes.

3. **Where are the load-bearing technical degrees of freedom in the architecture, and what tradeoffs do current implementation choices imply?** The most consequential design freedom identified in the technical-architecture phase is how encrypted CRDT updates are authenticated and merged on an untrusted relay (Option A: opaque ciphertext relay; Option B: secsync-style signed updates with key-rotation snapshots).

4. **What is the unit-economics shape of Fileverse-as-gateway-operator absent a token, and what scale of paying users is required for self-sustaining operation?** With no disclosed native token, no published fee schedule, and no announced venture round, conventional tokenomics framing does not apply; the question is what the dual-economy of free public good plus metered services implies for sustainability.

5. **What are the dominant attack vectors and failure modes specific to this design, and how mitigated are they at the base mechanism layer?** The questions are which vectors are unique to Fileverse versus generic web app issues, and which mitigations exist in code today versus require future work.

6. **What would a successful 12–24 month execution look like under the architecture, competitive position, and economic constraints currently visible?** This includes the conditions an analyst could observe to read the project as on-track, the conflicts surfaced across the technical and economic phases, and the recommended resolution for each.

7. **What is unresolved or underdetermined by public materials?** The on-chain registry contract source, the exact wire format of `encrypted_yjs_update`, the Dedalo audit scope, the dSheets per-cell trust boundary, the local-LLM integration's threat model, and the funding path are all partially documented at best.

---

## Section 2: Sources Consulted

Sources are deduplicated by URL across Phase 1–3 outputs. Categories follow the inline citation format applied throughout the report: `[Source title](URL "Source: Publisher, YYYY-MM-DD")`.

### Primary sources — Fileverse repositories and homepage

- [Fileverse homepage](https://fileverse.io "Source: Fileverse, retrieved 2026-04-27") — Stated value proposition, product overview, conference deployments, and the absence of any disclosed token or fundraising round.
- [`fileverse/walk-away`](https://github.com/fileverse/walk-away "Source: Fileverse walk-away README, retrieved 2026-04-27") — Documentation of the RSA + AES hybrid encryption scheme and the Portal/Owner/Link triple-lock recovery model.
- [`fileverse/fileverse-storage-v2`](https://github.com/fileverse/fileverse-storage-v2 "Source: Fileverse fileverse-storage-v2 README, retrieved 2026-04-27") — Storage gateway: multi-provider IPFS pinning, UCAN authorization, registry chain headers (chain ID 1 default).
- [`fileverse/collaboration-server`](https://github.com/fileverse/collaboration-server "Source: Fileverse collaboration-server README, retrieved 2026-04-27") — Stateless WebSocket relay for `encrypted_yjs_update` payloads, Gnosis Chain RPC reference, Waku discovery branch, Dedalo audit note.
- [`fileverse/fileverse-ddoc`](https://github.com/fileverse/fileverse-ddoc "Source: Fileverse ddoc README, retrieved 2026-04-27") — Client editor: TipTap surface, viem dependency, IndexedDB sync flag, AGPL-3.0 license.
- [`fileverse/fileverse-dsheet`](https://github.com/fileverse/fileverse-dsheet "Source: Fileverse fileverse-dsheet README, retrieved 2026-04-27") — Spreadsheet client, Fortune Sheet engine, V0.3 onchain-write feature.
- [`fileverse/fortune-sheet`](https://github.com/fileverse/fortune-sheet "Source: Fileverse fortune-sheet repo, retrieved 2026-04-27") — In-tree fork of the spreadsheet engine.
- [ddocs.new](https://ddocs.new "Source: Fileverse, retrieved 2026-04-27") — Live dDocs deployment.
- [dsheets.new](https://dsheets.new "Source: Fileverse, retrieved 2026-04-27") — Live dSheets deployment.

### On-chain / explorer / gas data

- [Etherscan Gas Tracker](https://etherscan.io/gastracker "Source: Etherscan Gas Tracker, retrieved 2026-04-27") — Mainnet gas observations supporting the per-document-write cost ranges in the technical-architecture and economics phases.

### Competing protocols and adjacent products

- [CryptPad homepage](https://cryptpad.org/ "Source: CryptPad, retrieved 2026-04-27"), [features](https://cryptpad.org/features/ "Source: CryptPad Features, retrieved 2026-04-27"), [pricing](https://cryptpad.fr/pricing/ "Source: CryptPad Pricing, retrieved 2026-04-27"), [pinning docs](https://docs.cryptpad.org/en/user_guide/pinning.html "Source: CryptPad Docs, retrieved 2026-04-27"), [whitepaper](https://blog.cryptpad.org/2023/02/02/CryptPad-whitepaper/ "Source: CryptPad Blog, 2023-02-02"), [2023 roadmap recap](https://blog.cryptpad.org/2023/12/14/CryptPad-2023-roadmap-recap/ "Source: CryptPad Blog, 2023-12-14"), [Onwards & Upwards 2024 metrics post](https://blog.cryptpad.org/2024/06/12/Onwards-and-upwards/ "Source: CryptPad Blog, 2024-06-12") — Operating model, paid-user counts, scale curve over a decade.
- [Proton Drive Docs page](https://proton.me/drive/docs "Source: Proton, retrieved 2026-04-27"), [Proton Docs encryption blog](https://proton.me/blog/docs-proton-drive "Source: Proton Blog, 2024-07-03"), [Proton pricing](https://proton.me/pricing "Source: Proton Pricing, retrieved 2026-04-27"), [Proton Drive pricing](https://proton.me/drive/pricing "Source: Proton, retrieved 2026-04-27"), [Proton non-profit foundation announcement](https://proton.me/blog/proton-non-profit-foundation "Source: Proton Blog, 2024-06-17"), [100M accounts post](https://proton.me/blog/100-million-accounts "Source: Proton Blog, 2024-09-17"), [transparency report](https://proton.me/blog/transparency-report "Source: Proton Blog, retrieved 2026-04-27") — Comparable encryption model, scale, and non-profit structural choice.
- [MacRumors on Proton Sheets launch](https://www.macrumors.com/2025/12/04/proton-sheets-launches-encrypted-spreadsheet/ "Source: MacRumors, 2025-12-04") — Independent reporting on launch date and feature gaps.
- [404 Media on Proton recovery-email disclosure](https://www.404media.co/proton-mail-discloses-recovery-email-to-spanish-police/ "Source: 404 Media, 2024-05-06") — Custodial-metadata exposure case relevant to the comparison.
- [CyberInsider on Skiff shutdown](https://cyberinsider.com/skiff-shutting-down-alternatives-to-skiff-mail/ "Source: CyberInsider, 2024-02-12") and [The Verge coverage](https://www.theverge.com/24039915/skiff-shutting-down-notion-acquired "Source: The Verge, 2024-02-12") — Acquihire-then-sunset precedent.
- [Wayback-archived Skiff IPFS post](https://skiff.com/blog/skiff-ipfs "Source: Wayback Machine — Skiff Blog, 2022-11-15"), [Skiff Web3 post](https://skiff.com/blog/web3 "Source: Wayback Machine — Skiff Blog, retrieved 2026-04-27"), [Skiff pricing snapshot](https://web.archive.org/web/20240201000000*/skiff.com/pricing "Source: Wayback Machine — Skiff Pricing, retrieved 2024-02-01"), [TechCrunch Skiff seed coverage](https://techcrunch.com/2022/03/24/skiff-seed-round/ "Source: TechCrunch, 2022-03-24") — Historical product, funding, and IPFS-mode details.
- [Anytype docs on data security](https://doc.anytype.io/anytype-docs/advanced/data-and-security/how-we-keep-your-data-safe "Source: Anytype Docs, retrieved 2026-04-27"), [Any-Sync repo](https://github.com/anyproto/any-sync "Source: GitHub, retrieved 2026-04-27"), [Any-Sync architecture](https://github.com/anyproto/any-sync/blob/main/docs/architecture.md "Source: Any-Sync Architecture, retrieved 2026-04-27"), [Anytype 1.0 blog](https://blog.anytype.io/anytype-1-0/ "Source: Anytype Blog, retrieved 2026-04-27"), [Anytype pricing](https://anytype.io/pricing "Source: Anytype Pricing, retrieved 2026-04-27") — Object-graph local-first comparator, Swiss non-profit structure.
- [AFFiNE notion-alternative blog](https://affine.pro/blog/notion-alternative "Source: AFFiNE Blog, retrieved 2026-04-27"), [AFFiNE Cloud security docs](https://docs.affine.pro/docs/affine-cloud/security "Source: AFFiNE Docs, retrieved 2026-04-27"), [AFFiNE pricing](https://affine.pro/pricing "Source: AFFiNE Pricing, retrieved 2026-04-27"), [About page](https://affine.pro/about-us "Source: AFFiNE About, retrieved 2026-04-27"), [GitHub issues](https://github.com/toeverything/AFFiNE/issues "Source: GitHub Issues, retrieved 2026-04-27"), [TechCrunch seed coverage](https://techcrunch.com/2023/02/22/affine-seed-funding/ "Source: TechCrunch, 2023-02-22") — Local-first Yjs-based competitor with VC funding profile.
- [Logseq homepage](https://logseq.com/ "Source: Logseq, retrieved 2026-04-27"), [AppFlowy entry on OpenAlternative](https://openalternative.co/alternatives/notion "Source: OpenAlternative, retrieved 2026-04-27"), [Standard Notes homepage](https://standardnotes.com/ "Source: Standard Notes, retrieved 2026-04-27") and [plans page](https://standardnotes.com/plans "Source: Standard Notes, retrieved 2026-04-27"), [NLnet EteSync project page](https://nlnet.nl/project/EteSyncEnhancements/ "Source: NLnet, retrieved 2026-04-27"), [Etebase homepage](https://www.etebase.com/ "Source: Etebase, retrieved 2026-04-27") — Adjacent E2EE notes, wikis, and SDKs.
- [ONLYOFFICE homepage](https://www.onlyoffice.com/ "Source: ONLYOFFICE, retrieved 2026-04-27"), [pricing](https://www.onlyoffice.com/docs-enterprise-prices.aspx "Source: ONLYOFFICE Pricing, retrieved 2026-04-27"), [business edition](https://www.onlyoffice.com/business-edition.aspx "Source: ONLYOFFICE Business, retrieved 2026-04-27"), [customers](https://www.onlyoffice.com/customers.aspx "Source: ONLYOFFICE Customers, retrieved 2026-04-27"), [Private Rooms help](https://helpcenter.onlyoffice.com/installation/docs-developer-private-rooms.aspx "Source: ONLYOFFICE Help Center, retrieved 2026-04-27") — Self-hosting procurement comparator.
- [Etherpad](https://etherpad.org/ "Source: Etherpad Foundation, retrieved 2026-04-27"), [Collabora Online](https://www.collaboraonline.com/ "Source: Collabora, retrieved 2026-04-27"), [Cryptee](https://crypt.ee/ "Source: Cryptee, retrieved 2026-04-27"), [Outline](https://www.getoutline.com/ "Source: Outline, retrieved 2026-04-27"), [Docmost via OpenAlternative](https://openalternative.co/alternatives/notion "Source: OpenAlternative, retrieved 2026-04-27"), [SaaSHub Arcane Office entry](https://www.saashub.com/cryptpad-alternatives "Source: SaaSHub, retrieved 2026-04-27"), [Internxt blog](https://blog.internxt.com/alternative-to-skiff/ "Source: Internxt Blog, retrieved 2026-04-27") — Adjacent collaborative editors, self-hosted office stacks, and E2EE drives.
- [Lit Protocol Spark blog on private data](https://spark.litprotocol.com/private-data-on-the-open-web/ "Source: Lit Protocol Spark, retrieved 2026-04-27"), [Datil mainnet announcement](https://spark.litprotocol.com/datil-mainnet/ "Source: Lit Spark, 2024-09"), [Lit developer docs networks page](https://developer.litprotocol.com/network/networks/mainnet "Source: Lit Developer Docs, retrieved 2026-04-27"), [capacity credits docs](https://developer.litprotocol.com/concepts/capacity-credits "Source: Lit Developer Docs, retrieved 2026-04-27"), [security and trust docs](https://developer.litprotocol.com/security-and-trust "Source: Lit Developer Docs, retrieved 2026-04-27"), [case studies](https://litprotocol.com/case-studies "Source: Lit Protocol, retrieved 2026-04-27"), [CoinDesk funding coverage](https://www.coindesk.com/business/2023/04/04/lit-protocol-raises-13m/ "Source: CoinDesk, 2023-04-04") — Threshold-decryption infrastructure comparator.
- [Dune homepage](https://dune.com/ "Source: Dune, retrieved 2026-04-27"), [data catalog](https://docs.dune.com/data-catalog/overview "Source: Dune Docs, retrieved 2026-04-27"), [pricing](https://dune.com/pricing "Source: Dune Pricing, retrieved 2026-04-27"), [customers](https://dune.com/customers "Source: Dune Customers, retrieved 2026-04-27"), [TechCrunch Series B coverage](https://techcrunch.com/2022/02/02/dune-analytics-series-b/ "Source: TechCrunch, 2022-02-02") — Onchain analytics substitute.
- [Web3 Sheets by GFX Labs](https://sheets.gfx.xyz/ "Source: GFX Labs, retrieved 2026-04-27"), [Blockchain For Sheet Workspace listing](https://workspace.google.com/marketplace/app/blockchain_for_sheet/413227996607 "Source: Google Workspace Marketplace, retrieved 2026-04-27"), [The Graph](https://thegraph.com/ "Source: The Graph, retrieved 2026-04-27"), [Apillon Web3 alternative recipe](https://blog.apillon.io/apillon-recipe-3-a-web3-alternative-to-google-drive-dropbox-dd764c1556bb/ "Source: Apillon Blog, retrieved 2026-04-27"), [Ceramic how-it-works](https://ceramic.network/how-it-works "Source: Ceramic, retrieved 2026-04-27"), [SecSync GitHub](https://github.com/nikgraf/secsync "Source: SecSync GitHub, retrieved 2026-04-27") — Onchain spreadsheet, indexer, storage-gateway, and reference-implementation neighbors.
- [ethereum.org dSheets entry](https://ethereum.org/en/apps/fileverse-dsheets/ "Source: ethereum.org, retrieved 2026-04-27") — Independent listing of dSheets among ecosystem applications.

### Pinning, IPFS, and infrastructure providers

- [Pinata pricing](https://www.pinata.cloud/pricing "Source: Pinata, retrieved 2026-04-27"), [Pinata billing FAQ](https://docs.pinata.cloud/account-management/billing#what-happens-if-i-dont-pay "Source: Pinata, retrieved 2026-04-27") — Storage and egress unit economics, dropped-pin grace period.
- [Filebase pricing](https://filebase.com/pricing "Source: Filebase, retrieved 2026-04-27"), [Filebase pricing alt](https://filebase.com/pricing/ "Source: Filebase, retrieved 2026-04-27") — Pay-as-you-go and committed-tier thresholds.
- [Web3.Storage homepage](https://web3.storage "Source: Web3.Storage, retrieved 2026-04-27"), [Web3.Storage data layer post](https://blog.web3.storage/posts/the-data-layer-is-here-with-the-new-web3-storage "Source: Web3.Storage blog, 2024-04-15"), [Storacha rebrand post](https://blog.storacha.network/web3-storage-rebrands-to-storacha-network/ "Source: Storacha (formerly Web3.Storage), 2024") — Provider repositioning event used in cascade-failure modeling.
- [IPFS Pinning Service API spec](https://ipfs.github.io/pinning-services-api-spec/ "Source: IPFS PSA spec, retrieved 2026-04-27") — Standard for community-run pinning interoperability.
- [AWS data-transfer pricing](https://aws.amazon.com/ec2/pricing/on-demand/#Data_Transfer "Source: AWS, retrieved 2026-04-27") — Egress benchmark for stateless-relay free-rider modeling.
- [TweetNaCl.js](https://tweetnacl.js.org "Source: TweetNaCl.js, retrieved 2026-04-27") — Audited cryptographic primitive used in walk-away.

### Chain and ecosystem references

- [Polygon Heimdall recovery post-mortem](https://polygon.technology/blog/heimdall-recovery-and-our-path-forward "Source: Polygon Labs blog, 2023-03-15") — Reorg-history reference for chain selection comparison.
- [Safe homepage](https://safe.global "Source: Safe, retrieved 2026-04-27") — Multisig referenced in persona modeling and registry-admin recommendation.
- [ENS docs](https://docs.ens.domains "Source: ENS Docs, retrieved 2026-04-27") — ENS-based access primitive used in dSheets permissions.
- [Filecoin homepage](https://filecoin.io "Source: Filecoin, retrieved 2026-04-27") — Cold-storage tier candidate for the fourth-fallback option.
- [Ledger shop](https://shop.ledger.com "Source: Ledger, retrieved 2026-04-27") — Hardware wallet referenced in privacy-conscious-persona modeling.
- [World](https://world.org "Source: World, retrieved 2026-04-27") — Proof-of-personhood candidate for sybil-mitigation policy.
- [Google Workspace pricing](https://workspace.google.com/pricing.html "Source: Google, retrieved 2026-04-27") — Per-seat baseline for cost-structure comparison.

### Academic literature and protocol white papers

- Petru Nicolaescu et al., [_Yjs: A framework for near real-time P2P shared editing on arbitrary data types_](https://link.springer.com/chapter/10.1007/978-3-319-19890-3_55 "Nicolaescu et al., Yjs, ICWE 2015") — CRDT library underpinning Fileverse's collaboration server.
- Martin Kleppmann & Alastair R. Beresford, [_A Conflict-Free Replicated JSON Datatype_](https://arxiv.org/abs/1608.03960 "Kleppmann & Beresford, arXiv:1608.03960, 2016") — Formal basis for strong eventual consistency under arbitrary message order.
- Geoffrey Litt et al., [_Peritext: A CRDT for Rich-Text Collaboration_](https://www.inkandswitch.com/peritext/ "Ink & Switch, 2022") — Rich-text CRDT relevant to dDocs's editing surface.
- Martin Kleppmann et al., [_Local-first software: You own your data, in spite of the cloud_](https://www.inkandswitch.com/local-first/ "Kleppmann, Wiggins, van Hardenberg, McGranaghan, Ink & Switch 2019") — Canonical local-first essay used as a normative reference.
- Nik Graf et al., [_secsync: Architecture for end-to-end encrypted CRDTs_](https://github.com/nikgraf/secsync "Graf, secsync, 2023") — Reference protocol for signed encrypted Yjs updates.
- Dennis Trautwein et al., [_Design and Evaluation of IPFS_](https://dl.acm.org/doi/10.1145/3544216.3544232 "Trautwein et al., SIGCOMM 2022") — Empirical IPFS reliability analysis underlying the cascade-failure modeling.
- Juan Benet, [_IPFS — Content Addressed, Versioned, P2P File System_](https://arxiv.org/abs/1407.3561 "Benet, arXiv:1407.3561, 2014") — Original IPFS whitepaper.
- ENS Lyon analysis, [URL-fragment key sharing weaknesses](https://hal.science/hal-01781231 "Source: HAL Archive, 2018") — 2018 academic note on Link-Lock-style URL-fragment leakage.
- Ariel Procaccia & Moshe Tennenholtz, [_Approximate mechanism design without money_](https://arxiv.org/abs/0906.0710 "Procaccia & Tennenholtz, EC 2009") — Mechanism-design framing applied to gateway capacity allocation.
- Vitalik Buterin, Zoë Hitzig, E. Glen Weyl, [_A flexible design for funding public goods_](https://arxiv.org/abs/1809.06421 "Buterin, Hitzig, Weyl, arXiv:1809.06421, 2018") — Quadratic-funding framing applied to grant-funded scenarios.
- Garrett Hardin, [_The Tragedy of the Commons_](https://www.science.org/doi/10.1126/science.162.3859.1243 "Hardin, Science, 1968-12-13") — Common-pool-resource framing for free-tier consumption.
- Jean-Charles Rochet & Jean Tirole, [_Platform competition in two-sided markets_](https://www.jstor.org/stable/40005188 "Rochet & Tirole, JEEA 2003") — Two-sided platform pricing applied to gateway-as-mediator.
- Eytan Adar & Bernardo Huberman, [_Free riding on Gnutella_](https://firstmonday.org/ojs/index.php/fm/article/view/792 "Adar & Huberman, First Monday 2000") — P2P free-rider reference for the self-host narrative.
- Ross Anderson, [_Why information security is hard — an economic perspective_](https://www.acsac.org/2001/papers/110.pdf "Anderson, ACSAC 2001") — Audit underinvestment as predicted equilibrium.
- Martin Kleppmann & Heidi Howard, [_Making CRDTs Byzantine fault tolerant_](https://martin.kleppmann.com/papers/bft-crdt-papoc22.pdf "Kleppmann & Howard, PaPoC 2022") — BFT-CRDT cost framing.
- [Yjs benchmarks (y-protocols)](https://github.com/dmonad/y-protocols "Source: GitHub, retrieved 2026-04-27") — Operating-point references for self-host viability floor.
- [NLnet/NGI Trust CryptPad funding page](https://nlnet.nl/project/CryptPad-NGITrust/ "Source: NLnet, retrieved 2026-04-27") — Public-interest funding precedent.

### Industry / practitioner sources

- [Trail of Bits 2022 audit-cycle post](https://blog.trailofbits.com/2023/02/14/our-2022-security-audit-cycle/ "Source: Trail of Bits, 2023-02-14") — Cryptographic re-audit cadence guidance.

The deduplicated count across these categories exceeds the ten-source gate threshold by a wide margin; full bibliographic entries appear inline at point-of-claim throughout the body sections.

---

## Section 3: Frameworks Applied

The frameworks listed below were applied to specific subsections of the report. We name each, identify its origin, and describe how it was used. We do not claim formal simulation or quantitative model implementation beyond the scenario arithmetic in the economics phase.

1. **Mechanism design without money — Procaccia & Tennenholtz (EC 2009).** Applied in the economics phase to frame UCAN-gated free-tier allocation as a no-payment public-goods allocation problem; used to surface the gap between the framework's fixed-agent assumption and Fileverse's open sybil surface ([Procaccia & Tennenholtz, arXiv:0906.0710](https://arxiv.org/abs/0906.0710 "Procaccia & Tennenholtz, EC 2009")).

2. **Local-first software design pattern — Kleppmann, Wiggins, van Hardenberg, McGranaghan (Ink & Switch 2019).** Used as the normative reference for "local-first" claims throughout the report; applied to assess whether Fileverse, AFFiNE, Anytype, and Logseq satisfy the seven local-first ideals and where each diverges ([Ink & Switch local-first essay](https://www.inkandswitch.com/local-first/ "Kleppmann et al., Ink & Switch 2019")).

3. **Two-sided platform pricing — Rochet & Tirole (JEEA 2003).** Applied to model Fileverse-as-gateway-operator as a mediator between users and pinning providers, with the prediction that one side is subsidized by the other; used to bound the expected paid-tier price corridor ([Rochet & Tirole, JEEA 2003](https://www.jstor.org/stable/40005188 "Rochet & Tirole, JEEA 2003")).

4. **Public-goods funding via quadratic mechanisms — Buterin, Hitzig, Weyl (2018).** Used to consider grant-funded and quadratic-funded scenarios as alternatives to a token launch, with explicit caveat that QF requires sybil resistance Fileverse has not formally evaluated ([Buterin, Hitzig, Weyl, arXiv:1809.06421](https://arxiv.org/abs/1809.06421 "Buterin, Hitzig, Weyl, 2018")).

5. **Tragedy of the commons — Hardin (Science 1968).** Applied to the stateless collaboration server and free-tier pinning as common-pool resources; used to motivate UCAN-bound metering at the WebSocket-handshake layer ([Hardin, Science 1968](https://www.science.org/doi/10.1126/science.162.3859.1243 "Hardin, Science, 1968-12-13")).

6. **Free-riding in P2P systems — Adar & Huberman (First Monday 2000).** Used as a prior on what fraction of "self-hosters" actually self-host versus rely on the operator's gateway; informs the 5–25% self-hosting band recommended in the operational gates ([Adar & Huberman, First Monday 2000](https://firstmonday.org/ojs/index.php/fm/article/view/792 "Adar & Huberman, First Monday 2000")).

7. **Security investment economics — Anderson (ACSAC 2001).** Used to interpret the audit-gap on `@fileverse/crypto` as a documented equilibrium rather than a one-off oversight, and to recommend audit-first sequencing before paid-tier launch ([Anderson, ACSAC 2001](https://www.acsac.org/2001/papers/110.pdf "Anderson, ACSAC 2001")).

8. **Byzantine-fault-tolerant CRDTs — Kleppmann & Howard (PaPoC 2022).** Used to characterize the threat surface of Yjs's non-Byzantine assumption when UCAN authorization is the only line of defense at the relay layer ([Kleppmann & Howard, PaPoC 2022](https://martin.kleppmann.com/papers/bft-crdt-papoc22.pdf "Kleppmann & Howard, PaPoC 2022")).

9. **Threat modeling (ad hoc, vector-by-vector).** Eight Fileverse-specific attack vectors were enumerated in the technical-architecture phase and six failure modes in the economics phase. We did not apply STRIDE or another formal taxonomy verbatim; the structure resembles attack-tree decomposition with severity tags (HIGH / MODERATE) and explicit mitigation status ("partial," "not mitigated by base mechanism").

10. **Trail of Bits audit-cycle guidance.** Applied as an operational SLO benchmark: 18-month re-audit cadence on cryptographic primitives and audit-on-breaking-change ([Trail of Bits, 2023-02-14](https://blog.trailofbits.com/2023/02/14/our-2022-security-audit-cycle/ "Source: Trail of Bits, 2023-02-14")).

11. **Comparative-teardown method (six-section per competitor).** A conventional analyst-firm framing — Mechanism / Revenue / Persistence / UX & Target / Known Failures / Comparison-to-subject — applied uniformly across the eight Phase 2a teardowns and rolled up into a thirteen-dimension comparison matrix.

We did not apply formal token-engineering primitives (bonding curves, conviction voting, augmented bonding curves) because Fileverse has no disclosed token; classical tokenomics framing was explicitly excluded by the structure of the asset under study. We did not run agent-based simulation, formal game-theoretic equilibrium proofs, or Monte Carlo sensitivity analysis on the unit-economics scenarios; the scenario arithmetic is back-of-envelope point estimates with stated assumptions.

---

## Section 4: Process

Phases were produced sequentially on 2026-04-27 and logged in [research-log.md](https://github.com/fileverse "Source: Fileverse, retrieved 2026-04-27") (the GitHub link here points only to the public org for reference; the log itself is local to this project). We describe each phase's task, agent, and artifact.

**Phase 0 — Concept document.** The seed document `concept.md` was compiled on 2026-04-27 from a fetch of the Fileverse homepage and the public GitHub organization. It enumerated dDocs and dSheets as live products, the multi-provider IPFS pinning architecture, the triple-lock encryption scheme, the wallet/vOPRF authentication modes, and eight open research questions. It identified ten initial sources (homepage, organization page, six core repositories, two live product URLs).

**Phase 1 — Competitor mapping.** Completed at 2026-04-27 15:46. The `research-competitor-mapper` agent produced `competitors.md` with 26 entries spanning E2EE collaborative productivity (CryptPad, Proton, Skiff, Anytype, AFFiNE, Logseq, AppFlowy, Standard Notes, Etebase, ONLYOFFICE, Etherpad, Collabora, Cryptee, Arcane Office, Serenity Notes), onchain spreadsheets and analytics (dSheets self-reference, Web3 Sheets, Blockchain For Sheet, Dune, The Graph), Web3 storage (Apillon, Ceramic, Lit Protocol, Internxt), and self-hosted wikis (Outline, Docmost). An academic-references appendix added eight foundational papers (Yjs, JSON CRDT, Peritext, local-first, secsync, IPFS empirical, OpenPGP / Proton, IPFS whitepaper).

**Phase 2a — Competitive teardown.** Completed at 2026-04-27 15:55. The `research-competitor-teardown` agent produced `competitive-analysis.md` with eight in-depth teardowns (CryptPad, Proton Docs/Sheets, Skiff, Anytype, AFFiNE, ONLYOFFICE, Lit Protocol, Dune Analytics) using a six-section per-competitor template plus a thirteen-dimension comparison matrix. Selection rationale is articulated in the document's opening paragraph; the criterion was analytical relevance to Fileverse's specific architecture choices, not market share alone.

**Phase 2b — Technical architecture.** Completed at 2026-04-27 15:55. The `research-tech-architecture` agent produced `technical-architecture.md` with seven sections: core mechanism design (with two encryption-envelope options and a recommendation), state management across four tiers, settlement/resolution model, infrastructure (with a five-chain comparison and three-provider IPFS comparison), integration layer (twelve external dependencies catalogued), contract/module architecture (six components plus an implied registry interface), and eight attack vectors with severity and mitigation status.

**Phase 2c — Economics and game theory.** Completed at 2026-04-27 15:55. The `research-econ-game-theory` agent produced `economics.md` with eight sections: participation incentive model (two scenarios), equilibrium analysis with quantified collapse conditions, attack/gaming analysis (five vectors with worked examples), fee structure recommendation benchmarked against four direct competitors, unit economics (three scenarios from 25k to 1.5M registered users), minimum viable parameters, applicable academic frameworks (eight), and six failure modes. The closing note explicitly addresses why standard tokenomics framing does not apply.

**Phase 3 — Product spec.** Completed at 2026-04-27 16:01. The `research-product-spec-writer` agent produced `product-spec.md` with nine sections: vision and analyst-observable success criteria, three user personas, MVP/V2/Future feature list, five user flows, technical interface specifications (registry contract, gateway HTTP surface, relay protocol envelopes), eight integration adapters with fallback chains, operational deployment and monitoring spec, nine open conflicts surfaced from Phases 2a–2c with options and recommendations, and twelve analyst-observable Go/No-Go conditions.

**Phase 3.5 — Methodology (this document).** The `research-methodology-writer` agent produces this `methodology.md` to record sources, frameworks, process, limitations, and disclosures. The skill was added to the pipeline on 2026-04-27.

Phases 4 (thesis writing) and 5 (presentation curation) are downstream of this document and not yet produced.

---

## Section 5: Limitations

The report's credibility rests on acknowledging what it does not cover.

- **No primary-source team contact.** No interviews, private correspondence, or off-the-record conversations with Fileverse contributors informed this report. All claims about team intent are inferred from public materials.
- **No on-chain telemetry.** We did not query Etherscan or Gnosisscan for the registry contract addresses (which the public repositories do not disclose), did not measure per-document write frequency on-chain, and did not establish IPFS pin counts directly. Adoption and economic claims rely on indirect signals.
- **No independent code audit.** We read repository READMEs and dependency manifests; we did not perform line-level review of `@fileverse/crypto`, the collaboration-server source, the storage-v2 service, or the dSheets transaction-bearing-cell logic. Claims about wire format, nonce derivation, AAD construction, and per-cell trust boundaries are explicitly flagged as reconstructed in the body sections.
- **No formal simulation.** The unit-economics scenarios are point-estimate arithmetic, not Monte Carlo or sensitivity analysis. Equilibrium-collapse conditions are stated as inequalities without proof of dominance. Attack-vector severity is qualitative (HIGH / MODERATE) rather than quantified expected-loss.
- **No quantitative competitive benchmarking.** We did not run latency, throughput, or feature-coverage benchmarks against CryptPad, Proton, Anytype, or others. Comparison claims rely on vendor-published documentation and independent reporting.
- **English-only sources, US-and-EU-centric framing.** We did not consult Russian, Chinese, Korean, or Japanese-language sources on adjacent productivity tools or storage providers. Regulatory framing implicitly assumes US (FTC, SEC) and EU (GDPR, NIS2) regimes.
- **Skipped categories.** Onchain governance, treasury composition, validator-set analysis, and token distribution sections that would normally appear in a protocol report are absent because Fileverse has no disclosed token. If a token launch is announced, those sections become required and the report would need re-scoping.
- **Local-LLM threat model deferred.** The homepage references on-device language model integration, but no implementation details are public; the threat model is not analyzed beyond noting the deferral.
- **Time-bounded snapshot.** All retrieval is dated 2026-04-27; commit activity, audit completion status, pricing, and personnel can change after that date and are not tracked.

---

## Section 6: Disclosures

**Positions held.** No positions held. The researcher holds no Fileverse-related tokens (none are disclosed by the protocol in any case), no equity in Fileverse or its parent organization, and no positions in the related tokens of competing protocols listed in this report (CryptPad, Proton, Anytype, AFFiNE, ONLYOFFICE, Lit Protocol, Dune, The Graph, Filecoin, or any L1/L2 referenced).

**Conflicts of interest.** None disclosed. The researcher has no prior advisory roles with Fileverse or any direct competitor, has not participated in any Fileverse bounty or grant program, and has no paid relationships with any vendor referenced in this report (Pinata, Filebase, Web3.Storage / Storacha, QuickNode, Alchemy, Safe, ENS, World, Trail of Bits, Dedalo).

**Data cutoff date.** 2026-04-27. No information published or made available after this date is incorporated. Subsequent commits to Fileverse repositories, audit-report releases, pricing changes by pinning providers, or competitor product launches are out of scope.

**Compensation for this research.** None. This research was conducted independently for inclusion in the researcher's public portfolio. No fee, retainer, sponsorship, advance, or in-kind payment was received from Fileverse, any competitor, any vendor cited, or any third party in connection with the production of this report.

---

## Section 7: Acknowledgments

Skipped: no external reviewers contributed to this draft.
