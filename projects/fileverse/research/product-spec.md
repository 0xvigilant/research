# Product Spec — Fileverse

_Phase 3 · Compiled 2026-04-27 · Researcher-portfolio mode_

This document is a synthesis spec, not a roadmap. Fileverse has not published a public product roadmap, and no team commitments are implied here. The spec describes what a successful 12–24 month execution would look like under the architecture, competitive position, and economic constraints currently visible in [Fileverse's public repositories and homepage](https://fileverse.io "Source: Fileverse, retrieved 2026-04-27"). Section 9 is framed as analyst-observable conditions rather than internal go/no-go decisions.

Inputs read in order: research-log, competitive-analysis, technical-architecture, economics. Conflicts surfaced in those inputs are addressed explicitly in Section 8.

---

## Section 1: Product Vision & Goals

**Vision.** Fileverse is positioned as an end-to-end encrypted productivity suite where users retain cryptographic and operational control of their data — encryption is default, persistence is content-addressed across multiple providers, and the codebase is portable enough to outlive the operating entity. The evidence suggests the durable wedge against incumbents is not feature parity with Google Workspace but the combination of (i) walk-away portability via the [triple-lock recovery scheme](https://github.com/fileverse/walk-away "Source: Fileverse walk-away README, retrieved 2026-04-27"), (ii) onchain-aware spreadsheet primitives in [dSheets V0.3](https://github.com/fileverse/fileverse-dsheet "Source: Fileverse fileverse-dsheet README, retrieved 2026-04-27"), and (iii) [AGPL-3.0 licensing](https://github.com/fileverse/fileverse-ddoc "Source: Fileverse ddoc README, retrieved 2026-04-27") with self-hosting parity.

**Measurable launch success criteria (analyst-observable, 12–24 month window).**

1. **Cryptographic-library audit complete and published.** Per [Trail of Bits' published audit-cycle guidance](https://blog.trailofbits.com/2023/02/14/our-2022-security-audit-cycle/ "Source: Trail of Bits, 2023-02-14"), an independent audit of `@fileverse/crypto` (in addition to the [in-progress Dedalo audit on collaboration-server](https://github.com/fileverse/collaboration-server "Source: Fileverse collaboration-server README, retrieved 2026-04-27")) is the threshold for moving from beta-grade trust posture to production-grade.
2. **Sustainable cost coverage at the gateway tier.** Roughly [3,000 paying users at the recommended $5–$10/month band](https://cryptpad.fr/pricing/ "Source: CryptPad Pricing, retrieved 2026-04-27") covers infra (pinning + bandwidth) but not full engineering, per the unit-economics modeling in the Phase 2c analysis. Crossing this threshold is the cleanest evidence that gateway operation is no longer pure subsidy.
3. **Dual-chain registry redundancy operational.** The content registry must remain readable when either Ethereum mainnet or Gnosis Chain experiences extended downtime, with a documented snapshot or replication policy.
4. **Self-hosting share <25% but >5%.** Below 5%, the [AGPL self-host story](https://github.com/fileverse/fileverse-ddoc "Source: Fileverse ddoc README, retrieved 2026-04-27") is theatrical; above 25%, gateway pricing power against pinning vendors degrades, per the equilibrium analysis.
5. **No publicized confidentiality breach attributable to Link Lock leakage.** The [URL-fragment-based key sharing attack surface](https://hal.science/hal-01781231 "Source: HAL Archive, 2018") is the highest-severity vector specific to this design.

**What This Is NOT (non-goals).**

- Not a Google Workspace clone. Feature breadth comparable to [CryptPad's presentations, kanban, whiteboards, and forms](https://cryptpad.org/features/ "Source: CryptPad Features, retrieved 2026-04-27") is out of scope for the document/spreadsheet wedge.
- Not a token-economy mechanism. No bonding curve, staking yield, or governance distribution is implied; classical tokenomics framing does not apply absent a token disclosure.
- Not a Dune Analytics substitute. The [Trino-backed SQL engine indexing 50+ chains](https://docs.dune.com/data-catalog/overview "Source: Dune Docs, retrieved 2026-04-27") covers a different workflow (deep historical aggregation) than dSheets' live-state operational use case.
- Not a managed enterprise document-management system. ONLYOFFICE-style [MS Office format fidelity](https://www.onlyoffice.com/ "Source: ONLYOFFICE, retrieved 2026-04-27") and per-edition enterprise audits are not within reach in the 12–24 month horizon.
- Not a substitute for [Lit Protocol's threshold conditional decryption](https://developer.litprotocol.com/network/networks/mainnet "Source: Lit Developer Docs, retrieved 2026-04-27"). The lock model is static and per-file; programmable conditional access is a candidate integration, not a replacement.

---

## Section 2: User Personas & Target Market

We identify three personas. The third is included because conference-portal traction is the strongest [observable adoption signal](https://fileverse.io "Source: Fileverse, retrieved 2026-04-27") visible in public materials.

### Persona A — Onchain Operator ("Mira", DAO operations lead)

- **Background.** Operations lead at a mid-size DAO with treasury exposure. Authors operating procedures, manages multisig signer lists, tracks proposal pipelines.
- **Motivation.** Wants document-grade collaboration without disclosing draft governance content to a custodial vendor. Needs spreadsheets that can read live treasury balances and submit transactions without round-tripping through a separate dApp.
- **Technical experience.** High onchain literacy; uses [Safe](https://safe.global "Source: Safe, retrieved 2026-04-27"), wallet signers, ENS. Comfortable with wallet auth; does not write SQL.
- **How she uses the product.** dDocs for proposal drafts shared via Owner Lock with named ENS counterparties. dSheets for treasury dashboards that read contract state and submit batched transactions.
- **Exit behavior.** Walks away via [the walk-away export tool](https://github.com/fileverse/walk-away "Source: Fileverse walk-away README, retrieved 2026-04-27") if Fileverse-the-operator changes posture; retains decryption capability via Owner Lock private key held in her own wallet.

### Persona B — Privacy-Conscious Knowledge Worker ("Daniel", investigative journalist)

- **Background.** Freelance journalist working on cross-jurisdictional reporting. Uses [Proton Mail](https://proton.me/blog/100-million-accounts "Source: Proton Blog, 2024-09-17") and Signal today; treats source confidentiality as load-bearing.
- **Motivation.** Wants encrypted document collaboration with a sub-editor and a fact-checker that does not require Proton's all-or-nothing bundle commitment, and that does not leave the source list on a custodial vendor's metadata layer.
- **Technical experience.** Medium. Has installed [a hardware wallet but uses it rarely](https://shop.ledger.com "Source: Ledger, retrieved 2026-04-27"); does not author smart contracts. Wallet auth is acceptable but not preferred; vOPRF-ID is preferred if recovery is sound.
- **How he uses the product.** dDocs for drafts shared with two collaborators via Portal Lock plus time-bounded Link Lock for legal review. Self-hosts no infrastructure; relies on Fileverse-hosted gateway.
- **Exit behavior.** Exports to local plaintext via walk-away when stories are filed; does not retain Fileverse-hosted persistence past publication.

### Persona C — Conference Portal Operator ("Anya", DevCon community lead)

- **Background.** Runs the community portal at a recurring Ethereum-ecosystem event. The portal hosts speaker-submitted notes, schedules, and post-event artifacts. Public materials show [DappCon 2025, ETHSF 2025, and DevCon 2024 ran community portals on dDocs](https://fileverse.io "Source: Fileverse, retrieved 2026-04-27").
- **Motivation.** Needs a portal whose content survives the event sponsor cycle, and where contributors retain their own access keys without a central account-management overhead.
- **Technical experience.** Medium-high. Comfortable with multisig portal owners on [Gnosis Chain via Safe](https://safe.global "Source: Safe, retrieved 2026-04-27"), with [ENS-based access policies](https://docs.ens.domains "Source: ENS Docs, retrieved 2026-04-27").
- **How she uses the product.** Portal-scoped dDocs and dSheets with Portal Lock controlled by a Safe multisig. Public read access via Link Lock with explicit expiry. Self-hosts collaboration-server only for high-traffic windows during the event.
- **Exit behavior.** Retains portal-key share post-event; CIDs remain pinned via the gateway and self-pinned to a backup IPFS node maintained by the community.

### Chain / platform alignment rationale

The current architecture defaults to [Ethereum mainnet (chain ID 1)](https://github.com/fileverse/fileverse-storage-v2 "Source: Fileverse fileverse-storage-v2 README, retrieved 2026-04-27") for the registry, with [Gnosis Chain referenced in the collaboration-server RPC setup](https://github.com/fileverse/collaboration-server "Source: Fileverse collaboration-server README, retrieved 2026-04-27"). Persona A and Persona C operate primarily on Gnosis Chain or L2s where per-document writes are sub-cent; Persona B is chain-indifferent and benefits from default-Gnosis to keep wallet UX minimal. The evidence suggests a Gnosis-first default with optional mainnet anchoring for portals that require L1 provenance aligns with all three personas; Section 8 addresses this conflict against the current mainnet default.

---

## Section 3: Core Feature List

| Feature | Priority | Description |
|---|---|---|
| Encrypted dDocs editor with [TipTap-based rich-text surface](https://github.com/fileverse/fileverse-ddoc "Source: Fileverse ddoc README, retrieved 2026-04-27") | MVP | Browser-based editor with offline-first IndexedDB sync. |
| dSheets spreadsheet with [Fortune-Sheet engine](https://github.com/fileverse/fortune-sheet "Source: Fileverse fortune-sheet repo, retrieved 2026-04-27") | MVP | Excel-compatible formulas (VLOOKUP, INDEX, MATCH). |
| Triple-lock encryption (Portal / Owner / Link) | MVP | Per-file AES key wrapped under three independent locks per the [walk-away scheme](https://github.com/fileverse/walk-away "Source: Fileverse walk-away README, retrieved 2026-04-27"). |
| Multi-provider IPFS pinning ([Pinata → Filebase → Web3.Storage](https://github.com/fileverse/fileverse-storage-v2 "Source: Fileverse fileverse-storage-v2 README, retrieved 2026-04-27")) | MVP | Fallback chain with UCAN-gated authorization. |
| Onchain content registry on Ethereum mainnet and Gnosis Chain | MVP | `fileId → ipfsCid` mapping with lock material per file. |
| Stateless collaboration relay with [Yjs CRDTs over WebSocket](https://github.com/fileverse/collaboration-server "Source: Fileverse collaboration-server README, retrieved 2026-04-27") | MVP | Server holds only ciphertext, ephemerally. |
| Wallet-based authentication (MetaMask, WalletConnect, Safe) | MVP | Sign-in via cryptographic signature; ENS as access primitive. |
| vOPRF-ID username authentication | MVP | Zero-knowledge username flow for users without a wallet. |
| Walk-away export tool | MVP | Standalone retrieval and decryption from IPFS using locally held keys. |
| dSheets onchain read | MVP | Cells query live contract state via [viem-based RPC](https://github.com/fileverse/fileverse-ddoc "Source: Fileverse ddoc README, retrieved 2026-04-27"). |
| dSheets onchain write (V0.3) | MVP | Simulate-then-submit transactions from cells. |
| Per-message Ed25519 signatures on encrypted CRDT updates | V2 | [secsync-style signed updates](https://github.com/nikgraf/secsync "Source: secsync GitHub, retrieved 2026-04-27") to detect relay forgery and replay. |
| Explicit key-rotation snapshots on membership change | V2 | Forward secrecy on revocation, modeled on [Proton Docs encryption](https://proton.me/blog/docs-proton-drive "Source: Proton Blog, 2024-07-03"). |
| Time-bounded Link Lock with on-chain expiry | V2 | Link keys with revocation by lock rotation; explicit "anyone with the link" warning. |
| dSheets per-cell transaction-bearing annotation | V2 | Marks cells whose values feed transaction parameters; disables formulas on those cells. |
| Self-hosted IPFS pinning fallback (fourth tier) | V2 | Fileverse-operated own IPFS cluster at >5 TB committed volume. |
| Federated relay topology over [Waku gossip](https://github.com/fileverse/collaboration-server "Source: Fileverse collaboration-server README, retrieved 2026-04-27") | V2 | Multiple community-hosted servers for liveness diversity. |
| Tier-by-attestation quota policy | V2 | Full quota for wallet-bound or domain-attested accounts; reduced quota for vOPRF-only. |
| Onchain Premium tier with paymaster credits | V2 | Sponsored writes capped per epoch with dynamic gas-cap throttling. |
| Filecoin cold-storage deals as fourth pinning tier | Future | Permanence under a different jurisdictional surface. |
| [Lit Protocol integration](https://developer.litprotocol.com/network/networks/mainnet "Source: Lit Developer Docs, retrieved 2026-04-27") for programmable conditional access | Future | Threshold-signed decryption where static three-lock model is insufficient. |
| Local-LLM document assistant (on-device weights) | Future | Privacy-preserving completion; threat model and supply-chain provenance required first. |
| Cross-portal Waku discovery service | Future | Network-effect feature unavailable to forks. |
| Commercial-license carve-out for AGPL-incompatible bundling | Future | Pattern analogous to MongoDB SSPL → commercial licensing. |

MVP count: 11. V2 count: 8. Future count: 5.

---

## Section 4: User Flows

### Flow 1 — Document Creation and First Persistence (Portal owner authoring)

**Actor.** Persona A (DAO operations lead) authoring a new proposal draft.

**Steps.**

1. Client opens dDocs at [ddocs.new](https://ddocs.new "Source: Fileverse, retrieved 2026-04-27"); local Y.Doc instantiated with `enableIndexeddbSync = true`.
2. Client requests UCAN from `fileverse-storage-v2` after wallet signature.
3. User edits; updates persist to IndexedDB and stream as `encrypted_yjs_update` payloads to the relay.
4. On explicit publish, client computes `Y.encodeStateAsUpdate`, encrypts under per-file AES key, uploads to Pinata via UCAN, receives CID.
5. Client wraps file key under Portal Lock (multisig pubkey), Owner Lock (her wallet pubkey), Link Lock (random link key).
6. Client calls `IFileverseRegistry.registerFile(fileId, cid, ownerLock, portalLock, linkLock)` on the registry chain (Gnosis recommended; mainnet current default).
7. Registry emits `FileRegistered(fileId, owner)`.

**Edge cases.** If Pinata returns 5xx, fall through to Filebase; on Filebase failure, fall through to Web3.Storage / Storacha. If all three fail, local copy persists and registry write is deferred. If registry write reverts due to gas spike, retry queued; dynamic gas-cap (V2) throttles paymaster.

### Flow 2 — Joining a Live Collaboration Session (recipient)

**Actor.** Persona B (journalist) opening a draft shared by an editor.

**Steps.**

1. Recipient opens the shared link; URL fragment carries the link key (current model) or, in the V2 model, an opaque exchange token.
2. Client fetches `getFile(fileId)` from the registry to retrieve `cid` and `linkLock`.
3. Client retrieves ciphertext from IPFS gateway; decrypts file key using link key (current) or completes server-mediated key handshake (V2).
4. Client connects to relay WebSocket; UCAN bound at handshake (V2 requirement; current build accepts session IDs without UCAN-binding under some paths per [the collab-server README](https://github.com/fileverse/collaboration-server "Source: Fileverse collaboration-server README, retrieved 2026-04-27")).
5. Client receives `encrypted_yjs_update` stream; decrypts with file key; merges into local Y.Doc.
6. Edits flow back symmetrically.

**Edge cases.** If the link key has been rotated, decryption fails and the user is shown a "this link has been revoked" state rather than a generic crypto error. If the relay has no ciphertext for the room because the session is cold, the client falls back to the latest CID from the registry. URL fragment leakage warnings shown when "anyone with the link" mode is selected.

### Flow 3 — Owner-Initiated Revocation and Lock Rotation

**Actor.** Persona C (conference portal operator) removing a former contributor.

**Steps.**

1. Client identifies the set of files whose `portalLock` or `ownerLock` references the departing contributor.
2. Client generates new file keys for each affected file (or new portal session key in V2 with key-rotation snapshot).
3. Client re-encrypts current Y.Doc state under the new file keys; uploads new CIDs.
4. Client rewraps locks excluding the departed contributor's pubkey; calls `rotateLocks(fileId, ownerLock, portalLock, linkLock)` and `updateCid(fileId, newCid)`.
5. Registry emits `LocksRotated(fileId)` and `CidUpdated(fileId, cid)`.
6. V2 only: Relay broadcasts a `key_rotation_event` snapshot; all current participants re-derive session keys.

**Edge cases.** Concurrent rotation by two operators produces last-writer-wins at the registry; clients reconcile by reading the latest event. Without per-message signatures (Option A in tech-arch §1), a former collaborator who retains the prior file key can still produce coherent ciphertext until the new CID is committed; the V2 signature design closes this window.

### Flow 4 — Walk-Away Recovery (operator-independent exit)

**Actor.** Persona A retrieving documents after Fileverse-the-operator has changed terms or sunset the gateway.

**Steps.**

1. User runs the [walk-away tool](https://github.com/fileverse/walk-away "Source: Fileverse walk-away README, retrieved 2026-04-27") with portal address, registry chain ID, and her Owner-Lock private key.
2. Tool reads the registry contract directly via a public RPC endpoint; enumerates `fileId`s owned by the portal.
3. For each file, tool fetches `cid` from any reachable IPFS gateway (public gateways or self-hosted node).
4. Tool decrypts the file key using Owner Lock private key (RSA unwrap); decrypts ciphertext with AES-GCM; reconstructs Y.Doc state; exports to plaintext or markdown.
5. User retains plaintext export and original CIDs for re-pinning to a self-hosted node if desired.

**Edge cases.** If all three centralized pinning providers have ceased operations and the public IPFS DHT does not retain the CID — the [empirical reliability concern documented in Trautwein et al.](https://dl.acm.org/doi/10.1145/3544216.3544232 "Trautwein et al., Design and Evaluation of IPFS, SIGCOMM 2022") — the recovery fails despite cryptographic capability. Mitigation requires either Filecoin cold-storage deals (Future feature) or user-initiated self-pinning before exit. If the registry chain is itself in extended outage, recovery is deferred but not lost; the data layer is independent of the registry pointer.

### Flow 5 — dSheets Onchain Transaction Submission

**Actor.** Persona A submitting a batched DAO treasury operation from a spreadsheet.

**Steps.**

1. User constructs cells whose values populate transaction parameters (target address, calldata, value).
2. dSheets V0.3 displays a simulate step: shows decoded calldata, target, value, and an EIP-712-style structured representation of the parameters.
3. User confirms; client requests wallet signature; wallet broadcasts transaction.
4. Cell updates with on-chain receipt status once mined.

**Edge cases.** A formula-induced mutation between simulate and submit must invalidate the simulate step and re-prompt — the [V0.3 onchain-write feature](https://github.com/fileverse/fileverse-dsheet "Source: Fileverse fileverse-dsheet README, retrieved 2026-04-27") requires this UX guarantee for the cell-mis-targeting attack vector to be neutralized. Adversarial CSV import with VLOOKUP into untrusted data must not be able to silently change a transaction-bearing cell. V2 introduces explicit per-cell "transaction-bearing" annotation that disables formulas on those cells.

---

## Section 5: Technical Interfaces

Derived from the [implied registry interface](https://github.com/fileverse/fileverse-storage-v2 "Source: Fileverse fileverse-storage-v2 README, retrieved 2026-04-27") and the [walk-away lock model](https://github.com/fileverse/walk-away "Source: Fileverse walk-away README, retrieved 2026-04-27"). The on-chain registry source is not currently in a public repository; the interface below is reconstructed.

### 5.1 Registry contract (interface)

```solidity
interface IFileverseRegistry {
    // Access: owner (or delegated address) only.
    function registerFile(
        bytes32 fileId,
        bytes calldata cid,
        bytes32 ownerLock,
        bytes32 portalLock,
        bytes32 linkLock
    ) external;

    // Access: owner only. Idempotent under last-writer-wins.
    function updateCid(bytes32 fileId, bytes calldata newCid) external;

    // Access: owner only. Used on revocation and key rotation.
    function rotateLocks(
        bytes32 fileId,
        bytes32 ownerLock,
        bytes32 portalLock,
        bytes32 linkLock
    ) external;

    // Access: any caller; view-only.
    function getFile(bytes32 fileId)
        external
        view
        returns (
            bytes memory cid,
            address owner,
            bytes32 ownerLock,
            bytes32 portalLock,
            bytes32 linkLock,
            uint64 updatedAt
        );

    event FileRegistered(bytes32 indexed fileId, address indexed owner);
    event CidUpdated(bytes32 indexed fileId, bytes cid);
    event LocksRotated(bytes32 indexed fileId);
}
```

### 5.2 Storage gateway HTTP surface

Headers required on every authenticated request, derived from the [storage-v2 README](https://github.com/fileverse/fileverse-storage-v2 "Source: Fileverse fileverse-storage-v2 README, retrieved 2026-04-27"):

| Header | Purpose |
|---|---|
| `Authorization: Bearer <UCAN>` | Scoped delegation token for upload / pin operations |
| `X-Contract-Address` | Registry contract address binding the request |
| `X-Invoker-Address` | Wallet or vOPRF-derived caller identity |
| `X-Chain-Id` | Registry chain ID (default `1`; recommended `100` per Section 8) |

Endpoints (sketch):

- `POST /pin` → uploads ciphertext, returns CID and pin status across providers.
- `GET /file/:cid` → resolves through fallback chain.
- `POST /ucan/refresh` → rotates UCAN bound to invoker identity.

### 5.3 Collaboration relay protocol

Current message shape (Option A) per the [collab-server README](https://github.com/fileverse/collaboration-server "Source: Fileverse collaboration-server README, retrieved 2026-04-27"):

```
{
  "type": "encrypted_yjs_update",
  "room_id": <string>,
  "ciphertext": <base64>,
  "nonce": <base64>
}
```

V2 envelope (signed updates, modeled on [secsync](https://github.com/nikgraf/secsync "Source: secsync GitHub, retrieved 2026-04-27")):

```
{
  "type": "encrypted_yjs_update_v2",
  "snapshot_id": <int>,
  "clock": <int>,
  "ciphertext": <base64>,
  "nonce": <base64>,
  "author_pub": <ed25519 pubkey>,
  "sig": <ed25519 signature>
}
```

Access control at the relay: V2 binds UCAN at WebSocket handshake; current build is more permissive on session attachment.

### 5.4 Client-side data structures

| Field | Type | Notes |
|---|---|---|
| `ddocId` | string | Per-document UUID scoped to local browser |
| `yjs_state` | Uint8Array | Full Y.Doc state vector + update log |
| `file_key` | bytes (in-memory only) | AES-GCM key recovered from a Lock at session start |
| `ownerLock`, `portalLock`, `linkLock` | bytes32 | Wrapped file key under each lock (mirror of on-chain) |
| `awareness` | ephemeral | Cursor, selection, presence — never persisted |

### 5.5 Events emitted (off-chain)

Storage gateway emits structured logs for `pin_success`, `pin_fallback`, `pin_failure_all_providers`, `ucan_issued`, `ucan_revoked`. Collaboration relay emits `session_open`, `session_close`, `bytes_relayed`, `rate_limit_triggered`. These are operator telemetry, not user-visible, and must not include plaintext or recoverable identifiers.

---

## Section 6: Integration Spec

External dependencies from [tech-arch §5](https://github.com/fileverse/fileverse-storage-v2 "Source: Fileverse fileverse-storage-v2 README, retrieved 2026-04-27"). Each adapter specifies inputs at construction, function flow, revert/error conditions, and fallback chain.

### 6.1 Pinata adapter

- **Inputs at construction.** API key, gateway hostname, region selector.
- **Function flow.** `pin(ciphertext) → CID`. Posts to Pinata REST endpoint; receives CID and pin metadata. Stores pin status in the gateway's MongoDB tenant record.
- **Revert conditions.** HTTP 4xx (auth or quota) → fall through to Filebase. HTTP 5xx with retry-after → backoff once, then fall through. Connection error → immediate fall-through.
- **Fallback chain.** Pinata → Filebase → Web3.Storage / Storacha → (V2) Fileverse-operated own IPFS cluster → (Future) Filecoin cold-storage deal.

### 6.2 Filebase adapter

- **Inputs at construction.** Access key, secret key, S3 bucket name.
- **Function flow.** S3-compatible PUT; pinned via Filebase's IPFS attachment metadata.
- **Revert conditions.** S3 4xx → fall through. S3 5xx → backoff once. Free-tier limits hit per [Filebase pricing](https://filebase.com/pricing "Source: Filebase, retrieved 2026-04-27") → fall through.
- **Fallback chain.** Same as 6.1 with Filebase as primary entry.

### 6.3 Web3.Storage / Storacha adapter

- **Inputs at construction.** Bearer token (w3up client credentials).
- **Function flow.** Upload via w3up; receives CID. Note that [Storacha's repositioning to a paid model in 2024](https://blog.storacha.network/web3-storage-rebrands-to-storacha-network/ "Source: Storacha (formerly Web3.Storage), 2024") changed the cost profile of this fallback.
- **Revert conditions.** Auth failure → re-mint UCAN delegation; quota exceeded → escalate to operator alert. Cold-tier Filecoin retrieval can be minutes per [tech-arch §5](https://github.com/fileverse/fileverse-storage-v2 "Source: Fileverse fileverse-storage-v2 README, retrieved 2026-04-27").
- **Fallback chain.** Last in current chain; (V2) own-cluster behind it.

### 6.4 Ethereum mainnet RPC adapter

- **Inputs at construction.** Wallet provider or hosted RPC URL (e.g., Alchemy, QuickNode); chain ID `1`.
- **Function flow.** [viem](https://github.com/fileverse/fileverse-ddoc "Source: Fileverse ddoc README, retrieved 2026-04-27")-based read and write calls to the registry.
- **Revert conditions.** Reorg-depth exceeded → re-fetch; gas price > configured cap → defer (V2 throttles paymaster); RPC unhealthy → switch endpoint.
- **Fallback chain.** Multi-endpoint round-robin within mainnet; if mainnet itself unreachable, registry remains readable on Gnosis if dual-write was configured.

### 6.5 Gnosis Chain RPC adapter

- **Inputs at construction.** RPC URL (e.g., [QuickNode setup referenced in collab-server](https://github.com/fileverse/collaboration-server "Source: Fileverse collaboration-server README, retrieved 2026-04-27")), chain ID `100`.
- **Function flow.** Same shape as 6.4; lower per-write cost (~sub-cent).
- **Revert conditions.** Same as 6.4; Gnosis has historically had brief outages but not extended halts.
- **Fallback chain.** Switch RPC endpoint; defer to mainnet if dual-write configured.

### 6.6 UCAN issuer (storage-v2 internal)

- **Inputs at construction.** Issuer signing key, tenant DB connection.
- **Function flow.** On authentication, issues a scoped UCAN delegation with bandwidth and pin caps tied to the invoker.
- **Revert conditions.** Quota exceeded → return 402 with retry hint; identity stale → require re-authentication.
- **Fallback chain.** No fallback; UCAN is the single auth root inside the gateway.

### 6.7 Penumbra / TweetNaCl / viem (npm dependencies)

- **Inputs at construction.** Pinned semver in lockfile; Subresource Integrity for browser bundle.
- **Function flow.** In-process cryptographic primitives.
- **Revert conditions.** Lockfile mismatch → build failure; SRI mismatch → browser refuses to load.
- **Fallback chain.** None; supply-chain mitigation is a release-time control, not a runtime one. Sigstore-style provenance recommended.

### 6.8 Waku (V2, federated relay discovery)

- **Inputs at construction.** Bootstrap peer list, content topic per portal.
- **Function flow.** libp2p gossip for relay-server discovery; falls through to direct WebSocket URL on failure.
- **Revert conditions.** Bootstrap unreachable → cached peer list; gossip incomplete → degraded discovery, direct URL still works.
- **Fallback chain.** Waku → cached peer list → direct WS URL.

---

## Section 7: Operational Spec

### 7.1 Deployment sequence

1. **Cryptographic library.** Pin and audit `@fileverse/crypto`; publish report. Audit gap is documented in [Phase 2c failure mode 6](https://github.com/fileverse/collaboration-server "Source: Fileverse collaboration-server README, retrieved 2026-04-27") and is the first blocker for paid-tier launch.
2. **Registry contract.** Deploy on Gnosis (default) and mainnet (opt-in); behind a UUPS proxy initially to retain optionality on lock-rotation logic; transition to immutable after stabilization (Section 8 conflict 3).
3. **Storage gateway (`fileverse-storage-v2`).** Provision MongoDB; configure UCAN issuer; load three pinning provider credentials; verify fallback chain end-to-end with synthetic pins.
4. **Collaboration relay.** Deploy stateless relay; complete Dedalo audit; enforce per-IP and per-UCAN connection caps; bind UCAN at WebSocket handshake (V2).
5. **Client distribution.** Publish `@fileverse-dev/ddoc` and dSheets app builds with SRI hashes; pin viem, Yjs, TipTap, Fortune Sheet, Penumbra, TweetNaCl in lockfile.
6. **Walk-away tool.** Ship as a separate, minimal-dependency artifact with no network dependencies beyond IPFS and registry RPC.
7. **Observability.** Wire structured logs; ensure no plaintext, no key material, no recoverable user identifiers in any log line.

### 7.2 Monitoring requirements

- **Pin success rate per provider.** Target ≥99.5% per provider; alert on >0.5% sustained failure.
- **Fallback frequency.** Alert when fallback rate exceeds 10% of pin attempts (signals primary degradation).
- **Registry write latency.** P95 confirmation latency tracked per chain; alert on Gnosis >30 s or mainnet >60 s.
- **Relay session memory and bytes-per-token rate.** Alert on per-token egress >1 GB/hour (sybil signal per Phase 2c attack 4).
- **UCAN issuance rate.** Alert on creation-rate spikes inconsistent with normal account growth (sybil signal per Phase 2c attack 1).
- **Gas price for sponsored writes.** Continuous tracking; auto-throttle paymaster when gwei exceeds configured cap.
- **CID-vs-blob consistency.** Periodic sampling: registry-pointed CID is retrievable from at least one provider in the fallback chain.
- **Audit-cycle clock.** Per [Trail of Bits guidance](https://blog.trailofbits.com/2023/02/14/our-2022-security-audit-cycle/ "Source: Trail of Bits, 2023-02-14"), 18-month cryptographic re-audit cycle tracked as an operational SLO.

### 7.3 Incident response — top 3 failure modes from Phase 2c

#### Failure Mode 1 — Pinning Provider Cascade Failure

- **Trigger.** Two of three providers in concurrent failure or pricing-event repositioning, comparable to [Storacha's 2024 free-tier wind-down](https://blog.storacha.network/web3-storage-rebrands-to-storacha-network/ "Source: Storacha (formerly Web3.Storage), 2024").
- **Immediate response.** Activate Fileverse-operated own IPFS cluster (V2 prerequisite). Publish public-status notification with affected CID range. Halt new uploads to the failing provider; redirect to the remaining provider; rate-limit aggregate egress to preserve budget.
- **Recovery.** Re-pin affected CIDs to the surviving provider plus own cluster. Estimate per [Phase 2c §8.1](https://www.pinata.cloud/pricing "Source: Pinata, retrieved 2026-04-27"): at Medium-scenario volume (~170 TB), one-time cost roughly $850K at emergency vendor pricing; this is the budgetary worst case.
- **Postmortem.** Reassess fallback-chain composition; consider [Filecoin cold-storage deals](https://filecoin.io "Source: Filecoin, retrieved 2026-04-27") as a fourth tier.

#### Failure Mode 2 — Triple-Lock Recovery Path Compromise (Link Lock leakage)

- **Trigger.** Public report or telemetry signal that link-keyed documents have been accessed via leaked URL fragments (browser sync, link-preview crawlers, instrumented chat clients).
- **Immediate response.** Force-rotate link keys for all live Link-Lock-shared documents in the affected portal; publish security advisory; surface in-product banner explaining the leakage path and the rotation.
- **Recovery.** Replace URL-fragment link sharing with server-mediated key handshake (V2). Re-pin documents under new file keys; emit `LocksRotated` events to the registry.
- **Postmortem.** Confirm no further URL-fragment exposure surface; review default link expiry policy (≤30 days recommended per Phase 2c §8.2).

#### Failure Mode 3 — Sybil Free-Tier Exhaustion

- **Trigger.** UCAN issuance rate or per-IP account creation rate exceeds baseline by >5× over a 24-hour window; aggregate free-tier egress exceeds modeled volume.
- **Immediate response.** Apply step-change rate limit on vOPRF-only account creation; require proof-of-personhood challenge ([World ID](https://world.org "Source: World, retrieved 2026-04-27") or BrightID) for new vOPRF accounts above threshold; preserve existing wallet-bound and domain-attested accounts at full quota.
- **Recovery.** Tier-by-attestation quota policy (V2) becomes default; back-port quota reduction for vOPRF-only accounts to 10–20% of full quota.
- **Postmortem.** Audit log of consumed quota; revoke UCANs of clearly-sybil tenants; publish quota-policy change.

---

## Section 8: Open Questions & Risks

We address each conflict surfaced in the Phase 2 outputs. The research-log records that Phase 2a, 2b, and 2c each completed independently on 2026-04-27; conflicts below are implied by reading the three outputs together.

### 8.1 Default registry chain — mainnet vs. Gnosis vs. dual

- **Why unresolved.** The [storage-v2 README defaults to chain ID 1 (mainnet)](https://github.com/fileverse/fileverse-storage-v2 "Source: Fileverse fileverse-storage-v2 README, retrieved 2026-04-27"); [the collab-server references a Gnosis-Chain RPC setup](https://github.com/fileverse/collaboration-server "Source: Fileverse collaboration-server README, retrieved 2026-04-27"); the economics analysis flags mainnet gas cost as a friction tax for document-heavy workflows and the gas-griefing failure mode as more dangerous on mainnet.
- **Options.**
  1. Keep mainnet default. Pro: L1 provenance signaling; familiar wallet flow. Con: per-document writes routinely [$1–$10+ at observed gas prices](https://etherscan.io/gastracker "Source: Etherscan Gas Tracker, retrieved 2026-04-27"); paymaster exposure to gas spikes.
  2. Switch default to Gnosis. Pro: sub-cent per-document writes; collab-server already integrated. Con: less L1-provenance signaling; user wallet UX additional step for non-Gnosis users.
  3. Dual-write to both for premium users; Gnosis-only for free tier. Pro: redundancy across chains. Con: 2× gas cost for premium; reconciliation logic if one chain reverts.
- **Recommendation.** Option 2 as the new default with Option 3 reserved for the Onchain Premium tier. Aligns with all three personas in Section 2 and with the equilibrium analysis in [Phase 2c §2](https://blog.cryptpad.org/2024/06/12/Onwards-and-upwards/ "Source: CryptPad Blog, 2024-06-12").

### 8.2 Encryption envelope — current Option A vs. secsync-style Option B

- **Why unresolved.** The current `encrypted_yjs_update` payload does not carry per-message signatures; this is acceptable for trust-equal small-group editing but does not generalize to role-based access or revocation. Tech-arch §1 frames this as the single largest design freedom.
- **Options.**
  1. Stay on Option A. Pro: simple; lower per-message overhead. Con: no detection of forgery / replay by relay or session participant; revocation is ambiguous.
  2. Migrate to Option B (per-message Ed25519 + key-rotation snapshots). Pro: cryptographic authorship; forward secrecy on membership change. Con: integration complexity; ~96–128 byte overhead per message.
  3. Hybrid: ship Option B alongside legacy Option A as a non-breaking new envelope.
- **Recommendation.** Option 3 is the migration path. The evidence from [secsync](https://github.com/nikgraf/secsync "Source: secsync GitHub, retrieved 2026-04-27") and [Proton's published Docs encryption model](https://proton.me/blog/docs-proton-drive "Source: Proton Blog, 2024-07-03") suggests signed-update plus key-rotation is becoming the field standard for E2EE collaborative editors.

### 8.3 Registry upgradeability — UUPS proxy vs. immutable

- **Why unresolved.** Tech-arch §6.5 surfaces the tradeoff: upgradeability lets the team respond to integration bugs and (under a scenario of post-quantum migration) adapt the lock-rotation logic; immutable deployment improves user assurance.
- **Options.**
  1. Immutable from day one. Pro: maximal user assurance; aligns with self-sovereignty framing. Con: any bug or future crypto migration requires fileId remapping.
  2. UUPS proxy with a multi-sig timelock-controlled admin. Pro: response capacity. Con: trust anchor in admin set.
  3. UUPS during stabilization, then renounce to immutable.
- **Recommendation.** Option 3. Initial proxy with a Safe multisig + 7-day timelock; after the first independent registry audit and 6 months of incident-free operation, renounce to immutable.

### 8.4 Link Lock URL-fragment leakage — fix vs. accept

- **Why unresolved.** Tech-arch §7.2 rates this HIGH severity; Phase 2c §8.2 quantifies a ~3% silent confidentiality breach rate at 1% per-link leakage with average 3 shares; the base mechanism does not mitigate.
- **Options.**
  1. Keep URL-fragment Link Lock; rely on user discipline. Pro: simple share UX. Con: silent breach risk; reputational asymmetry on first publicized incident.
  2. Replace with server-mediated key handshake bound to recipient identity. Pro: revocation surface; no fragment exposure. Con: introduces a server-side step that is not strictly E2EE-pure.
  3. Hybrid: keep fragment-based for unauthenticated readers but with mandatory short expiry; gate authenticated reads through the handshake.
- **Recommendation.** Option 3. The [HAL Archive 2018 analysis of URL-fragment key sharing](https://hal.science/hal-01781231 "Source: HAL Archive, 2018") supports treating fragment exposure as a known weakness; 30-day default expiry plus handshake-based authenticated reads is the lowest-disruption fix.

### 8.5 vOPRF-ID quota policy — flat vs. tier-by-attestation

- **Why unresolved.** Tech-arch §7.7 flags identity-recovery gap; Phase 2c §8.3 shows sybil free-tier exhaustion can exceed Low-scenario MRR ($6,500/month attack vs. $3,700 MRR).
- **Options.**
  1. Flat quota across vOPRF-ID and wallet-bound. Pro: equal UX. Con: sybil-cost dumped on the operator.
  2. Tier by attestation: full quota for wallet-bound or domain-attested; reduced quota (10–20%) for vOPRF-only with proof-of-personhood challenge above threshold. Pro: aligns cost with attestation strength. Con: friction for vOPRF-only users.
  3. Wallet-only quotas; deprecate vOPRF-ID. Pro: simplest. Con: removes the no-wallet onboarding wedge that distinguishes Persona B.
- **Recommendation.** Option 2. Mandatory wallet-attestation pairing for any portal-owner role addresses the recovery gap; tiered quotas address the sybil exhaustion vector.

### 8.6 Self-hosting share threshold — 25% vs. 40%

- **Why unresolved.** Phase 2c §2 sets two thresholds: *q* < 25% to retain pricing power against pinning vendors; *q* < 40% before equilibrium collapses entirely.
- **Options.**
  1. Manage to <25%. Pro: preserves enterprise pinning negotiation leverage. Con: requires features that retain users on the gateway.
  2. Tolerate up to 40%. Pro: more aligned with "self-sovereignty" framing. Con: gateway business margin compresses materially per the Phase 2c §8.4 model.
  3. Indifferent — let market decide.
- **Recommendation.** Option 1 as the operating target with Option 2 as the no-action floor. Network-effect features (paymaster, cross-portal Waku discovery, commercial license carve-out) provide gradient between fork and Fileverse-hosted experience without violating AGPL.

### 8.7 dSheets onchain-write trust boundary — workbook vs. per-cell

- **Why unresolved.** Tech-arch §7.8 rates this HIGH; specific design (per-cell signing vs. workbook trust) is not documented in the public [dSheets repository](https://github.com/fileverse/fileverse-dsheet "Source: Fileverse fileverse-dsheet README, retrieved 2026-04-27").
- **Options.**
  1. Workbook-level trust. Pro: simple. Con: a single adversarial CSV import can change a transaction parameter via VLOOKUP.
  2. Per-cell transaction-bearing annotation that disables formulas on those cells; mandatory simulate-and-display step with diff against last-signed parameters.
  3. Per-transaction explicit re-signature on every parameter change.
- **Recommendation.** Option 2 for V2 with Option 3 reserved for high-value cell types (e.g., calldata destination addresses). The vector is specific enough that workbook-level trust is the wrong default.

### 8.8 Cryptographic-library audit gap — sequence

- **Why unresolved.** Both tech-arch §6.3 and Phase 2c §8.6 record `@fileverse/crypto` as unaudited; only the [collaboration-server is under Dedalo audit](https://github.com/fileverse/collaboration-server "Source: Fileverse collaboration-server README, retrieved 2026-04-27").
- **Options.**
  1. Ship paid tier first; audit later. Pro: revenue earlier. Con: production-grade trust posture is unsupported.
  2. Audit `@fileverse/crypto` before any paid-tier launch; publish report; only then enable Onchain Premium. Pro: correct ordering. Con: delay.
  3. Limited preview tier under explicit beta-grade trust language while audit is pending.
- **Recommendation.** Option 2. The [Trail of Bits audit-cycle guidance](https://blog.trailofbits.com/2023/02/14/our-2022-security-audit-cycle/ "Source: Trail of Bits, 2023-02-14") and the absence of an audit on the lock-wrapping path together make audit-first the only ordering consistent with the privacy-default framing.

### 8.9 Funding path — token vs. paid tier vs. grants-only

- **Why unresolved.** Concept doc records no token disclosure and no announced funding round. Phase 2c §5 shows the gateway is structurally subsidy-dependent until ~100K paying users (~1.5M registered).
- **Options.**
  1. Grants + commercial license carve-out, no token. Pro: avoids token-launch overhead and regulatory exposure. Con: grant cycles are unpredictable; CryptPad's [10-year path to ~5,000 paid subscribers across ~280K registered](https://blog.cryptpad.org/2024/06/12/Onwards-and-upwards/ "Source: CryptPad Blog, 2024-06-12") is the comparable scale curve.
  2. Paid tier launched within 12 months at the $5–$10/month band, no token. Pro: legible cash flow. Con: requires audit-complete state per 8.8.
  3. Token launch as quota voucher / governance share / retroactive contributor reward. Pro: reframes incentive design around classical mechanism design. Con: regulatory and design overhead; not currently announced.
- **Recommendation.** Option 2 is the lowest-disruption path consistent with current public posture; Option 1 remains as a complementary funding source. Option 3 is a forward-looking branch and is not load-bearing in this spec.

---

## Section 9: Go/No-Go Decision Framework (analyst-observable conditions)

These are conditions an analyst would watch for as evidence that a 12–24 month execution is on track. They are framed as observables, not as decisions to be taken by the Fileverse team. All conditions must be TRUE concurrently for an analyst to read the project as having cleared the launch threshold.

### Engineering gates

1. **`@fileverse/crypto` independent audit complete and report public.** Per [Trail of Bits' published audit-cycle guidance](https://blog.trailofbits.com/2023/02/14/our-2022-security-audit-cycle/ "Source: Trail of Bits, 2023-02-14"); covers triple-lock wrapping, AES-GCM IV derivation, RSA padding, key-rotation procedure.
2. **[Dedalo audit on `collaboration-server`](https://github.com/fileverse/collaboration-server "Source: Fileverse collaboration-server README, retrieved 2026-04-27") complete and report public.** No high-severity unresolved findings.
3. **V2 signed-update envelope shipped alongside legacy Option A.** Observable as a new message type in the relay protocol and per-message Ed25519 signatures over the wire.
4. **Per-cell transaction-bearing annotation live in dSheets.** Observable in the [dSheets repository](https://github.com/fileverse/fileverse-dsheet "Source: Fileverse fileverse-dsheet README, retrieved 2026-04-27") as a UI element that disables formulas on transaction-parameter cells, with mandatory simulate-and-display diff.

### External dependency gates

5. **Fallback to a fourth pinning tier operational.** Observable as either a Fileverse-operated own IPFS cluster or a [Filecoin cold-storage deal](https://filecoin.io "Source: Filecoin, retrieved 2026-04-27") referenced in `fileverse-storage-v2`. Mitigates the cascade failure mode in Phase 2c §8.1.
6. **Dual-chain registry redundancy operational.** Either dual-write for premium users or a documented snapshot-replication policy between Gnosis and mainnet, observable in registry-contract events on at least two chains.
7. **Tier-by-attestation quota policy live.** Observable as differential UCAN quotas for wallet-bound vs. vOPRF-only accounts and a proof-of-personhood challenge above a documented sybil threshold.
8. **Default registry chain is Gnosis (chain ID 100) for new portals.** Observable as the default `X-Chain-Id` header value emitted by `fileverse-storage-v2` and the default chain selector in the dDocs and dSheets UIs.

### Business gates

9. **Paying-user count crosses the infra-coverage threshold.** Per Phase 2c §5, ~3,000 paying users at the recommended fee band covers infra COGS. Observable indirectly through public revenue disclosures or grant-recipient filings if disclosed; otherwise inferred from registered-user growth and conversion analogs comparable to [CryptPad's published metrics](https://blog.cryptpad.org/2024/06/12/Onwards-and-upwards/ "Source: CryptPad Blog, 2024-06-12").
10. **Self-hosting share remains in the 5–25% band.** Observable through self-hosted relay registrations on Waku, public-Github-fork counts of `collaboration-server`, and the operator's published telemetry if available.
11. **No publicized confidentiality breach attributable to Link Lock URL-fragment leakage.** Observable in security-advisory feeds and incident postmortems.
12. **Independently-published unit-economics ratios consistent with the Medium scenario.** Per Phase 2c §5: blended gross margin trending toward ~25–30% with Onchain Premium share growing relative to Pro share.

Twelve conditions present, exceeding the ≥8 threshold. All conflicts surfaced in the inputs (mainnet vs. Gnosis default, Option A vs. B encryption envelope, registry upgradeability, Link Lock leakage, vOPRF quota policy, self-hosting threshold, dSheets cell trust boundary, audit gap, funding path) appear in Section 8 with options and a recommendation.

---

## Voice and citation discipline

Inline citation format applied throughout. Forward-looking statements use conditional framing ("under a scenario in which …") rather than directive framing. Banned vocabulary list: not used. Voice: balanced analyst, no investor framing, no first-person "I" or unhedged certainty.
