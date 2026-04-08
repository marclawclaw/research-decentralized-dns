---
tags: [project, ipfs, naming, dht, libp2p, p2p]
ecosystem: IPFS/libp2p
category: Infrastructure
website: https://docs.ipfs.tech/concepts/ipns/
docs: https://specs.ipfs.tech/ipns/
launched: 2016
---

# IPNS (InterPlanetary Name System)

IPNS is the mutable naming layer of IPFS, providing self-certifying, updateable pointers to content-addressed data. An IPNS name is a cryptographic hash of a public key; whoever holds the corresponding private key can publish signed records redirecting that name to any CID (Content Identifier), making it possible to share a stable address whose target can change over time without any central authority.

IPNS is transport-agnostic: records may be routed and resolved via the Kademlia-based Amino DHT, via libp2p GossipSub PubSub channels, via the Delegated Routing HTTP API, or through DNS TXT records (DNSLink). Human-readable names are layered on top through DNSLink (mapping DNS domains to IPNS names) and ENS (mapping `.eth` names to IPFS/IPNS content hashes).

## Adoption metrics

| Metric | Value | Date | Source |
|--------|-------|------|--------|
| Amino DHT server node count (stabilised) | ~25,000 | February 2026 | https://discuss.ipfs.tech/t/probelabs-notable-ipfs-performance-results-week-07-2026/20048 |
| Amino DHT client node count | ~550,000+ (spike from ~350k observed) | February 2026 | https://discuss.ipfs.tech/t/probelabs-notable-ipfs-performance-results-week-07-2026/20048 |
| IPNS median resolution latency (DHT, quorum 16) | ~11 seconds (p50); up to 37–60s outliers | August 2025 | https://www.probelab.network/blog/ipns-performance-amino-dht |
| IPNS p50 rolling-window latency range | 7–11 seconds (with peaks to 15–20s) | August 2025 | https://www.probelab.network/blog/ipns-performance-amino-dht |
| IPNS DHT record max TTL (enforced by network) | 48 hours | Ongoing | https://specs.ipfs.tech/ipns/ipns-record/ |
| IPNS republish interval (Kubo default) | Every 4 hours | Ongoing | https://github.com/ipfs/kubo/blob/master/docs/config.md |
| DHT quorum for IPNS resolution | 16 responses from distinct peers | Ongoing | https://www.probelab.network/blog/ipns-performance-amino-dht |
| Default IPNS record lifetime (Kubo, since v0.24) | 48 hours (increased from 24h in v0.24) | 2023 | https://github.com/ipfs/kubo/releases/tag/v0.24.0 |
| Default IPNS record TTL hint (Kubo, since v0.24) | 1 hour (increased from 1 minute in v0.24) | 2023 | https://github.com/ipfs/kubo/releases/tag/v0.24.0 |
| Delegated routing P95 latency improvement (Someguy v0.7+) | ~30% reduction (~560ms faster) | 2025 | https://blog.ipfs.tech/2025-delegated-routing-caching/ |
| Delegated routing: additional peer lookups eliminated | ~83% fewer extra lookups | 2025 | https://blog.ipfs.tech/2025-delegated-routing-caching/ |
| Registered ENS names (proxy for IPFS/IPNS web3 naming demand) | [NOT FOUND] | — | — |
| Number of active IPNS names on the network | [NOT FOUND] | — | — |

## How it works

### User perspective

A user generates a keypair and derives their IPNS name from the public key hash (expressed as a CIDv1 with the `libp2p-key` multicodec, encoded in base36 by convention). They then publish a signed record pointing that name to a CID:

```
ipfs name publish --key=mysite /ipfs/bafybeig...
```

Anyone who knows the IPNS name (e.g. `/ipns/k51qz...`) can resolve it to the current CID without trusting a central server; they simply verify the cryptographic signature embedded in the record. The name itself acts as the public-key fingerprint, so no certificate authority is needed.

To give users a human-readable address, two approaches exist:
1. **DNSLink:** publish a DNS TXT record at `_dnslink.example.com` with value `dnslink=/ipns/k51qz...`. IPFS gateways and resolvers then serve `example.com` by first resolving the IPNS name.
2. **ENS:** store the IPNS name (or a raw IPFS CID) as the `contenthash` field of an ENS `.eth` name. Browsers with ENS-aware resolvers (or IPFS Companion) redirect `alice.eth` to the resolved content.

### Protocol perspective

**Record structure.** An IPNS record is serialised as a protobuf `IpnsEntry` with an extensible DAG-CBOR payload in the `data` field. Key fields:

| Field | Purpose |
|-------|---------|
| `value` | The target path (e.g. `/ipfs/bafybei...`) |
| `sequence` | Monotonically increasing counter; resolvers pick the highest-sequence valid record |
| `validity` | ISO 8601 expiry timestamp (e.g. 48 hours from publication) |
| `ttl` | Cache hint in nanoseconds; default 1 hour since Kubo v0.24 |
| `signatureV2` | Ed25519 signature over the DAG-CBOR `data` field |
| `pubKey` | Optional embedded public key (required if not derivable from the name) |

The Kademlia key for DHT storage is `SHA2-256(/ipns/<binary-key>)`.

**Resolution via DHT.** The resolver queries the Amino DHT using a Kademlia lookup for the 20 closest peers to the IPNS key. It collects responses until it has at least 16 distinct replies (the quorum). The record with the highest valid sequence number wins. Any peer that returned an outdated record is then corrected with the fresh record (write-back). The full round trip takes a median of ~11 seconds under current network conditions (August 2025 measurement).

**Resolution via PubSub.** Each IPNS name maps to a unique GossipSub topic with the format `/record/<base64url-unpadded(/ipns/<binary-key>)>`. When a publisher posts a new record, all subscribed peers receive it in near-real time. The initial subscription requires a DHT lookup to discover interested peers (slow first time); subsequent updates propagate quickly. PubSub trades availability (no guarantee of delivery) for lower steady-state latency compared with DHT polling.

**Delegated routing.** Clients that cannot run a full DHT node (browsers, mobile) can use the Delegated Routing V1 HTTP API (served by Someguy at `delegated-ipfs.dev`). The server proxies DHT and IPNI queries and caches peer addresses. Since Someguy v0.7.0 (2025), a cached address book and active peer probing reduce P95 latency by ~30% and eliminate ~83% of additional peer lookups.

**Record expiry and republishing.** DHT peers store IPNS records for at most 48 hours regardless of the `validity` field. Publishers must republish every ~4 hours (Kubo default) to keep records alive. Failure to republish causes the name to resolve to nothing after expiry.

## Key behaviours

| Behaviour | Detail |
|-----------|--------|
| Self-certifying names | Name = hash of public key; record contains signature; anyone can verify without a trusted third party |
| Mutable pointer to immutable content | IPNS name stays constant; the CID it points to can change with every update |
| Sequence-number conflict resolution | Highest sequence number wins; DHT write-back propagates the freshest record to stale nodes |
| 48-hour DHT expiry | Records disappear from DHT if not republished; no persistent storage guarantee |
| PubSub as fast-path alternative | GossipSub delivers updates in near-real time to subscribed peers, bypassing the slow DHT round trip |
| DNSLink as human-readable bridge | A standard DNS TXT record maps a domain name to an IPNS name, enabling browser-native resolution via conventional DNS |
| Key loss means name loss | There is no recovery mechanism; losing the private key permanently loses control of the name |
| No revocation mechanism | There is currently no standardised scheme to revoke an IPNS key; a stolen key grants permanent control |

## Architecture decisions

**Self-certifying PKI namespace.** IPNS derives from the Self-certifying File System (SFS) model: the name is the public key hash, so no external authority (DNS root, blockchain) is required to bind name to key. This makes IPNS fully permissionless and infrastructure-independent.

**Kademlia DHT for global state.** Using the same Amino DHT as IPFS content routing gives IPNS a globally shared, replication-redundant record store with no single point of failure. The trade-off is that DHT lookups are inherently slow (multiple network round trips to collect a quorum), giving the ~11s median resolution time.

**Dual routing (DHT + PubSub).** PubSub was added as an opt-in fast path for clients that subscribe to specific IPNS names. DHT favours consistency (guaranteed to find the latest record given enough peers); PubSub favours availability and low latency (works even if the DHT is congested, but delivery is not guaranteed).

**Ed25519 as default key type.** IPNS implementations must support Ed25519 (RFC 8032). Ed25519 provides 128-bit equivalent security with 256-bit public keys and 512-bit signatures, and is fast to verify. RSA, Secp256k1, and ECDSA are also supported for compatibility.

**DAG-CBOR for extensibility.** The inner record payload uses deterministic CBOR (DAG-CBOR) inside a protobuf envelope. This allows adding new record fields (e.g. extra metadata) without breaking existing parsers, as the protobuf outer layer is stable.

**TTL as cache hint (since v0.24).** Before Kubo v0.24, the TTL field was ignored and a hardcoded 1-minute resolution cache was used. From v0.24, publishers control caching behaviour: a short TTL forces resolvers to re-query more frequently; a long TTL allows aggressive HTTP gateway caching.

## Differentiators

- **No blockchain dependency.** IPNS is chain-independent; it requires only libp2p connectivity and does not involve gas fees, on-chain transactions, or consensus round trips.
- **Zero-cost name creation.** Any keypair generates a valid IPNS name instantly. There are no registration fees, no auctions, and no namespace gatekeepers.
- **Self-certifying security model.** The security of the name binding reduces to the security of the underlying asymmetric key, not to the security of a smart contract or DNS registrar.
- **Layered human-readability.** IPNS does not compete with DNS or ENS; it composes with both. DNSLink makes IPNS names accessible via conventional domain names. ENS makes IPNS content accessible via `.eth` names. The name layer and the routing layer are decoupled.
- **Transport flexibility.** The same IPNS record can be distributed via DHT, PubSub, HTTP (delegated routing), or any custom routing system, using the same record format and verification logic.

## Limitations and criticisms

**Slow DHT resolution.** Median resolution latency is ~11 seconds (quorum 16, Amino DHT, August 2025). Outliers extend to 37–60 seconds. This is orders of magnitude slower than DNS (typically <100ms) and makes IPNS unsuitable for interactive use cases requiring fast name resolution. Community characterisation: "IPNS is very reliable, but very slow." [Source: https://www.probelab.network/blog/ipns-performance-amino-dht]

**Mandatory republishing.** Because DHT peers only store records for 48 hours, publishers must republish every ~4 hours. If a publisher goes offline, the IPNS name eventually resolves to nothing. This creates an operational burden that does not exist with traditional DNS (which stores records persistently at authoritative nameservers).

**No key revocation.** There is currently no standardised scheme for revoking a compromised IPNS key. A stolen private key grants permanent control over the name. Proposals exist (reserving sequence number `MaxUInt64` as a revocation sentinel) but are not yet part of the canonical spec. [Source: https://github.com/ipfs/specs/issues/219]

**No key rotation without losing the name.** Rotating to a new keypair means losing the original IPNS name, because the name is derived from the public key hash. Proposals for chained key rotation exist but are not standardised.

**Human-readable names require external systems.** IPNS names are long cryptographic identifiers (e.g. `k51qz...`). Usable human-readable names require either a DNS domain (centralised registrar, ICANN governance) or an ENS name (on-chain registration, gas fees). IPNS itself does not provide human-readable naming.

**PubSub delivery not guaranteed.** GossipSub does not guarantee message delivery. If a subscriber is offline when a publisher posts an update, and the update is not in the DHT (e.g. DHT record has expired), the subscriber may miss the update entirely.

**Namespace squatting impossible but also no global namespace.** Because every keypair yields a valid name, there is no global namespace to discover names by human-memorable strings. Name discovery requires out-of-band sharing of the IPNS identifier or a human-readable alias (DNSLink, ENS).

## Sources

- https://docs.ipfs.tech/concepts/ipns/ (IPFS Docs: IPNS concept, 2025)
- https://specs.ipfs.tech/ipns/ (IPFS Standards: IPNS overview, 2025)
- https://specs.ipfs.tech/ipns/ipns-record/ (IPFS Standards: IPNS Record and Protocol spec, 2025)
- https://specs.ipfs.tech/ipns/ipns-pubsub-router/ (IPFS Standards: IPNS PubSub Router spec, 2025)
- https://specs.ipfs.tech/routing/kad-dht/ (IPFS Standards: Kademlia DHT spec, 2025)
- https://www.probelab.network/blog/ipns-performance-amino-dht (ProbeLab: Measuring IPNS Performance on the Public Amino DHT, August 2025)
- https://discuss.ipfs.tech/t/probelabs-notable-ipfs-performance-results-week-07-2026/20048 (ProbeLab: Notable IPFS Performance Results Week 07, 2026)
- https://blog.ipfs.tech/2025-delegated-routing-caching/ (IPFS Blog: Faster P2P Retrieval with Delegated Routing Caching, 2025)
- https://github.com/ipfs/kubo/releases/tag/v0.24.0 (Kubo v0.24.0 release notes, 2023)
- https://docs.ipfs.tech/concepts/dnslink/ (IPFS Docs: DNSLink, 2025)
- https://eth-limo.gitbook.io/documentation/beginner/configuring-your-ens-name/content-hash-overview/understanding-content-hashes-ipns-and-ipfs-for-ens (eth.limo: IPNS and ENS content hashes, 2025)
- https://github.com/ipfs/specs/issues/219 (GitHub: IPNS Key Revocation issue, ongoing)
- https://github.com/ipfs/camp/blob/master/DEEP_DIVES/16-revocation-rotating-of-ipns-keys.md (IPFS Camp: Key revocation and rotation deep dive)
- https://arxiv.org/pdf/2105.08395 (Academic: Enabling self-verifiable mutable content items in IPFS, 2021)
