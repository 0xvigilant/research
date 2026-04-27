# Technical Architecture — Fileverse

_Phase 2b · Compiled 2026-04-27 · Researcher-portfolio mode_

This document analyzes the technical architecture of Fileverse's productivity suite (dDocs and dSheets) as observed in its public repositories and live deployments. Where the existing implementation is documented, we reconstruct its design rationale; where the architecture has degrees of freedom or open questions (e.g., long-term IPFS persistence, identity recovery), we present alternative options and offer a balanced recommendation. Forward-looking statements are framed conditionally.

---

## Section 1: Core Mechanism Design — Encrypted Local-First CRDT Sync

The central technical primitive of Fileverse is **encrypted local-first collaborative editing**: documents live in the client (IndexedDB), are merged via [Y.js CRDTs](https://link.springer.com/chapter/10.1007/978-3-319-19890-3_55 "Nicolaescu et al., Yjs, ICWE 2015"), and are relayed by a stateless WebSocket server that never sees plaintext. Persistence is delegated to IPFS pinning, with smart contracts acting as a content registry. We find this model sits at the intersection of three published lineages: Yjs (CRDT), the [secsync architecture](https://github.com/nikgraf/secsync "Source: secsync GitHub, retrieved 2026-04-27") for E2EE Yjs relays, and the [local-first essay by Ink & Switch](https://www.inkandswitch.com/local-first/ "Ink & Switch, 2019").

The design freedom that matters most is **how encrypted CRDT updates are authenticated and merged on an untrusted relay**. We present two options.

### Option A — Client-encrypted Yjs updates over an opaque WebSocket relay (current Fileverse)

The collaboration server [accepts payloads typed as `"encrypted_yjs_update"` and stores them only ephemerally before pushing the latest state to IPFS](https://github.com/fileverse/collaboration-server "Source: Fileverse collaboration-server README, retrieved 2026-04-27"). The server has no decryption key; it routes ciphertext between participants in the same session.

**Pseudocode (client side):**

```
on local_edit(doc):
    yjs_update = Y.encodeStateAsUpdate(doc, prev_state_vector)
    ciphertext = AES_GCM_encrypt(file_key, nonce, yjs_update)
    ws.send({type: "encrypted_yjs_update", room_id, ciphertext, nonce})

on receive(ciphertext, nonce):
    yjs_update = AES_GCM_decrypt(file_key, nonce, ciphertext)
    Y.applyUpdate(doc, yjs_update)
    persist_local(doc)  # IndexedDB
```

**Tradeoffs.**
- Pro: Server is genuinely server-blind; participants only need a shared symmetric `file_key` (delivered out-of-band via the Portal/Owner/Link lock model).
- Pro: Yjs guarantees [strong eventual consistency](https://arxiv.org/abs/1608.03960 "Kleppmann & Beresford, A Conflict-Free Replicated JSON Datatype, 2016") regardless of message order.
- Con: Without per-message signatures, the relay (or any session participant) could forge or replay updates; integrity rests on AES-GCM authenticity plus knowledge of `file_key`. Anyone holding `file_key` can mutate history.
- Con: No causal log on the server means the server cannot detect dropped updates or enforce author identity at the relay layer.

### Option B — secsync-style signed encrypted updates with key-rotation events

The [secsync protocol](https://github.com/nikgraf/secsync "Source: secsync GitHub, retrieved 2026-04-27") defines a published format for E2EE Yjs in which each update carries (i) a per-message symmetric encryption (XChaCha20-Poly1305), (ii) an Ed25519 signature from the author's session key, and (iii) explicit key-rotation events when membership changes.

**Pseudocode (client side):**

```
on local_edit(doc):
    update = Y.encodeStateAsUpdate(doc, prev_sv)
    nonce = random(24)
    ct = XChaCha20Poly1305_encrypt(current_session_key, nonce, update,
                                   aad = (snapshot_id, clock))
    sig = Ed25519_sign(author_priv, hash(snapshot_id, clock, ct, nonce))
    relay.publish({snapshot_id, clock, nonce, ct, author_pub, sig})

on member_change(new_set):
    new_session_key = HKDF(random())
    for member in new_set:
        wrap = sealed_box(member.pub, new_session_key)
        relay.publish_keyrotation(snapshot_id+1, wraps)
```

**Tradeoffs.**
- Pro: Per-update signatures give cryptographic authorship inside an otherwise opaque CRDT log; replay and forgery are detectable by clients without trusting the relay.
- Pro: Explicit key-rotation snapshots provide forward secrecy for membership changes — a property A does not have today.
- Con: Higher per-message overhead (~96–128 bytes signature + key-id metadata) and more complex client state machine.
- Con: Existing Yjs application code does not produce signatures; integration requires a wrapper layer and careful ordering with awareness updates (cursors, presence).

### Recommendation

We recommend **migrating Option A toward Option B incrementally**, beginning with per-message Ed25519 signatures keyed to wallet identities and ENS, and adding explicit key-rotation snapshots when membership changes. The current model is adequate for trust-equal small groups (the primary observed use case at conference portals — see Phase 1), but does not generalize cleanly to "open team" semantics where a former collaborator should lose write access. The evidence from the [secsync reference](https://github.com/nikgraf/secsync "Source: secsync GitHub, retrieved 2026-04-27") and from [Proton's published Docs encryption model](https://proton.me/blog/docs-proton-drive "Source: Proton, 2024-07-03") both suggest that signed-update plus key-rotation is becoming the field standard. The migration path is non-breaking if the new envelope is added alongside the legacy `encrypted_yjs_update` type.

Under a scenario where Fileverse retains its current trust-equal collaboration mode only, Option A is acceptable; under any scenario involving role-based access (editor vs. viewer enforced cryptographically) or revocation, Option B becomes load-bearing.

---

## Section 2: State Management

Fileverse splits state across four tiers, each with distinct mutability and authority semantics.

### 2.1 Client-local (mutable, authoritative for in-flight edits)

[IndexedDB sync is enabled per-document via the `enableIndexeddbSync` flag in `@fileverse-dev/ddoc`](https://github.com/fileverse/fileverse-ddoc "Source: Fileverse ddoc README, retrieved 2026-04-27"). Each document is stored as a Yjs document blob keyed by a `ddocId`. This is the working copy and survives offline.

| Field | Type | Notes |
|---|---|---|
| `ddocId` | string | UUID-style identifier scoped to the local browser |
| `yjs_state` | Uint8Array | Full Y.Doc state vector + update log |
| `file_key` | bytes | AES key (in-memory only; recovered from a Lock at session start) |
| `awareness` | ephemeral | Cursor, selection, presence (not persisted) |

### 2.2 Relay-ephemeral (mutable, non-authoritative)

The collaboration server [holds session state only in memory and deletes it once the latest document is pushed to IPFS](https://github.com/fileverse/collaboration-server "Source: Fileverse collaboration-server README, retrieved 2026-04-27"). State at this tier is ciphertext.

### 2.3 IPFS pinned (immutable per-CID, mutable via new CID)

The encrypted blob is content-addressed; mutation is "publish a new CID". `fileverse-storage-v2` uses [Pinata, Filebase, and Web3.Storage as redundant pinning providers, with API-key authentication per provider](https://github.com/fileverse/fileverse-storage-v2 "Source: Fileverse fileverse-storage-v2 README, retrieved 2026-04-27"). The MongoDB layer in the storage service maps tenants to provider credentials and tracks pin status.

### 2.4 On-chain registry (mutable, authoritative for "current CID for file ID")

The storage service [requires a Contract Address, Invoker Address, and Chain ID in request headers](https://github.com/fileverse/fileverse-storage-v2 "Source: Fileverse fileverse-storage-v2 README, retrieved 2026-04-27"), implying a registry contract per portal. Default chain ID is `1` (Ethereum mainnet). Sketch of the on-chain schema implied by the headers and the walk-away repository:

```solidity
struct File {
    bytes32 fileId;          // app-assigned identifier
    bytes   ipfsCid;         // current CID of encrypted blob
    address owner;           // portal owner
    bytes32 ownerLock;       // file_key encrypted under owner pubkey
    bytes32 portalLock;      // file_key encrypted under portal pubkey
    bytes32 linkLock;        // file_key encrypted under link key (optional)
    uint64  updatedAt;
}

mapping(bytes32 => File) files;        // fileId -> File
mapping(address => uint256) quota;     // owner -> bytes used (UCAN-enforced)
```

**Mutability rules.** `fileId` and `owner` immutable post-creation; `ipfsCid` and `*Lock` fields rotate on key change or content change; `quota` updates on every pin operation.

---

## Section 3: Settlement / Resolution

"Settlement" here refers to **finalizing a document version**: the moment an edit transitions from ephemeral CRDT updates in the relay to a durable, shareable artifact.

### 3.1 Trigger model

Three triggers are observable in the codebase:

1. **Idle persistence.** The collaboration server [pushes the latest state to IPFS once the RTC session ends and clears its memory](https://github.com/fileverse/collaboration-server "Source: Fileverse collaboration-server README, retrieved 2026-04-27"). This is the default settlement path.
2. **Explicit publish.** The dDocs UI exposes an explicit "publish to IPFS" button (observed at [ddocs.new](https://ddocs.new "Source: Fileverse, retrieved 2026-04-27")) which forces the client to encrypt the current Y.Doc, upload to IPFS via the storage service, and update the on-chain registry.
3. **Walk-away export.** The [walk-away repository describes a recovery flow where users supply portal address + RSA key material to retrieve and decrypt content independently of the dDocs frontend](https://github.com/fileverse/walk-away "Source: Fileverse walk-away README, retrieved 2026-04-27"). This is settlement-as-export.

### 3.2 External data needed

- **IPFS gateway availability** for fetching the encrypted blob.
- **RPC endpoint** for the registry chain (mainnet or [Gnosis Chain via QuickNode setup referenced in the collaboration server](https://github.com/fileverse/collaboration-server "Source: Fileverse collaboration-server README, retrieved 2026-04-27")).
- **UCAN issuer** to mint an authorization token gating uploads (the storage service issues these post-authentication).

### 3.3 Who can trigger

| Action | Authorized party | Mechanism |
|---|---|---|
| Push update to relay | Any session participant with `file_key` | AES-GCM-authenticated message |
| Pin to IPFS | Holder of valid UCAN | Bearer token in `Authorization` header |
| Update on-chain CID | Portal owner (or delegate) | Signed transaction to registry contract |
| Decrypt via walk-away | Anyone holding any of {Portal, Owner, Link} private key | Local decryption only |

### 3.4 Failure and ambiguity modes

- **All three IPFS providers unreachable.** The fallback chain ([Pinata → Filebase → Web3.Storage](https://github.com/fileverse/fileverse-storage-v2 "Source: Fileverse fileverse-storage-v2 README, retrieved 2026-04-27")) has no further escape; a self-hosted IPFS pin is the only out.
- **On-chain CID stale relative to actual blob.** If a registry update fails after a successful pin, readers see the previous version. Idempotent retry from the client mitigates this; no on-chain "pending pin" state is documented.
- **Concurrent settlement.** Two clients pushing on-chain CID updates near-simultaneously will produce a last-writer-wins outcome at the EVM level; the CRDT history on IPFS may diverge from the registry pointer until reconciled.

---

## Section 4: Infrastructure / Platform

The architecture spans three independent infrastructure decisions: **registry chain**, **IPFS pinning provider**, and **collaboration relay topology**. We present a scored comparison of registry-chain options, since this is where the cost structure and the user-experience footprint are most concentrated.

### Registry chain comparison (scored 1–5; higher is better)

| Chain | Tx cost (1=expensive) | Wallet UX | Confirmation latency | Ecosystem alignment | EVM equivalence | Notes |
|---|---|---|---|---|---|---|
| Ethereum mainnet (chain ID 1) | 1 | 5 | 2 (12s blocks) | 5 | 5 | [Currently the default in fileverse-storage-v2](https://github.com/fileverse/fileverse-storage-v2 "Source: Fileverse fileverse-storage-v2 README, retrieved 2026-04-27"). High symbolic value; gas cost makes per-document writes [routinely $1–$10+ at observed gas prices](https://etherscan.io/gastracker "Source: Etherscan Gas Tracker, retrieved 2026-04-27"). |
| Gnosis Chain (chain ID 100) | 5 | 4 | 4 (~5s blocks) | 4 | 5 | [Gnosis Chain is referenced in the collaboration-server RPC setup](https://github.com/fileverse/collaboration-server "Source: Fileverse collaboration-server README, retrieved 2026-04-27"). Low fixed-fee model (xDAI gas) keeps per-document anchoring sub-cent. |
| Base (chain ID 8453) | 4 | 4 | 4 (2s blocks) | 4 | 5 | L2 rollup with growing dApp footprint; gas typically <$0.05. Good consumer wallet support. Not currently observed in Fileverse repos. |
| Arbitrum One (chain ID 42161) | 4 | 4 | 5 (sub-second sequencer) | 4 | 5 | Rollup with deepest TVL among Ethereum L2s. Per-document anchoring typically <$0.10. Not currently observed in Fileverse repos. |
| Polygon PoS (chain ID 137) | 5 | 3 | 4 (2s blocks) | 3 | 5 | Lowest absolute cost but [reorg history and validator-set centralization concerns documented in 2023 incident reports](https://polygon.technology/blog/heimdall-recovery-and-our-path-forward "Source: Polygon Labs blog, 2023-03-15"). |

**Recommendation.** The current Ethereum-mainnet default is at odds with the document-heavy write pattern Fileverse encourages. Under a scenario where typical user flow involves more than a handful of document publishes per month, Ethereum mainnet's gas cost is a friction tax. We recommend defaulting new portals to **Gnosis Chain** (already partially integrated) for cost, with **Base** as a secondary option for users embedded in the Coinbase / Farcaster ecosystem. Mainnet should remain as an opt-in for portals where on-chain provenance must anchor to L1.

### IPFS provider comparison (current implementation)

| Provider | Trust model | Pricing observed | Geographic redundancy | Notes |
|---|---|---|---|---|
| [Pinata](https://www.pinata.cloud/pricing "Source: Pinata, retrieved 2026-04-27") | Centralized SaaS | Free tier 1 GB; paid from $20/mo | Multi-region | Largest IPFS pinning operator. |
| [Filebase](https://filebase.com/pricing "Source: Filebase, retrieved 2026-04-27") | Centralized with Sia/Storj backing | Pay-as-you-go from $0.0059/GB | Multi-cloud | S3-compatible API. |
| [Web3.Storage](https://web3.storage "Source: Web3.Storage, retrieved 2026-04-27") | Centralized (Storacha) with Filecoin cold storage | Free tier; paid plans | Multi-region | Underwent a [governance/operations transition to Storacha in 2024](https://blog.web3.storage/posts/the-data-layer-is-here-with-the-new-web3-storage "Source: Web3.Storage blog, 2024-04-15"). |

The empirical IPFS reliability picture is documented in [Trautwein et al., SIGCOMM 2022](https://dl.acm.org/doi/10.1145/3544216.3544232 "Trautwein et al., Design and Evaluation of IPFS, SIGCOMM 2022"): retrieval latency from cold IPFS providers can be tens of seconds at the long tail. The three-provider fallback is the right architectural response to this empirical picture, but it does not address the case of all three providers ceasing operations simultaneously (an operational rather than technical risk).

### Collaboration relay topology

The current single-relay model with [Waku-based discovery on the `feat/waku` branch marked early alpha and highly experimental](https://github.com/fileverse/collaboration-server "Source: Fileverse collaboration-server README, retrieved 2026-04-27") is a sensible staged plan. A federated relay model (multiple community-hosted servers connected via Waku gossip) would distribute liveness risk; a pure P2P model (libp2p direct without a relay) would eliminate the relay but lose NAT-traversal reliability in browsers.

---

## Section 5: Integration Layer

External dependencies grouped by trust model and operational impact.

| Integration | Mechanism | Trust model | Latency | Failure mode |
|---|---|---|---|---|
| **Pinata IPFS** | REST upload + pin | Trusted custodian for liveness; cannot read ciphertext | <1s upload typical | If down, fallback to Filebase |
| **Filebase IPFS** | S3-compatible API | Same as Pinata | <1s typical | Fallback to Web3.Storage |
| **[Web3.Storage / Storacha](https://web3.storage "Source: Web3.Storage, retrieved 2026-04-27")** | UCAN + w3up client | Trusted custodian; UCAN allows scoped delegation | Variable; cold-tier Filecoin can be minutes | Manual retry / self-hosted IPFS |
| **Ethereum mainnet RPC** | JSON-RPC via wallet provider or hosted endpoint | Wallet-trusted; many provider options | ~12s confirmation | Switch RPC; chain itself rarely halts |
| **Gnosis Chain RPC** ([referenced via QuickNode in collab-server](https://github.com/fileverse/collaboration-server "Source: Fileverse collaboration-server README, retrieved 2026-04-27")) | JSON-RPC | Provider-trusted | ~5s confirmation | Switch RPC |
| **UCAN issuer** (storage service) | Server-issued bearer token | Trusted issuer per portal | ms | Auth re-challenge |
| **Penumbra encryption library** | npm dependency | Supply-chain trust on package registry | n/a | Lockfile pinning + audit |
| **TweetNaCl.js** | npm dependency | Widely audited; [has formal test vectors](https://tweetnacl.js.org "Source: TweetNaCl.js, retrieved 2026-04-27") | n/a | Same as above |
| **Y.js** | npm dependency | Widely deployed (e.g., JupyterLab, AFFiNE) | n/a | Same as above |
| **TipTap / ProseMirror** | npm dependency for the dDocs editor surface | Stable, widely deployed | n/a | Same as above |
| **Fortune Sheet** | Forked spreadsheet engine ([fileverse/fortune-sheet](https://github.com/fileverse/fortune-sheet "Source: Fileverse fortune-sheet repo, retrieved 2026-04-27")) | In-tree fork; controlled supply chain | n/a | Maintainership commitment |
| **Waku** ([early-alpha relay discovery](https://github.com/fileverse/collaboration-server "Source: Fileverse collaboration-server README, retrieved 2026-04-27")) | libp2p gossip network | Decentralized; experimental | Variable | Fall back to direct WS URL |
| **viem** | EVM RPC client used by [`@fileverse-dev/ddoc`](https://github.com/fileverse/fileverse-ddoc "Source: Fileverse ddoc README, retrieved 2026-04-27") | Audited TypeScript Ethereum lib | n/a | npm pinning |

The dependency on three pinning providers and at least two RPC providers is a deliberate diversification at the operational layer. The supply-chain attack surface (npm packages) is conventional for a TypeScript stack; lockfile review and Sigstore-style provenance would strengthen it but are not currently observed.

---

## Section 6: Contract / Module Architecture

Component breakdown across the four observable services.

### 6.1 `@fileverse-dev/ddoc` (client editor)

| Attribute | Value |
|---|---|
| Purpose | TipTap-based rich-text editor with offline IndexedDB sync and optional collaboration |
| License | [AGPL-3.0](https://github.com/fileverse/fileverse-ddoc "Source: Fileverse ddoc README, retrieved 2026-04-27") |
| Upgradeability | npm semver via `@fileverse-dev/ddoc` (current v3.2.9 as of 2026-04-27) |
| Key props | `enableCollaboration`, `enableIndexeddbSync`, `collaborationId`, `ddocId` |
| Dependencies | `@fileverse/crypto >=0.0.21`, `viem >=2.13.8`, `@fileverse/ui >=5.0.0`, `framer-motion`, `@dnd-kit/core` |

### 6.2 `fileverse-dsheet` (client spreadsheet)

| Attribute | Value |
|---|---|
| Purpose | Spreadsheet UI with onchain read/write cells |
| License | [AGPL-3.0](https://github.com/fileverse/fileverse-dsheet "Source: Fileverse fileverse-dsheet README, retrieved 2026-04-27") |
| Upgradeability | App-level deployment at [dsheets.new](https://dsheets.new "Source: Fileverse, retrieved 2026-04-27") |
| Key features | Fortune-Sheet engine, simulate + submit transactions (V0.3) |
| Open question | Specific chains supported — README does not enumerate as of 2026-04-27 |

### 6.3 `collaboration-server` (relay)

| Attribute | Value |
|---|---|
| Purpose | Stateless WebSocket relay for encrypted Yjs updates |
| License | [MIT](https://github.com/fileverse/collaboration-server "Source: Fileverse collaboration-server README, retrieved 2026-04-27") |
| Language | TypeScript (100%) |
| Key endpoint | `/documents/update` accepting `encrypted_yjs_update` payloads |
| Statelessness | Memory-only session state; flushed to IPFS on session close |
| Audit | Dedalo audit in progress; no findings yet published |

### 6.4 `fileverse-storage-v2` (pin gateway + UCAN issuer)

| Attribute | Value |
|---|---|
| Purpose | Multi-provider IPFS pinning with UCAN-gated authorization and on-chain registry mediation |
| License | [ISC](https://github.com/fileverse/fileverse-storage-v2 "Source: Fileverse fileverse-storage-v2 README, retrieved 2026-04-27") |
| Stack | Node.js v16+, Express, MongoDB, TypeScript |
| Auth | Bearer UCAN + headers `Contract Address`, `Invoker Address`, `Chain ID` |
| Providers | Pinata (api key), Filebase (access/secret), Web3.Storage (token) |

### 6.5 Implied registry contract (interface sketch)

The on-chain side is not in a public Fileverse repository as of this review; the interface below is reconstructed from the request headers required by `fileverse-storage-v2` and the lock model in `walk-away`.

```solidity
interface IFileverseRegistry {
    function registerFile(bytes32 fileId, bytes calldata cid,
                          bytes32 ownerLock, bytes32 portalLock, bytes32 linkLock) external;
    function updateCid(bytes32 fileId, bytes calldata newCid) external;
    function rotateLocks(bytes32 fileId, bytes32 ownerLock, bytes32 portalLock, bytes32 linkLock) external;
    function getFile(bytes32 fileId) external view returns (
        bytes memory cid, address owner,
        bytes32 ownerLock, bytes32 portalLock, bytes32 linkLock, uint64 updatedAt);

    event FileRegistered(bytes32 indexed fileId, address indexed owner);
    event CidUpdated(bytes32 indexed fileId, bytes cid);
    event LocksRotated(bytes32 indexed fileId);
}
```

**Upgradeability stance.** A proxy-based upgrade pattern (e.g., UUPS) on the registry would let the team evolve the lock-rotation logic without breaking existing fileIds; immutable deployment improves user assurance but loses the option to fix integration bugs. Under a scenario in which the registry must respond to changes in the encryption library (e.g., post-quantum migration), upgradeability is load-bearing.

### 6.6 `walk-away` (export tool)

| Attribute | Value |
|---|---|
| Purpose | Standalone retrieval and decryption from IPFS using locally held keys |
| License | [AGPL-3.0](https://github.com/fileverse/walk-away "Source: Fileverse walk-away README, retrieved 2026-04-27") |
| Crypto | RSA (key pairs), AES (file keys via Penumbra), TweetNaCl (signing) |
| Lock unlock paths | Portal Lock, Owner Lock, Link Lock — any one suffices |

---

## Section 7: Attack Vectors & Mitigations

≥5 vectors specific to Fileverse's mechanism (not generic web app issues).

### 7.1 Compromised collaboration relay → traffic analysis and denial of service (MODERATE)

Even though the relay is cryptographically blind to content, it sees room IDs, participant counts, message timing, and message sizes. An adversary controlling the relay (or sniffing TLS metadata) can fingerprint documents and infer activity patterns. Severity is bounded by the absence of plaintext but real for journalistic and legal use cases. **Mitigation:** padding to fixed message sizes; running through Tor or a randomized Waku peer selection; encouraging self-hosted relays for high-sensitivity tenants.

### 7.2 Lock-key leakage via shareable Link Lock (HIGH)

The [Link Lock model](https://github.com/fileverse/walk-away "Source: Fileverse walk-away README, retrieved 2026-04-27") encodes the file key under a link key embedded in the URL fragment. Anyone with the link decrypts the file; common URL leakage paths (referrer headers, copy-paste into chat apps, search-bar autosuggest, browser history sync) generalize the audience beyond the intended recipients. Severity is high because the user-visible UX ("share via link") encourages exactly the leakage paths most likely to materialize. **Mitigation:** time-bounded link keys with on-chain expiry; per-link revocation by lock rotation; explicit user warning when "anyone with the link" mode is selected; option to require wallet attestation in addition to link key.

### 7.3 IPFS pinning provider collusion or simultaneous failure (MODERATE)

All three current pin providers ([Pinata, Filebase, Web3.Storage](https://github.com/fileverse/fileverse-storage-v2 "Source: Fileverse fileverse-storage-v2 README, retrieved 2026-04-27")) are centralized SaaS operations subject to legal compulsion in their respective jurisdictions. Simultaneous deplatforming or coordinated subpoena would render content unavailable even though it remains decryptable in principle. The [empirical IPFS reliability findings of Trautwein et al.](https://dl.acm.org/doi/10.1145/3544216.3544232 "Trautwein et al., SIGCOMM 2022") suggest that the public IPFS DHT is unreliable as a sole fallback. **Mitigation:** Filecoin cold-storage deals as a fourth tier; user-configurable self-hosted IPFS pinning; encryption-preserving cross-pinning to ARweave for permanence under a different jurisdictional surface.

### 7.4 Registry CID poisoning by a compromised owner key (HIGH)

The on-chain registry treats the `owner` address as authoritative for `updateCid`. Compromise of the owner's wallet private key allows an attacker to replace the CID with a malicious encrypted blob. Because the relay does not authenticate authorship of CRDT updates (Section 1, Option A), and because the locks contain the symmetric key, an attacker who also obtains the file_key can write coherent malicious history. **Mitigation:** multi-sig portal owners (e.g., Safe on Gnosis); on-chain timelocks for CID updates; client-side detection of "CID changed in unexpected window" with user re-confirmation; the Option-B per-update signature design from Section 1.

### 7.5 Penumbra / TweetNaCl supply-chain attack (MODERATE)

The encryption stack [composes Penumbra (AES key generation) with TweetNaCl (signing) and RSA (lock wrapping)](https://github.com/fileverse/walk-away "Source: Fileverse walk-away README, retrieved 2026-04-27"). A poisoned npm version of any of these — or of a transitive dependency — would silently exfiltrate keys. **Mitigation:** pinned lockfiles audited at release; Subresource Integrity for the loaded JS; reproducible builds; publishing release artifacts via Sigstore-equivalent signed provenance. The walk-away repository's existence as a separate, minimal-dependency tool already mitigates this for the export path.

### 7.6 Stateless relay denial-of-service via session flooding (MODERATE)

The collaboration server holds [session state ephemerally in memory](https://github.com/fileverse/collaboration-server "Source: Fileverse collaboration-server README, retrieved 2026-04-27"). An attacker opening many concurrent ciphertext-spam sessions can exhaust memory because the server cannot inspect content to apply rate limits. **Mitigation:** per-IP and per-UCAN connection caps; session size budgets enforced by the relay even on encrypted payloads; CAPTCHA / proof-of-work gating for unauthenticated session creation.

### 7.7 vOPRF-ID identity loss without recovery path (MODERATE)

vOPRF-ID, the username-based authentication mode, derives credentials from local key material. If the user clears browser storage without a wallet backup, no recovery path is documented. Severity depends on how often a single user mode is the only access route to a document; the walk-away repository explicitly addresses the *content* recovery path but not the *identity* recovery path. **Mitigation:** mandatory wallet attestation pairing for any portal owner role; encrypted recovery share to an email-controlled address (à la Skiff's pre-shutdown model); social recovery via Safe / multi-sig.

### 7.8 dSheets onchain-write mis-targeting (HIGH)

The dSheets [V0.3 "simulate + submit transactions" feature](https://github.com/fileverse/fileverse-dsheet "Source: Fileverse fileverse-dsheet README, retrieved 2026-04-27") moves transaction parameters into spreadsheet cells. A formula-induced cell mutation — including adversarial CSV import or VLOOKUP into untrusted data — can change a transaction destination or amount before signing. **Mitigation:** mandatory simulate-and-display step with diff against last-signed parameters; explicit "this cell feeds a transaction" annotation that disables formulas; treating onchain-write cells as a different cell type with read-only inputs.

---

## Open architectural questions

We surface five for follow-up research:

1. The on-chain registry contract source is not in a public Fileverse repository; published source is needed to validate Section 6.5.
2. The exact wire format of `encrypted_yjs_update` (encryption mode, nonce derivation, AAD) is not documented in the public README; would require source-level review of `collaboration-server` to confirm the assumptions in Section 1, Option A.
3. The Dedalo audit scope (collab-server only, or also `@fileverse/crypto` and walk-away) determines how much of the security argument has third-party validation.
4. Whether dSheets enforces per-cell signing for transaction-bearing cells, or treats the spreadsheet trust boundary as the entire workbook, materially changes the severity of vector 7.8.
5. The local-LLM integration referenced on the homepage but not in any reviewed repository would constitute an additional supply-chain surface (model weights, runtime) requiring its own threat model.

---

## Citations index (selected)

- Fileverse repositories (verified via WebFetch 2026-04-27): [`fileverse-storage-v2`](https://github.com/fileverse/fileverse-storage-v2 "Source: Fileverse, retrieved 2026-04-27"), [`walk-away`](https://github.com/fileverse/walk-away "Source: Fileverse, retrieved 2026-04-27"), [`collaboration-server`](https://github.com/fileverse/collaboration-server "Source: Fileverse, retrieved 2026-04-27"), [`fileverse-ddoc`](https://github.com/fileverse/fileverse-ddoc "Source: Fileverse, retrieved 2026-04-27"), [`fileverse-dsheet`](https://github.com/fileverse/fileverse-dsheet "Source: Fileverse, retrieved 2026-04-27").
- Academic & protocol references: [Yjs (Nicolaescu et al., ICWE 2015)](https://link.springer.com/chapter/10.1007/978-3-319-19890-3_55), [JSON CRDT (Kleppmann & Beresford, 2016)](https://arxiv.org/abs/1608.03960), [Local-first (Ink & Switch, 2019)](https://www.inkandswitch.com/local-first/), [IPFS empirical evaluation (Trautwein et al., SIGCOMM 2022)](https://dl.acm.org/doi/10.1145/3544216.3544232), [secsync (Graf, 2023)](https://github.com/nikgraf/secsync), [Proton Docs encryption (Proton, 2024)](https://proton.me/blog/docs-proton-drive).
