# Competitor Map — fileverse
_Phase 1 · Generated 2026-04-27_

This map catalogs products, protocols, and projects operating in the same or adjacent space as Fileverse: end-to-end encrypted productivity suites, local-first collaboration tools, decentralized storage-backed document apps, and onchain spreadsheet utilities. Phase 2 agents will decide which entries warrant deeper teardown.

## CryptPad
- Status: Live
- Category: E2EE collaborative productivity suite (zero-knowledge)
- Mechanism: Browser-side encryption with server-blind storage; chainpad CRDT
- URL: https://cryptpad.org/
- Summary: [Open-source zero-knowledge office suite operated since 2014 by XWiki SAS, covering docs, sheets, slides, kanban, whiteboards, and forms](https://cryptpad.org/ "Source: CryptPad, retrieved 2026-04-27"); the closest functional analog to Fileverse without blockchain dependency.

## Proton Docs (and Proton Sheets)
- Status: Live
- Category: E2EE office suite inside an integrated privacy ecosystem
- Mechanism: Client-side encryption keyed to Proton account, hosted on Proton infrastructure
- URL: https://proton.me/drive/docs
- Summary: [Proton launched Docs in mid-2024 and Sheets in December 2025, encrypting both content and metadata such as filenames](https://www.macrumors.com/2025/12/04/proton-sheets-launches-encrypted-spreadsheet/ "Source: MacRumors, 2025-12-04"); custodial in operation but cryptographically blind to user content.

## Skiff (Pages, Drive, Calendar, Mail)
- Status: Defunct
- Category: E2EE workspace, formerly with optional IPFS storage
- Mechanism: Per-document key trees, client-side encryption, opt-in IPFS pinning
- URL: https://skiff.com/
- Summary: [Acquired by Notion in February 2024 and sunset on 2024-08-09, ending one of the few mainstream IPFS-backed E2EE document services](https://cyberinsider.com/skiff-shutting-down-alternatives-to-skiff-mail/ "Source: CyberInsider, 2024-02-12"); a direct historical precedent and cautionary case for Fileverse's category.

## Anytype
- Status: Live
- Category: Local-first encrypted knowledge workspace
- Mechanism: Per-object E2EE, P2P sync via libp2p, optional self-hosted backup nodes
- URL: https://anytype.io/
- Summary: [Object-graph workspace with end-to-end encryption by default and offline-first operation, positioned as a Notion alternative for privacy users](https://doc.anytype.io/anytype-docs/advanced/data-and-security/how-we-keep-your-data-safe "Source: Anytype Docs, retrieved 2026-04-27"); overlaps with Fileverse on the "self-sovereign Notion" framing without an onchain registry.

## AFFiNE
- Status: Live
- Category: Local-first collaborative workspace (open-source)
- Mechanism: Yjs CRDTs with optional self-hosted backend; encrypted local store
- URL: https://affine.pro/
- Summary: [Open-source local-first hybrid of Notion and Miro with self-hosting, marketed for privacy and offline-first usage](https://affine.pro/blog/notion-alternative "Source: AFFiNE Blog, retrieved 2026-04-27"); shares Yjs CRDT lineage with Fileverse's collaboration server.

## Logseq
- Status: Live
- Category: Local-first knowledge base / outliner
- Mechanism: Plain-text Markdown/Org files; optional Logseq Sync with E2EE
- URL: https://logseq.com/
- Summary: [Privacy-first outliner that keeps notes as local plain-text files with optional encrypted sync via Logseq Sync](https://logseq.com/ "Source: Logseq, retrieved 2026-04-27"); single-user-leaning, but shares the local-first ethos and E2EE-as-option pattern.

## AppFlowy
- Status: Live
- Category: Open-source self-hosted Notion alternative
- Mechanism: Self-hosted backend (AppFlowy Cloud) with workspace-level isolation
- URL: https://appflowy.io/
- Summary: [AGPL-licensed Notion alternative emphasizing self-hosting and data ownership rather than client-side E2EE](https://openalternative.co/alternatives/notion "Source: OpenAlternative, retrieved 2026-04-27"); overlaps Fileverse on data-sovereignty narrative, differs on encryption depth.

## Standard Notes
- Status: Live
- Category: E2EE notes (extensible to spreadsheets and editors)
- Mechanism: Client-side AES-256 encryption; subscription plugins extend functionality
- URL: https://standardnotes.com/
- Summary: [Long-running E2EE notes app (acquired by Proton in 2024) with extension marketplace covering markdown, spreadsheets, and tasks](https://standardnotes.com/ "Source: Standard Notes, retrieved 2026-04-27"); narrower scope than Fileverse but a comparable E2EE-document benchmark.

## Etebase / EteSync
- Status: Live
- Category: E2EE backend SDK and sync protocol
- Mechanism: Server-blind encrypted sync for contacts, calendars, tasks, notes; multi-language SDKs
- URL: https://www.etebase.com/
- Summary: [Open-source E2EE backend used by EteSync and Etesync-Notes, with NLnet funding for protocol enhancements](https://nlnet.nl/project/EteSyncEnhancements/ "Source: NLnet, retrieved 2026-04-27"); infrastructure-layer competitor to the encrypted-storage half of Fileverse.

## ONLYOFFICE Docs
- Status: Live
- Category: Self-hosted collaborative office suite
- Mechanism: Server-side document rendering with optional private rooms (E2EE rooms feature)
- URL: https://www.onlyoffice.com/
- Summary: [Open-source MS Office-format-compatible collaborative suite with self-hosting and an "private rooms" E2EE option](https://www.onlyoffice.com/ "Source: ONLYOFFICE, retrieved 2026-04-27"); the dominant non-Google self-hosted office stack and a likely procurement comparator.

## Etherpad
- Status: Live
- Category: Real-time collaborative text editor (open-source)
- Mechanism: Server-mediated real-time editing; no native E2EE
- URL: https://etherpad.org/
- Summary: [Long-standing self-hostable collaborative editor underpinning many privacy-friendly deployments, though not encrypted by default](https://etherpad.org/ "Source: Etherpad Foundation, retrieved 2026-04-27"); a baseline reference for what "open-source collaborative editor" looked like pre-CRDT.

## Collabora Online
- Status: Live
- Category: Self-hosted office suite (LibreOffice-derived)
- Mechanism: Server-side rendering of LibreOffice documents with WOPI integrations
- URL: https://www.collaboraonline.com/
- Summary: [Enterprise-grade self-hosted office suite used by Nextcloud and ownCloud deployments](https://www.collaboraonline.com/ "Source: Collabora, retrieved 2026-04-27"); not E2EE, but the leading self-hosted MS-Office-compatible competitor for organizations wary of Google.

## Cryptee
- Status: Live
- Category: Personal E2EE document and photo vault
- Mechanism: Client-side encryption; self-hostable; subscription tiers
- URL: https://crypt.ee/
- Summary: [Privacy-focused E2EE service for personal documents, journals, and photos with Tor/Onion support](https://crypt.ee/ "Source: Cryptee, retrieved 2026-04-27"); narrower scope than Fileverse but a peer in the privacy-personal-documents niche.

## Arcane Office
- Status: Live (low activity)
- Category: Web3 office suite on Stacks blockchain
- Mechanism: Gaia decentralized storage tied to Stacks identity; client-side encryption
- URL: https://arcane.office/
- Summary: [Decentralized office suite with documents, sheets, photo storage, and contacts using Stacks/Gaia](https://www.saashub.com/cryptpad-alternatives "Source: SaaSHub, retrieved 2026-04-27"); one of the few prior-art examples of a wallet-keyed productivity suite.

## Serenity Notes
- Status: In Development / Alpha
- Category: E2EE collaborative notes (research-grade)
- Mechanism: Yjs CRDTs relayed via the SecSync E2EE architecture
- URL: https://www.serenity.li/
- Summary: [Reference implementation of end-to-end encrypted Yjs collaboration via the SecSync protocol by Nik Graf](https://github.com/nikgraf/secsync "Source: SecSync GitHub, retrieved 2026-04-27"); the closest technical analog to Fileverse's encrypted-CRDT collaboration server.

## dSheets (Fileverse) — own product, included for category map completeness
- Status: Live
- Category: Onchain spreadsheet
- Mechanism: Fortune Sheet engine plus live smart contract reads/writes from cells
- URL: https://dsheets.new
- Summary: [Fileverse's own spreadsheet, the only spreadsheet listed on ethereum.org as an Ethereum app](https://ethereum.org/en/apps/fileverse-dsheets/ "Source: ethereum.org, retrieved 2026-04-27"); included here as the reference point against which onchain-spreadsheet competitors are measured.

## Web3 Sheets (gfx.xyz)
- Status: Live
- Category: Onchain data spreadsheet (Google Sheets add-on)
- Mechanism: Custom formulas that pull onchain data from 12+ chains into Google Sheets
- URL: https://sheets.gfx.xyz/
- Summary: [Google Sheets add-on that imports onchain balances, prices, and contract data with custom functions](https://sheets.gfx.xyz/ "Source: GFX Labs, retrieved 2026-04-27"); a direct read-side competitor to dSheets, but custodial via Google.

## Blockchain For Sheet
- Status: Live
- Category: Onchain write-from-sheet add-on
- Mechanism: Google Sheets add-on that records cell entries to Ethereum
- URL: https://workspace.google.com/marketplace/app/blockchain_for_sheet/413227996607
- Summary: [Free Google Workspace add-on that anchors sheet entries to Ethereum, narrower in scope than dSheets but adjacent](https://workspace.google.com/marketplace/app/blockchain_for_sheet/413227996607 "Source: Google Workspace Marketplace, retrieved 2026-04-27").

## Dune Analytics
- Status: Live
- Category: Onchain SQL analytics platform
- Mechanism: Indexed multi-chain data warehouse queried via SQL; dashboards and alerts
- URL: https://dune.com/
- Summary: [Dominant onchain analytics platform used by DAOs, DeFi teams, and researchers for SQL-based contract analysis](https://dune.com/ "Source: Dune, retrieved 2026-04-27"); not a spreadsheet, but the substitute analyst tool that dSheets must compete with for "I want to see live contract state" workflows.

## The Graph
- Status: Live
- Category: Decentralized indexing and query protocol
- Mechanism: Subgraphs serving GraphQL queries over indexed onchain data
- URL: https://thegraph.com/
- Summary: [Decentralized indexing protocol powering thousands of dApps with structured onchain data via GraphQL](https://thegraph.com/ "Source: The Graph, retrieved 2026-04-27"); infrastructure substitute that competes with dSheets when teams build custom dashboards instead.

## Apillon
- Status: Live
- Category: Web3 storage gateway (Drive/Dropbox alternative recipe)
- Mechanism: Crust Network IPFS pinning with hosting and identity primitives
- URL: https://apillon.io/
- Summary: [Web3 developer platform that publishes a "Google Drive alternative" recipe combining Crust IPFS pinning with hosting](https://blog.apillon.io/apillon-recipe-3-a-web3-alternative-to-google-drive-dropbox-dd764c1556bb/ "Source: Apillon Blog, retrieved 2026-04-27"); storage-layer competitor for the IPFS-pinning portion of Fileverse's stack.

## Ceramic Network (with Lit Protocol)
- Status: Live
- Category: Decentralized event-stream data network
- Mechanism: Signed event streams stored via IPFS/IPLD; Lit Protocol for access control
- URL: https://ceramic.network/
- Summary: [General-purpose decentralized data network that, combined with Lit Protocol's access control, supports private dApp backends](https://ceramic.network/how-it-works "Source: Ceramic, retrieved 2026-04-27"); infrastructure-layer alternative for the encrypted-data-with-onchain-access-control pattern Fileverse implements.

## Lit Protocol
- Status: Live
- Category: Decentralized access control / threshold cryptography network
- Mechanism: Threshold-signed conditional decryption keys gated by onchain conditions
- URL: https://litprotocol.com/
- Summary: [Decentralized network that issues decryption keys based on blockchain credentials, used by privacy-respecting dApps for content gating](https://spark.litprotocol.com/private-data-on-the-open-web/ "Source: Lit Protocol Spark, retrieved 2026-04-27"); architectural alternative to Fileverse's Portal/Owner/Link lock model.

## Internxt Drive
- Status: Live
- Category: E2EE cloud drive (open-source)
- Mechanism: Client-side encryption with file-fragment distribution
- URL: https://internxt.com/
- Summary: [Open-source E2EE drive marketed as a Skiff and Dropbox alternative, with a partial Web3 narrative](https://blog.internxt.com/alternative-to-skiff/ "Source: Internxt Blog, retrieved 2026-04-27"); storage competitor for users who want encrypted file sync without Fileverse's onchain layer.

## Outline
- Status: Live
- Category: Self-hosted team wiki / knowledge base
- Mechanism: Markdown wiki with workspace permissions; self-hostable Docker
- URL: https://www.getoutline.com/
- Summary: [Open-source self-hosted team wiki frequently named alongside Notion alternatives in privacy comparisons](https://www.getoutline.com/ "Source: Outline, retrieved 2026-04-27"); not E2EE, but a procurement substitute for teams choosing self-hosting over client-side encryption.

## Docmost
- Status: Live
- Category: Self-hosted open-source wiki / docs platform
- Mechanism: Docker-deployable workspace with permissioned spaces
- URL: https://docmost.com/
- Summary: [Open-source self-hosted Notion-style wiki, growing share of the self-host segment](https://openalternative.co/alternatives/notion "Source: OpenAlternative, retrieved 2026-04-27"); competes for the same "we will not put company docs in Google" buyer.

## Academic References

- Petru Nicolaescu, Kevin Jahns, Michael Derntl, Ralf Klamma — [_Yjs: A framework for near real-time P2P shared editing on arbitrary data types_](https://link.springer.com/chapter/10.1007/978-3-319-19890-3_55 "Nicolaescu et al., ICWE 2015") — Foundational paper for the Yjs CRDT library that powers the Fileverse collaboration server and several competitors (AFFiNE, JupyterLab, Serenity Notes).
- Martin Kleppmann, Alastair R. Beresford — [_A Conflict-Free Replicated JSON Datatype_](https://arxiv.org/abs/1608.03960 "Kleppmann & Beresford, arXiv:1608.03960, 2016") — Defines the JSON CRDT semantics underpinning rich-document local-first collaboration of the type Fileverse and CryptPad implement.
- Geoffrey Litt, Sarah Lim, Martin Kleppmann, Peter van Hardenberg — [_Peritext: A CRDT for Rich-Text Collaboration_](https://www.inkandswitch.com/peritext/ "Ink & Switch, 2022") — Proposes a rich-text CRDT that preserves authorial intent across formatting concurrent edits, relevant to dDocs's WYSIWYG editing surface.
- Martin Kleppmann, Adam Wiggins, Peter van Hardenberg, Mark McGranaghan — [_Local-first software: You own your data, in spite of the cloud_](https://www.inkandswitch.com/local-first/ "Ink & Switch, 2019") — The canonical local-first essay that names the design pattern Fileverse, Anytype, AFFiNE, and Logseq all claim.
- Nik Graf et al. — [_secsync: Architecture for end-to-end encrypted CRDTs_](https://github.com/nikgraf/secsync "Graf, secsync, 2023") — Open architecture and reference implementation for relaying encrypted Yjs CRDTs through an untrusted server, the closest published prior art to Fileverse's encrypted collaboration server.
- Dennis Trautwein et al. — [_Design and Evaluation of IPFS: A Storage Layer for the Decentralized Web_](https://dl.acm.org/doi/10.1145/3544216.3544232 "Trautwein et al., SIGCOMM 2022") — Empirical analysis of IPFS performance and reliability, directly relevant to Fileverse's multi-provider IPFS persistence strategy.
- Brooke Anderson, Daniel Huigens et al. — [_OpenPGP and the Proton Docs encryption model_](https://proton.me/blog/docs-proton-drive "Proton, 2024-07-03") — Proton's published technical description of E2EE collaborative editing, the closest comparable architecture document to Fileverse's `walk-away` repository.
- Juan Benet — [_IPFS - Content Addressed, Versioned, P2P File System_](https://arxiv.org/abs/1407.3561 "Benet, arXiv:1407.3561, 2014") — Original IPFS whitepaper; foundational for Fileverse's content registry and encrypted-blob pinning model.
