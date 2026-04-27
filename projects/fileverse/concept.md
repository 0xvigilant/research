# Fileverse — Concept Document

**Source:** https://fileverse.io · **Compiled:** 2026-04-27

## What It Is

Fileverse is an open-source, end-to-end encrypted productivity suite designed for privacy-first, decentralized collaboration. Its two primary live products are dDocs ([ddocs.new](https://ddocs.new)), a document editor, and dSheets ([dsheets.new](https://dsheets.new)), a spreadsheet that can read and write onchain data in real time. Both share an AGPL-3.0-licensed middleware layer. Fileverse positions itself as a self-sovereign alternative to Google Workspace and Microsoft 365, where users retain the ability to decrypt and recover their own data without depending on the vendor — a property the team calls "walk-away" data portability ([github.com/fileverse/walk-away](https://github.com/fileverse/walk-away)).

The architecture is local-first: documents live in the browser via IndexedDB and synchronize through an ephemeral, stateless WebSocket server. When persistence or sharing is required, encrypted blobs are pinned to IPFS through a multi-provider storage layer (Pinata, Filebase, Web3.Storage) with blockchain anchoring ([github.com/fileverse/fileverse-storage-v2](https://github.com/fileverse/fileverse-storage-v2)). Smart contracts on Ethereum mainnet (chain ID 1) and Gnosis Chain act as the onchain content registry, mapping file identifiers to IPFS hashes and managing storage quotas ([github.com/fileverse/collaboration-server](https://github.com/fileverse/collaboration-server)).

## What Problem It Addresses

Mainstream productivity suites — Google Workspace, Notion, Microsoft 365, Dropbox — are custodial by design: the vendor holds encryption keys, stores metadata in plaintext, and retains broad rights to analyze content. Users have no technical guarantee their data remains private or portable if the vendor changes terms or shuts down a product; Google has sunset entire product lines with minimal export windows, and Dropbox and Box have both been named in government data requests.

Fileverse frames this as a rights problem: users have "traded their digital rights for the convenience of online workspaces" ([fileverse.io](https://fileverse.io)). Its resolution is technical rather than contractual — encryption that prevents the platform from reading content, local-first storage that removes a single point of failure, and open-source code that allows independent verification of privacy claims. For Web3 users specifically, dSheets addresses the additional gap that no mainstream spreadsheet tool can query live smart contract state or submit transactions natively.

## Mechanism Sketch

**Storage layer.** Files are stored client-side in IndexedDB for offline access. When pinned remotely, the encrypted blob is uploaded to IPFS via the `fileverse-storage-v2` service, which implements multi-provider fallback (Pinata → Filebase → Web3.Storage) and uses UCAN (User Controlled Authorization Networks) bearer tokens for access control. Each stored file is identified by an IPFS content hash; the mapping from application file ID to IPFS hash is committed to a smart contract acting as an onchain content registry.

**Encryption scheme.** The `walk-away` repository documents an RSA + AES hybrid model using the Penumbra encryption toolkit and TweetNaCl. Files are encrypted with a per-file AES key. That key is then wrapped under three independent "locks": a Portal Lock (file key encrypted with the portal's public key), an Owner Lock (file key encrypted with the owner's keypair), and a Link Lock (file key encrypted with a shareable link key). This design allows access recovery through any available lock without requiring server-side key escrow ([github.com/fileverse/walk-away](https://github.com/fileverse/walk-away)).

**Identity.** The platform supports two authentication modes: username-based (vOPRF-ID, a zero-knowledge authentication scheme cited on the homepage) and wallet-based authentication via cryptographic signing. UCAN tokens are issued after authentication and gate IPFS upload operations and collaboration server access. Wallet addresses and ENS names are supported as access-control primitives in dSheets permissions.

**Real-time collaboration.** The `collaboration-server` uses WebSocket connections with Y.js CRDTs for deterministic merge of concurrent edits. The server is stateless: all data passing through it is encrypted client-side and held only ephemerally in memory. Users can self-host the collaboration server; community-hosted server discovery uses the Waku peer-to-peer network (early alpha). The codebase is under a security audit by Dedalo ([github.com/fileverse/collaboration-server](https://github.com/fileverse/collaboration-server)).

**Onchain data (dSheets).** dSheets adds live smart contract querying and transaction submission from spreadsheet cells, integrating the Fortune Sheet library for Excel-compatible formulas (VLOOKUP, INDEX, MATCH). Onchain write operations were added in V0.3.

**Local LLMs.** The homepage references on-device language model integration, not yet documented in public repositories beyond the marketing copy.

## Target Users

Fileverse's stated audience is privacy-conscious individuals and teams seeking Google Workspace or Microsoft 365 equivalents without vendor custody of their data. The homepage lists use cases from legal document management to academic writing and fan fiction — a broad framing that suggests the team has not yet narrowed to a specific wedge segment.

Observable traction is strongest in the Ethereum developer ecosystem: DappCon 2025, ETHSF 2025, and DevCon 2024 all ran community portals on dDocs, indicating conference organizers as early adopters. The dSheets onchain data querying feature targets a more specific cohort: DeFi analysts, protocol teams, and on-chain researchers who want spreadsheet logic applied to live contract state.

## Current State (as of 2026-04-27)

- **Live products.** dDocs ([ddocs.new](https://ddocs.new)) and dSheets ([dsheets.new](https://dsheets.new)) are publicly accessible. The GitHub repositories for both show commits as recently as April 27, 2026, indicating active development. The `fileverse-ddoc` npm package has 204 stars; `fileverse-dsheet` carries 415 commits across its lifespan.
- **On-chain deployments.** The storage and collaboration infrastructure references Ethereum mainnet (chain ID 1) and Gnosis Chain for the content registry smart contracts. No specific contract addresses are disclosed in public documentation. UCAN-based authorization is live in `fileverse-storage-v2`.
- **Token and governance.** No native governance token or tokenomics design is disclosed in any public repository or the homepage. Fileverse does not appear on CoinGecko or CoinMarketCap under any known ticker. The team's ENS address (`fileverse.eth`) is used for publishing; no governance contracts are documented.
- **Backers and partnerships.** No venture funding round or named institutional investors appear in public materials. Fileverse has received visibility from Ethereum Foundation-adjacent programs (DevCon 2024 community portal) and appeared at EthSF and DappCon. No fundraising announcement has been indexed.
- **Security audit.** The `collaboration-server` README notes an ongoing security audit by Dedalo, suggesting the team is approaching a production-readiness threshold.
- **Recent activity (last 90 days).** Both `fileverse-ddoc` and `fileverse-dsheet` show commits through April 27, 2026. The `collaboration-server` carries 74 commits on main; `fortune-sheet` was updated April 3, 2026. No public blog is reachable at `fileverse.io/blog`; Paragraph and Mirror endpoints for the team returned errors during this fetch.

## Open Research Questions

1. **What is the actual user base and retention profile?** GitHub stars and conference deployments are weak proxies. Are there on-chain signals (unique contract callers, IPFS pin counts) that give a more direct read on active usage, and how does this compare to competitors such as Skiff (acquired by Notion, subsequently shut down) and CryptPad?

2. **How does the multi-provider IPFS storage model handle long-term persistence and provider failure?** All three pinning providers (Pinata, Filebase, Web3.Storage) are centralized services with their own operational risks. What happens to documents if all three cease operations — is there a protocol-level pinning incentive, or does persistence fall back to user-managed nodes?

3. **What is the on-chain footprint and gas cost structure?** The content registry contracts on Ethereum mainnet and Gnosis Chain anchor file metadata, but the architecture does not document write frequency, transaction cost, or whether fees are abstracted for end users. The unit economics of a document-heavy workflow matter for adoption viability.

4. **Is vOPRF-ID authentication production-ready, and how does it interact with key recovery?** Zero-knowledge authentication eliminates password exposure but creates new failure modes: if a user loses local key material without a wallet backup, what recovery path exists? The walk-away scheme addresses encrypted content recovery but not the identity layer.

5. **What is the competitive differentiation from CryptPad?** CryptPad has operated an open-source E2EE collaboration suite since 2015 — documents, spreadsheets, presentations, Kanban — with zero-knowledge encryption and self-hosting, without blockchain dependency. How does Fileverse distinguish itself technically and in go-to-market terms?

6. **Does the absence of a token create a sustainable business model, or is a token launch anticipated?** The AGPL-3.0 license and apparent absence of venture funding suggest either a grant-funded or undisclosed revenue model. The UCAN quota system implies metered access infrastructure exists — is there a paid tier, and at what price relative to centralized alternatives?

7. **How battle-tested is the encryption implementation?** The `@fileverse/crypto` library and walk-away recovery scheme are not independently audited in any publicly listed security report. The Dedalo audit of the collaboration-server is noted as in-progress — what is its scope, and does it cover the client-side encryption path?

8. **What is the threat model for the local LLM integration?** On-device language models are consistent with the privacy-first framing, but the homepage cites this capability without implementation details. The threat model differs significantly depending on whether model weights are fetched from a CDN (supply-chain risk) or bundled locally.

## Sources Already Identified

- https://fileverse.io/ — Homepage, value proposition, product overview
- https://github.com/fileverse — Organization page, all public repositories
- https://github.com/fileverse/fileverse-ddoc — dDocs editor: README, architecture, dependencies
- https://github.com/fileverse/fileverse-dsheet — dSheets: README, onchain data querying, commit history
- https://github.com/fileverse/walk-away — Encryption scheme documentation (RSA/AES, lock model, key recovery)
- https://github.com/fileverse/fileverse-storage-v2 — Storage layer: IPFS multi-provider, UCAN authorization, chain ID references
- https://github.com/fileverse/collaboration-server — Real-time collaboration: Y.js CRDTs, WebSocket, Waku, Dedalo audit note
- https://github.com/fileverse/fortune-sheet — Fork of open-source spreadsheet library used in dSheets
- https://ddocs.new — Live dDocs product
- https://dsheets.new — Live dSheets product (referenced as sheets.fileverse.io)
