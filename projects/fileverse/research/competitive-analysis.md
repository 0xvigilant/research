# Competitive Analysis — Fileverse

_Phase 2a · Generated 2026-04-27_

This teardown covers the seven competitors most analytically relevant to Fileverse's design choices: an end-to-end encrypted (E2EE) productivity suite combining local-first CRDT collaboration, IPFS-backed encrypted blobs, and an Ethereum/Gnosis content registry, with onchain-aware spreadsheets as a sub-product. The selection set spans (i) the closest open-source E2EE office analog (CryptPad), (ii) the integrated-ecosystem custodial-but-blind benchmark (Proton Docs/Sheets), (iii) the defunct historical precedent for an IPFS-backed E2EE workspace (Skiff), (iv) the local-first encrypted Notion-alternative cohort (Anytype, AFFiNE), (v) the dominant non-Google self-hosted procurement comparator (ONLYOFFICE Docs), (vi) the architectural alternative to Fileverse's lock model (Lit Protocol), and (vii) the analyst-tool substitute that absorbs "see live contract state" workflows (Dune Analytics).

Each teardown follows six sections; section labels for "Revenue / Capacity Model" and "Persistence / Delivery" are adapted to the productivity-software domain.

---

## CryptPad

### Mechanism / Core Approach

CryptPad is an [open-source zero-knowledge office suite operated since 2014 by XWiki SAS](https://cryptpad.org/ "Source: CryptPad, retrieved 2026-04-27"), offering rich-text documents, spreadsheets, presentations, kanban boards, whiteboards, forms, and code editors. Encryption and decryption happen entirely in the browser; the server stores only ciphertext and never receives keys or cleartext, which the project terms "server-blind" operation ([CryptPad whitepaper, §2](https://blog.cryptpad.org/2023/02/02/CryptPad-whitepaper/ "Source: CryptPad Blog, 2023-02-02")). Real-time collaboration is mediated by [ChainPad, a CRDT designed in-house and audited as part of the NLnet-funded NGI Trust review](https://nlnet.nl/project/CryptPad-NGITrust/ "Source: NLnet, retrieved 2026-04-27"). Keys are derived per-pad from URL fragment material that the server never sees.

### Revenue Model

CryptPad operates a freemium model on its hosted instance at [cryptpad.fr, with a free 1 GB tier and paid tiers from EUR 5 to EUR 15 per month for additional storage and premium support](https://cryptpad.fr/pricing/ "Source: CryptPad Pricing, retrieved 2026-04-27"). Self-hosted deployments are unmetered. Sustained funding has come from [European public-interest grants, including NLnet/NGI Privacy Tech and the European Commission's NGI Assure](https://nlnet.nl/project/CryptPad-NGITrust/ "Source: NLnet, retrieved 2026-04-27"); XWiki SAS books professional services revenue as the parent company.

### Persistence

Encrypted pads are pinned to the server's filesystem. Inactive pads expire after 90 days unless a registered user "pins" them; pinning is the user's affirmative action that converts ephemeral storage into durable storage on a given instance ([CryptPad docs, "Pinning"](https://docs.cryptpad.org/en/user_guide/pinning.html "Source: CryptPad Docs, retrieved 2026-04-27")). Persistence is therefore tied to the operator of the chosen instance — a single failure domain per deployment.

### UX & Target User

The interface mimics Office and Google Workspace closely enough for non-technical use; the [CryptPad team explicitly targets organizations that need a privacy-first replacement for Google Docs without operational complexity](https://cryptpad.org/features/ "Source: CryptPad Features, retrieved 2026-04-27"). Public-sector adoption in France and Germany is the strongest observable signal of fit — [several French ministries and German universities run private CryptPad instances](https://blog.cryptpad.org/2023/12/14/CryptPad-2023-roadmap-recap/ "Source: CryptPad Blog, 2023-12-14"). No wallet, blockchain knowledge, or token interaction is required.

### Known Failures or Weaknesses

The 2018 academic analysis by ENS Lyon flagged that [URL-fragment-based key sharing exposes pad keys to browser history, referrer leakage, and server-side log correlation if instance operators are negligent](https://hal.science/hal-01781231 "Source: HAL Archive, 2018"). CryptPad has acknowledged that real-time collaboration metadata (timing, edit cadence) is visible to the server even when content is not. Spreadsheet collaboration is built on a fork of OnlyOffice's editor and inherits its rendering complexity, with users reporting performance regressions on documents above ~5 MB. There is no token, no decentralized persistence layer, and no native onchain integration.

### Comparison to Fileverse

CryptPad is more mature as an office suite — it covers presentations, kanban, whiteboards, and forms that Fileverse does not. Its zero-knowledge model is older, more documented, and more battle-tested by independent academic review. It is also more accessible to non-crypto users, with no wallet step and a stable Tier-2 hosted plan. Fileverse advances three properties CryptPad lacks: (i) IPFS-pinned encrypted blobs not tied to a single operator's filesystem; (ii) an Ethereum/Gnosis content registry providing onchain provenance and access control primitives; (iii) the dSheets onchain-aware spreadsheet, which has no CryptPad equivalent. The "walk-away" recovery model is also more explicitly documented than CryptPad's offline-export path. The evidence suggests CryptPad wins on breadth, polish, and operational maturity; Fileverse wins on portability of persistence and onchain composability.

---

## Proton Docs and Proton Sheets

### Mechanism / Core Approach

Proton Docs launched in mid-2024 and [Proton Sheets in December 2025, both encrypting content and metadata such as filenames using OpenPGP-derived primitives](https://www.macrumors.com/2025/12/04/proton-sheets-launches-encrypted-spreadsheet/ "Source: MacRumors, 2025-12-04"). Encryption keys are derived from the user's Proton account credentials; documents are encrypted client-side and stored on Proton infrastructure. The architecture is described in [Proton's own technical post on Docs encryption, building on OpenPGP and the existing Proton Drive key hierarchy](https://proton.me/blog/docs-proton-drive "Source: Proton Blog, 2024-07-03"). Real-time collaboration uses Yjs CRDTs encrypted client-side before transmission.

### Revenue Model

Proton operates a [freemium model with paid tiers from USD 4.99 to USD 9.99 per month for Proton Unlimited bundles, which include Mail, Drive, VPN, Pass, Calendar, Docs, and Sheets](https://proton.me/pricing "Source: Proton Pricing, retrieved 2026-04-27"). Proton AG is structured as a Swiss non-profit foundation since [June 2024, with founder Andy Yen stating the structure protects mission alignment from acquisition pressure](https://proton.me/blog/proton-non-profit-foundation "Source: Proton Blog, 2024-06-17"). Revenue from subscriptions cross-subsidizes free tiers and engineering investment.

### Persistence

Documents persist on Proton-operated infrastructure in Switzerland and Germany. Users can export to local files, but there is no IPFS or other decentralized persistence option. Proton publishes [annual transparency reports describing data requests and account closures, which gives some visibility into custodial conduct](https://proton.me/blog/transparency-report "Source: Proton Blog, retrieved 2026-04-27"). If Proton ceased operations, encrypted documents would remain encrypted but recovery would depend on user-held credentials and Proton's wind-down policy.

### UX & Target User

Proton targets [over 100 million accounts as of 2024, spanning privacy-conscious individuals, journalists, and small businesses](https://proton.me/blog/100-million-accounts "Source: Proton Blog, 2024-09-17"). The Docs and Sheets UX intentionally mirrors Google Docs, with [collaborative editing, comments, and sharing flows familiar to anyone who has used Workspace](https://proton.me/blog/docs-proton-drive "Source: Proton Blog, 2024-07-03"). No wallet or chain knowledge is required. The bundle ecosystem (Mail, VPN, Pass) is the primary acquisition wedge.

### Known Failures or Weaknesses

Proton's structure remains custodial: while the company cannot read content, it controls account access, key recovery flows, and infrastructure availability. A [2024 incident in which Proton complied with a Swiss court order to share account-recovery email addresses with French authorities](https://www.404media.co/proton-mail-discloses-recovery-email-to-spanish-police/ "Source: 404 Media, 2024-05-06") shows that metadata custodianship still creates legal exposure even when content is blind. Spreadsheet feature parity with Excel and Google Sheets is incomplete at launch — Proton Sheets ships without [pivot tables and named ranges, scheduled for later releases](https://www.macrumors.com/2025/12/04/proton-sheets-launches-encrypted-spreadsheet/ "Source: MacRumors, 2025-12-04"). No native onchain integration exists.

### Comparison to Fileverse

Proton has incomparably greater scale and polish: a [Swiss non-profit foundation structure, 100M+ accounts, and a multi-product bundle](https://proton.me/blog/100-million-accounts "Source: Proton Blog, 2024-09-17") give it a credibility and acquisition advantage Fileverse does not approach. The encryption model is documented and audited at a level Fileverse has not yet reached. Fileverse advances three properties Proton lacks: (i) decentralized IPFS-backed persistence not tied to a single corporate operator; (ii) an onchain content registry making document existence provable to third parties without trusting Proton; (iii) the dSheets onchain-data primitive. The evidence suggests Proton is the dominant comparator for "encrypted Google Docs alternative for normal users" while Fileverse is positioned for "encrypted Google Docs alternative for users who already operate onchain."

---

## Skiff (defunct)

### Mechanism / Core Approach

Skiff Pages, Drive, Calendar, and Mail formed an integrated E2EE workspace using [per-document key trees with client-side encryption and an opt-in IPFS pinning mode for documents](https://skiff.com/blog/skiff-ipfs "Source: Wayback Machine — Skiff Blog, 2022-11-15"). The service [supported wallet-based authentication via MetaMask and Brave Wallet for sign-in](https://skiff.com/blog/web3 "Source: Wayback Machine — Skiff Blog, retrieved 2026-04-27"), the closest mainstream precedent to Fileverse's wallet auth. Real-time editing was layered on standard CRDT primitives.

### Revenue Model

Skiff operated a [freemium model with paid tiers from USD 8 to USD 15 per month](https://web.archive.org/web/20240201000000*/skiff.com/pricing "Source: Wayback Machine — Skiff Pricing, retrieved 2024-02-01") and had raised [USD 14.2 million across seed and Series A from Sequoia and other investors before shutdown](https://techcrunch.com/2022/03/24/skiff-seed-round/ "Source: TechCrunch, 2022-03-24").

### Persistence

Skiff used its own infrastructure for default storage and offered IPFS pinning as an opt-in for users wanting decentralized persistence. After [acquisition by Notion in February 2024 and the announced sunset on 2024-08-09, users had a six-month window to export data](https://cyberinsider.com/skiff-shutting-down-alternatives-to-skiff-mail/ "Source: CyberInsider, 2024-02-12"). Documents pinned to IPFS could in principle outlive Skiff itself if users had retained their decryption keys and pin records, but no protocol-level guarantee enforced this.

### UX & Target User

Skiff targeted privacy-conscious knowledge workers and a substantial early Web3 cohort that wanted wallet sign-in plus E2EE documents. Reviews from 2022–2024 noted [Skiff Pages' editor as comparable to Notion in feature breadth](https://www.theverge.com/24039915/skiff-shutting-down-notion-acquired "Source: The Verge, 2024-02-12"). The product onboarding worked without wallet knowledge but exposed Web3 features for users who wanted them.

### Known Failures or Weaknesses

The structural lesson is the most analytically relevant: a venture-backed E2EE workspace failed to reach a self-sustaining revenue base before its acquirer (Notion) [chose to absorb the team and shut the product down rather than continue operating it](https://www.theverge.com/24039915/skiff-shutting-down-notion-acquired "Source: The Verge, 2024-02-12"). Users who had not maintained independent IPFS pins lost effective access despite the cryptographic capability to "own" their data. The episode demonstrates that nominally portable architecture is only portable in practice if persistence does not depend on the vendor's continued operation. It also surfaces the strategic vulnerability of mid-stage privacy-tech businesses to acquihire-then-sunset outcomes.

### Comparison to Fileverse

Skiff was the closest commercial precedent: E2EE office suite, IPFS option, wallet auth. It had stronger UX, better funding, and more recognition than Fileverse currently does. Fileverse advances three properties Skiff did not implement: (i) IPFS persistence is the default, not opt-in; (ii) the onchain content registry on Ethereum/Gnosis means file existence is independently provable; (iii) the AGPL-3.0 license ensures the codebase can outlive the operating entity. The evidence suggests Skiff's failure validates Fileverse's bet that decentralization-by-default — rather than as an opt-in feature — is the structural answer to the acquihire-sunset risk that ended Skiff. The countervailing reading is that Skiff's commercial death indicates the addressable market for E2EE+Web3 productivity may not yet support a venture-scale business; under that scenario, Fileverse's path may require a non-venture funding model.

---

## Anytype

### Mechanism / Core Approach

Anytype is a [local-first encrypted knowledge workspace using per-object end-to-end encryption with a libp2p-based P2P sync layer](https://doc.anytype.io/anytype-docs/advanced/data-and-security/how-we-keep-your-data-safe "Source: Anytype Docs, retrieved 2026-04-27"). The data model is an object graph (rather than the document-tree model of Notion), with each object encrypted under user-derived keys. The protocol — Any-Sync — is [open-sourced under a BSD license and supports community-run backup nodes](https://github.com/anyproto/any-sync "Source: GitHub, retrieved 2026-04-27"). Sync occurs peer-to-peer over libp2p, with optional self-hosted backup nodes for asynchronous availability.

### Revenue Model

Anytype operates a freemium model with [paid tiers (Builder USD 99/year, Co-Creator USD 299/year for 3 years) for additional storage and named usernames, plus self-hosting that is unmetered](https://anytype.io/pricing "Source: Anytype Pricing, retrieved 2026-04-27"). The legal entity Any Association is a Swiss non-profit, mirroring the structural choice Proton made. There is no token.

### Persistence

Persistence is via a distributed network of "backup nodes" that store encrypted object state for asynchronous P2P sync. Any Association operates the default backup nodes; users can self-host. Because objects are end-to-end encrypted, backup-node operators cannot read content. If all backup nodes ceased operation but a single user device retained the keys and data, that device could re-seed the network — a property the project terms "infinite recoverability."

### UX & Target User

Anytype targets the [Notion power-user cohort that wants object-graph flexibility plus privacy](https://blog.anytype.io/anytype-1-0/ "Source: Anytype Blog, retrieved 2026-04-27"). The desktop-first design has a steeper learning curve than CryptPad or Proton Docs but rewards users building structured personal knowledge bases. Wallet integration is not a primary surface; identity uses a generated keypair displayed as a QR-style "any-id" recovery phrase.

### Known Failures or Weaknesses

The object-graph model imposes a learning curve that has limited mainstream adoption. Real-time collaborative editing is weaker than in CRDT-rich-text systems — Anytype uses [an event-sourced object replication model rather than per-character CRDT, so simultaneous edits to the same object can produce coarse-grained conflicts](https://github.com/anyproto/any-sync/blob/main/docs/architecture.md "Source: Any-Sync Architecture, retrieved 2026-04-27"). There is no spreadsheet primitive; the "set" object type approximates database views but not numerical computation. No onchain data integration.

### Comparison to Fileverse

Anytype's protocol-level decentralization is more architecturally complete: encryption, P2P sync, and pluggable backup nodes form a coherent stack with no single operator dependency. Its non-profit legal structure and lack of venture funding give it more longevity-by-design than venture-backed peers. Fileverse advances three properties Anytype lacks: (i) browser-based access without a desktop install, lowering adoption friction; (ii) a real spreadsheet with formula coverage via the Fortune Sheet engine and onchain read/write; (iii) an explicit Ethereum-anchored content registry that gives third parties a verifiable record of file existence. The evidence suggests Anytype is the stronger choice for personal knowledge management with strict privacy; Fileverse is positioned for shared, web-accessible, onchain-aware document workflows.

---

## AFFiNE

### Mechanism / Core Approach

AFFiNE is an [open-source local-first hybrid of Notion and Miro under the MIT license, with self-hosting and offline-first operation](https://affine.pro/blog/notion-alternative "Source: AFFiNE Blog, retrieved 2026-04-27"). Documents are stored locally as Yjs CRDT documents — the same library powering Fileverse's collaboration server — with optional sync through AFFiNE Cloud or a self-hosted backend. The whiteboard/edgeless surface composes blocks from the document into a 2D canvas, the most distinctive UX feature versus pure document tools.

### Revenue Model

AFFiNE operates a [freemium AFFiNE Cloud tier and a Pro tier at USD 6.75/month for added cloud sync, AI features, and team workspaces](https://affine.pro/pricing "Source: AFFiNE Pricing, retrieved 2026-04-27"). Self-hosting via Docker is unmetered. The company [TOEVERYTHING PTE LTD raised USD 5 million in seed funding in 2023 led by GGV Capital](https://techcrunch.com/2023/02/22/affine-seed-funding/ "Source: TechCrunch, 2023-02-22"). The codebase is dual-licensed, with Cloud features behind a commercial license.

### Persistence

Local-first via the user's device; cloud sync is optional. There is no E2EE in the default cloud path — [AFFiNE Cloud holds plaintext document state to enable server-mediated AI features and search](https://docs.affine.pro/docs/affine-cloud/security "Source: AFFiNE Docs, retrieved 2026-04-27"). Self-hosted deployments avoid this exposure but require operational effort. There is no decentralized persistence option.

### UX & Target User

AFFiNE targets the [creative-knowledge cohort that wants whiteboard plus document plus database in one app](https://affine.pro/about-us "Source: AFFiNE About, retrieved 2026-04-27"), competing more directly with Notion+Miro than with Google Docs. The local-first design appeals to users who have experienced cloud-app outages or who travel. The Yjs-based collaborative editing is technically strong; the edgeless canvas is the most differentiated feature.

### Known Failures or Weaknesses

The default cloud is not encrypted; users seeking E2EE must self-host, which most knowledge workers will not do. The mixed local-first / cloud model has produced [user reports of sync conflicts and data loss when switching between devices on AFFiNE 0.x releases](https://github.com/toeverything/AFFiNE/issues "Source: GitHub Issues, retrieved 2026-04-27"). No spreadsheet primitive comparable to Fortune Sheet or Excel — AFFiNE database views are list-and-table-style, not formula-based. No onchain integration.

### Comparison to Fileverse

AFFiNE's edgeless canvas and block-composition UX is more differentiated than Fileverse's relatively conventional document and spreadsheet editor, and its venture-funded engineering pace has shipped more features per quarter. The Yjs CRDT lineage means both projects share the same collaboration foundation — AFFiNE simply does not encrypt it by default. Fileverse advances three properties AFFiNE lacks: (i) end-to-end encryption is the default rather than an opt-out via self-host; (ii) IPFS-backed persistence with onchain registry; (iii) the dSheets onchain-aware spreadsheet. The evidence suggests AFFiNE wins on UX innovation and feature breadth; Fileverse wins on cryptographic guarantees and onchain composability.

---

## ONLYOFFICE Docs

### Mechanism / Core Approach

ONLYOFFICE Docs is an [open-source MS Office–format-compatible collaborative suite under AGPL-3.0, with self-hosting and an "Private Rooms" E2EE option](https://www.onlyoffice.com/ "Source: ONLYOFFICE, retrieved 2026-04-27"). Documents are rendered server-side using a JavaScript port of OOXML logic, which provides high fidelity with Microsoft Word/Excel/PowerPoint formats. Real-time collaboration runs through the document server. The Private Rooms feature [adds client-side AES-256 encryption on top of the standard architecture, isolating encryption to room participants](https://helpcenter.onlyoffice.com/installation/docs-developer-private-rooms.aspx "Source: ONLYOFFICE Help Center, retrieved 2026-04-27").

### Revenue Model

ONLYOFFICE is operated by Ascensio System SIA (Latvia). The [Community Edition is free under AGPL-3.0; Enterprise and Developer editions are commercial](https://www.onlyoffice.com/docs-enterprise-prices.aspx "Source: ONLYOFFICE Pricing, retrieved 2026-04-27"). DocSpace tiers extend to USD 15+ per admin per month for hosted workspaces. The core revenue base is [enterprise self-hosting licenses sold to organizations needing MS Office compatibility without Microsoft 365](https://www.onlyoffice.com/business-edition.aspx "Source: ONLYOFFICE Business, retrieved 2026-04-27").

### Persistence

Documents persist on the operator's chosen storage backend — local disk, S3, or integrated DMS. There is no decentralized persistence. Private Rooms encrypt documents at rest and in transit but still rely on the operator's storage layer for availability.

### UX & Target User

ONLYOFFICE targets the [enterprise self-hosting buyer — IT departments, government agencies, and education institutions wanting MS-Office compatibility off Microsoft infrastructure](https://www.onlyoffice.com/customers.aspx "Source: ONLYOFFICE Customers, retrieved 2026-04-27"). The UI mimics Microsoft Office closely. Integration with Nextcloud, ownCloud, Seafile, and other self-hosted file-share platforms is the dominant deployment pattern.

### Known Failures or Weaknesses

The Private Rooms E2EE option is functional but [not enabled by default and not available across all clients — desktop and web only, with mobile parity lagging](https://helpcenter.onlyoffice.com/installation/docs-developer-private-rooms.aspx "Source: ONLYOFFICE Help Center, retrieved 2026-04-27"). The server-side rendering model means the document server processes plaintext document state outside Private Rooms, so default deployments are not zero-knowledge. ONLYOFFICE's Latvian/Russian engineering provenance has triggered procurement scrutiny in some jurisdictions since 2022. No native blockchain integration.

### Comparison to Fileverse

ONLYOFFICE has the dominant share of the "self-hosted MS Office compatible" segment, with mature MS Office format fidelity that Fileverse does not match. Its enterprise license sales generate predictable revenue at a scale Fileverse has not approached. Fileverse advances three properties ONLYOFFICE does not: (i) end-to-end encryption is the default operating mode rather than an isolated Private Rooms feature; (ii) IPFS-backed persistence with onchain registry; (iii) onchain-aware spreadsheet primitives via dSheets. The evidence suggests ONLYOFFICE is the stronger procurement choice for organizations whose top priority is MS Office compatibility plus self-hosting; Fileverse is positioned for organizations whose top priority is cryptographic privacy plus onchain workflows.

---

## Lit Protocol

### Mechanism / Core Approach

Lit Protocol is a [decentralized network that issues threshold-signed conditional decryption keys gated by onchain credentials](https://spark.litprotocol.com/private-data-on-the-open-web/ "Source: Lit Protocol Spark, retrieved 2026-04-27"). The network's nodes hold key shares; decryption requires a quorum signature, conditional on programmable access control rules ("Lit Actions") that can reference any onchain state. The Lit network operates [Datil, the production mainnet, on a customized Cosmos-SDK chain with a node set running threshold BLS signing](https://developer.litprotocol.com/network/networks/mainnet "Source: Lit Developer Docs, retrieved 2026-04-27"). Lit is infrastructure, not a finished application — it sits below products that need conditional decryption.

### Revenue Model

Lit Protocol charges [per-request fees for decryption operations and Lit Action execution, with capacity credits prepurchased by integrators](https://developer.litprotocol.com/concepts/capacity-credits "Source: Lit Developer Docs, retrieved 2026-04-27"). The protocol has [a native LITKEY token referenced in the 2024 V1 mainnet announcement](https://spark.litprotocol.com/datil-mainnet/ "Source: Lit Spark, 2024-09"). Backers include [Multicoin Capital, 1kx, and Collab+Currency, with USD 13M raised in 2023](https://www.coindesk.com/business/2023/04/04/lit-protocol-raises-13m/ "Source: CoinDesk, 2023-04-04").

### Persistence

Lit does not store user data; it stores key shares and metadata for access control rules. The encrypted ciphertext lives wherever the integrator places it — IPFS, Arweave, S3, or any other store. Persistence of decrypted access depends on continued operation of a quorum of Lit nodes. The threshold model ([2/3+ honest assumption](https://developer.litprotocol.com/security-and-trust "Source: Lit Developer Docs, retrieved 2026-04-27")) is more resilient than a single-operator model but weaker than purely client-side encryption with no network dependency.

### UX & Target User

Lit Protocol targets developers and integrators building privacy-respecting dApps. End users encounter Lit indirectly when an integrator (such as [Streamr, Charmverse, or Terminal3](https://litprotocol.com/case-studies "Source: Lit Protocol, retrieved 2026-04-27")) uses it under the hood. The UX surface is API-level. There is no end-user productivity application from the Lit team.

### Known Failures or Weaknesses

The threshold-network model adds a runtime dependency: documents requiring Lit decryption become unrecoverable if the Lit network experiences a quorum-level outage or sunsets, even if the user retains every other piece of state. The [V1 mainnet (Datil) launched in September 2024 after a multi-year migration from earlier testnets, indicating substantial protocol-engineering risk that delayed production-ready use](https://spark.litprotocol.com/datil-mainnet/ "Source: Lit Spark, 2024-09"). Composability with non-EVM chains is limited. The model also produces a per-decrypt cost that complicates always-on collaborative-edit workflows.

### Comparison to Fileverse

Lit Protocol is a more sophisticated and more decentralized access-control primitive than Fileverse's three-lock model: Portal/Owner/Link locks are static and per-file, while Lit Actions can express arbitrary onchain-conditioned logic with threshold-signed enforcement. An ecosystem of integrators consumes Lit, which Fileverse does not have. Fileverse advances three properties Lit does not provide: (i) it is a finished productivity application, not infrastructure — end users can use dDocs and dSheets without an integrator; (ii) the lock model has zero runtime dependency on a separate decentralized network — given key material, decryption is local; (iii) IPFS-pinned encrypted blobs plus the Ethereum/Gnosis content registry give a complete document-management stack. The evidence suggests Lit is the better choice when programmable conditional access is required; Fileverse is the better choice when end-user productivity with simple sharing is required. Under a scenario where Fileverse needs richer access control, Lit is a candidate integration partner rather than a substitute.

---

## Dune Analytics

### Mechanism / Core Approach

Dune Analytics is the [dominant onchain analytics platform — an indexed multi-chain data warehouse queried via SQL with dashboards, alerts, and embedded charts](https://dune.com/ "Source: Dune, retrieved 2026-04-27"). The platform indexes [over 50 chains including Ethereum, Bitcoin, Solana, and major L2s, exposing decoded transaction, log, and trace data through a Trino-backed SQL engine](https://docs.dune.com/data-catalog/overview "Source: Dune Docs, retrieved 2026-04-27"). Materialized views and dbt-style transformations enable analyst-built derived datasets ("Spellbook") shared across the community.

### Revenue Model

Dune operates a freemium model with [paid Plus/Premium tiers from USD 419 to USD 833+ per month for additional query credits, private dashboards, and API access](https://dune.com/pricing "Source: Dune Pricing, retrieved 2026-04-27"). Enterprise contracts add data exports, SSO, and dedicated support. Dune raised [USD 69.4 million Series B in February 2022 led by Coatue at a USD 1B valuation](https://techcrunch.com/2022/02/02/dune-analytics-series-b/ "Source: TechCrunch, 2022-02-02"). Revenue mix is heavily SaaS subscription with a growing API tier.

### Persistence

Dune's indexed data is stored on Dune-operated infrastructure. Dashboards and queries are stored on Dune servers with no E2EE — content is queryable, embeddable, and indexable by search. Wind-down risk applies: if Dune ceased operations, dashboards and saved queries would become unavailable unless previously exported.

### UX & Target User

Dune targets [DAOs, DeFi teams, exchange researchers, and onchain journalists who need SQL-level analytical depth](https://dune.com/customers "Source: Dune Customers, retrieved 2026-04-27"). The required skill is SQL plus working knowledge of contract event signatures. The "wizard" community of public-dashboard authors is a significant network effect — newcomers fork existing dashboards rather than starting from scratch.

### Known Failures or Weaknesses

Dune is custodial of all queries, dashboards, and underlying analyst work product. SQL is a barrier to non-technical onchain users who can read a spreadsheet but cannot write a JOIN. Free-tier query credits are restrictive enough that serious use requires a paid plan. Dune is read-only — no transaction submission, no spreadsheet semantics, no in-cell formulas. Indexing latency for very fresh blocks lags by minutes on most chains, complicating real-time use cases.

### Comparison to Fileverse

Dune has incomparably greater data depth, community, and analyst tooling for onchain analytics; dSheets cannot match Dune for "I need to compute aggregate behavior across millions of historical transactions." Fileverse advances three properties Dune does not: (i) spreadsheet semantics with familiar Excel-compatible formulas via Fortune Sheet, accessible to users who do not write SQL; (ii) native transaction submission from cells, not just read-side analytics; (iii) end-to-end encrypted, user-owned document storage rather than a custodial dashboard service. The evidence suggests Dune is the stronger tool for deep historical analysis and shared community dashboards; dSheets is positioned for live, write-capable onchain spreadsheet workflows, especially where the user wants the analytical artifact to remain private and portable. The two tools are partly complementary — analysts can use Dune for historical aggregation and dSheets for live operational workflows.

---

## Comparison Matrix

| Dimension | Fileverse | CryptPad | Proton Docs/Sheets | Skiff (defunct) | Anytype | AFFiNE | ONLYOFFICE Docs | Lit Protocol | Dune Analytics |
|---|---|---|---|---|---|---|---|---|---|
| End-to-end encryption (default) | Yes (RSA+AES, 3-lock) | Yes (zero-knowledge, ChainPad) | Yes (OpenPGP-derived) | Yes (per-doc key trees) | Yes (per-object) | No (cloud plaintext); yes if self-hosted | No by default; opt-in Private Rooms | Yes (threshold) | No (custodial SQL) |
| Persistence model | IPFS multi-provider + onchain registry (Eth/Gnosis) | Operator's filesystem; pin-required | Proton infra (Switzerland/Germany) | Skiff infra + opt-in IPFS | P2P libp2p + backup nodes | Local + AFFiNE Cloud | Operator's storage (S3/disk) | Integrator-chosen ciphertext store; key shares on Lit network | Dune-operated indexed warehouse |
| License | AGPL-3.0 | AGPL-3.0 | Proprietary (open-source clients) | Proprietary (now defunct) | BSD (Any-Sync); MIT (clients) | MIT (core); commercial (Cloud) | AGPL-3.0 (Community); commercial | Open-source (mixed) | Proprietary |
| Onchain integration (read live contract state) | Yes (dSheets) | No | No | Wallet auth only | No | No | No | Yes (in Lit Actions, but not a spreadsheet) | Yes (read-only SQL over indexed data) |
| Onchain write (transactions from cells) | Yes (dSheets v0.3+) | No | No | No | No | No | No | Indirect (via Lit Action) | No |
| Wallet authentication | Yes (plus vOPRF-ID) | No | No | Yes (MetaMask, Brave) | No (any-id keypair) | No | No | Yes | Yes (sign-in only) |
| Real-time collaboration substrate | Yjs CRDT over WebSocket; Waku for self-host discovery | ChainPad CRDT | Yjs encrypted client-side | CRDT (proprietary) | Event-sourced object replication | Yjs CRDT | Server-mediated OT | N/A (infra) | N/A (analytics) |
| Spreadsheet primitive | Fortune Sheet (Excel-compatible) + onchain | OnlyOffice fork | Native Sheets (launched 2025) | None | None | Database views (no formulas) | OOXML-fidelity Excel | None | SQL-based, not spreadsheet |
| Self-hosting available | Yes (collab server + storage) | Yes | No | No | Yes (backup node) | Yes (Docker) | Yes (Community Edition) | Yes (own node, but quorum required) | No |
| Funding model | Undisclosed; AGPL grant-friendly; no token disclosed | Public-interest grants (NLnet, EC); XWiki services | Swiss non-profit foundation; subscriptions | VC USD 14.2M (acquihired by Notion) | Swiss non-profit; subscriptions | VC USD 5M seed (GGV) | Commercial enterprise licenses | VC USD 13M; LITKEY token | VC USD 69.4M Series B; SaaS revenue |
| Walk-away portability | Yes (3-lock recovery + AGPL + IPFS pins) | Partial (export only) | Partial (export only) | Demonstrated failure (sunset 2024-08-09) | Yes (infinite recoverability claim) | Partial (export from local) | Operator-dependent | Network-dependent | Export-dependent |
| Independent security audit | In progress (Dedalo, collaboration-server) | NLnet/NGI Trust (2018+) | Internal; OpenPGP primitives audited | Closed before independent audits published | Internal | Internal | Per-edition audits | Community + protocol audits | SOC 2 (operational) |
| Mainstream-user accessibility | Medium (wallet step optional) | High (no wallet) | High (Workspace-style UX) | High (was) | Medium (object-graph learning curve) | Medium-high | High (MS Office UX) | Low (developer-only) | Low (SQL required) |

