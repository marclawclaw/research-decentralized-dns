# Research Summary: Decentralised DNS and Naming Systems

## Methodology

This synthesis covers 8 decentralised naming systems selected from a discovery pass of 16 candidates. Projects were identified through ecosystem aggregators (CoinMarketCap, Messari, DeFi Llama), academic literature (WEIS 2015 Namecoin study, CANS 2014 GNS paper, RFC 9498), developer documentation, and IETF standards databases. Selection prioritised architectural diversity: smart contract systems (ENS, Unstoppable Domains, SPACE ID), dedicated PoW blockchains (Handshake, Namecoin), DHT/P2P systems (GNS, IPNS), and hybrid approaches (D3/Doma). Each project was researched by a dedicated agent, then independently fact-checked via a PR review process. Research was conducted during April 2026.

## Ecosystem landscape

The decentralised naming space is dominated by smart-contract systems built on existing blockchains. Total registered names across the top 5 systems exceed 25 million, though "registered" and "active" differ significantly across protocols.

| Protocol | Registered names | Active/non-expired | Revenue model | Source |
|----------|-----------------|-------------------|---------------|--------|
| [[projects/handshake]] | 11.3M TLDs | [NOT FOUND] | Auction burn (62.5M HNS burned) | [Namebase stats](https://www.namebase.io/stats/) |
| [[projects/space-id]] | 6.7M+ | [NOT FOUND] | Registration fees + $ID token | [CoinMarketCap](https://coinmarketcap.com/cmc-ai/space-id/what-is/) |
| [[projects/unstoppable-domains]] | 4.6M+ Web3 + 750K DNS | [NOT FOUND] | One-time purchase | [Tracxn](https://tracxn.com/d/companies/unstoppable-domains/__5yK6TVAPTXnNb2zNMdcTpmVHN7NvV6ZGoWi9vQdwYz8) |
| [[projects/ens]] | ~2.8M cumulative (2022 peak) | ~910K | Annual fees ($28.77M in 2024) | [ENS DAO Revenue](https://discuss.ens.domains/t/ens-revenue-reports/20577/2) |
| [[projects/namecoin]] | 196,023 (2015 study; 28 non-trivial) | Near zero | 0.01 NMC per registration | [Kalodner et al. 2015](https://econinfosec.org/archive/weis2015/papers/WEIS_2015_kalodner.pdf) |
| [[projects/gns]] | [NOT FOUND] | [NOT FOUND] | None (no token) | Self-described "tiny" network |
| [[projects/ipns]] | [NOT FOUND] | [NOT FOUND] | None (no token) | ~25,000 DHT server nodes; ~550K clients |

ENS is the clear revenue leader ($28.77M in 2024) and the most deeply integrated naming system, with ~3.2 million daily resolution requests and 500+ wallet/dApp integrations. SPACE ID leads on raw registration count across chains but has weaker documentation of active usage. Handshake's 11.3M registrations are heavily skewed by speculative bulk registrations with minimal active SLD deployment.

The DHT-based systems (GNS, IPNS) have negligible user-facing adoption by comparison, but operate the largest and most mature peer-to-peer infrastructure: IPFS's Amino DHT has ~25,000 server nodes and ~550,000 client nodes as of early 2026.

## Project rankings

| Rank | Project | Ecosystem | Primary metric | Value | Source |
|------|---------|-----------|---------------|-------|--------|
| 1 | [[projects/ens]] | Ethereum | Annual protocol revenue | $28.77M (2024) | [ENS DAO Revenue](https://discuss.ens.domains/t/ens-revenue-reports/20577/2) |
| 2 | [[projects/space-id]] | Multi-chain (24+) | Registered domains | 6.7M+ | [CoinMarketCap](https://coinmarketcap.com/cmc-ai/space-id/what-is/) |
| 3 | [[projects/unstoppable-domains]] | Ethereum/Polygon | Web3 domains + DNS | 4.6M + 750K | [Tracxn](https://tracxn.com/d/companies/unstoppable-domains/__5yK6TVAPTXnNb2zNMdcTpmVHN7NvV6ZGoWi9vQdwYz8) |
| 4 | [[projects/handshake]] | Dedicated PoW chain | TLD registrations | 11.3M | [Namebase stats](https://www.namebase.io/stats/) |
| 5 | [[projects/ipns]] | IPFS/libp2p | DHT server nodes | ~25,000 | [ProbeLab](https://discuss.ipfs.tech/t/probelabs-notable-ipfs-performance-results-week-07-2026/20048) |
| 6 | [[projects/gns]] | GNUnet (chain-independent) | RFC standardisation | RFC 9498 (Nov 2023) | [RFC Editor](https://www.rfc-editor.org/rfc/rfc9498.html) |
| 7 | [[projects/d3-doma]] | Purpose-built L2 | Series A funding | $25M (Paradigm-led) | [PR Newswire](https://www.prnewswire.com/news-releases/d3-raises-25m-series-a-led-by-paradigm-announces-the-first-blockchain-for-internets-362m-domain-names-302362780.html) |
| 8 | [[projects/namecoin]] | Bitcoin fork (PoW) | Historical significance | First blockchain naming (2011) | [Wikipedia](https://en.wikipedia.org/wiki/Namecoin) |

## Architectural taxonomy

The 8 projects split into four distinct architectural categories:

### Smart contract based (ENS, Unstoppable Domains, SPACE ID)

Names are stored as on-chain state in smart contracts deployed on existing blockchains. Resolution queries the contract directly or via an RPC node. The blockchain provides consensus, Sybil resistance, and censorship resistance. All registration and update operations require gas fees on the host chain.

**Strengths:** highest adoption; deepest wallet integrations; composable with DeFi.
**Weaknesses:** chain-dependent; no query privacy (all state is public); gas costs for registration and updates.

### Dedicated blockchain (Handshake, Namecoin)

A purpose-built PoW blockchain acts as the canonical name registry. Handshake replaces the DNS root zone; Namecoin operates a parallel `.bit` TLD. Consensus is independent of any other chain.

**Strengths:** no dependency on another chain's governance or tokenomics; Handshake can replace the DNS root zone entirely.
**Weaknesses:** requires sustaining an independent mining ecosystem; low market cap creates 51% attack risk; no query privacy; requires custom resolver software.

### DHT/P2P (GNS, IPNS)

Names are stored in a distributed hash table without any blockchain. GNS uses GNUnet's R5N DHT with key blinding for privacy; IPNS uses the Kademlia-based Amino DHT. Both are chain-agnostic and token-free.

**Strengths:** no blockchain dependency; no token required; no gas fees; GNS provides query privacy by design; chain-agnostic.
**Weaknesses:** DHT records are ephemeral (IPNS: 48h max; GNS: publisher-defined expiry); publishers must stay online or delegate republishing; adoption is minimal; human-readable names require an additional layer.

### Hybrid (D3/Doma)

Bridges ICANN-governed DNS domains with on-chain tokenisation. The DNS record remains authoritative; a blockchain layer adds ownership tokens, DeFi composability, and ENS integration.

**Strengths:** works in existing browsers; ICANN-compliant; instant mainstream DNS compatibility.
**Weaknesses:** does not improve DNS censorship resistance or privacy; introduces compliance modules that enable forced revocation; centralised validator set at launch.

## Common patterns

Across all 8 projects, the following design patterns recur:

- **Two-step commit/reveal for anti-frontrunning** ([[patterns/ens-namehash]], [[patterns/tld-auction-mechanism]]): ENS, Handshake, and Namecoin all use a hash-then-reveal flow to prevent miners or mempool observers from sniping name registrations. This is a solved pattern that any naming system should adopt.

- **Self-certifying names** ([[patterns/self-certifying-names]]): IPNS and GNS both derive name identifiers from public keys, making verification possible without any trusted authority. This is the foundation for chain-agnostic naming.

- **DNS bridge mechanisms** ([[patterns/gns-dns-bridge]]): every alternative naming system must provide a path for legacy DNS clients. Approaches range from NSS plugins (GNS) to CCIP-Read gateways (ENS) to SPV resolvers (Handshake). The trust implications of each bridge mechanism differ significantly.

- **Name expiry as anti-squatting** ([[patterns/name-expiry-renewal-mechanism]], [[patterns/name-squatting-prevention]]): ENS (annual), Handshake (2-year), and Namecoin (~250 days) all require renewal. Unstoppable Domains is the notable exception with perpetual ownership, which eliminates squatting friction entirely.

- **Namespace hierarchy via delegation** ([[patterns/gns-zone-based-naming]], [[patterns/ens-wildcard-resolution]]): both ENS (via namehash hierarchy and wildcard resolution) and GNS (via PKEY/EDKEY delegation records) support arbitrary-depth sub-namespace delegation.

## Key differentiators

### GNS: query privacy without anonymity overlay

GNS's key-blinding scheme ([[patterns/gns-query-privacy]], [[patterns/gns-dht-name-resolution]]) is unique among all 8 systems. Records are stored under derived keys that are computationally unlinkable to the zone identity without knowledge of both the zone key and the specific label. Record data is symmetrically encrypted with the expiry timestamp baked into the IV, making expired records cryptographically inert. No other production naming system provides comparable query privacy.

### IPNS: transport-agnostic record distribution

IPNS records can be distributed via DHT, PubSub, HTTP (delegated routing), or any custom transport, using the same record format and verification logic ([[patterns/content-addressing-vs-name-addressing]]). This transport flexibility is unmatched by any other system and directly relevant to a Logos/Waku naming layer that must operate over multiple transport protocols.

### ENS: CCIP-Read as industry standard

ENS's CCIP-Read (EIP-3668) ([[patterns/ccip-read-offchain-resolution]]) has become the de facto standard for off-chain data retrieval with on-chain verification. Any new naming system that wants interoperability with the Ethereum ecosystem should consider CCIP-Read compatibility.

### Handshake: root zone replacement

Handshake is the only production system where the blockchain IS the root zone ([[patterns/root-zone-replacement]]). This is architecturally ambitious but faces the structural barrier that no major browser supports it natively.

### D3/Doma: ownership/control separation

The dual-token model (DOT for ownership, DST for DNS control) ([[patterns/dual-token-name-ownership]]) demonstrates that name ownership and operational control can be separated, enabling domain trading without DNS downtime. This pattern is relevant for any naming system that bridges on-chain and off-chain worlds.

## The DHT vs blockchain trade-off

This is the central architectural question for the RFP. The research reveals a clear trade-off space:

### What GNS and IPNS get right

1. **Privacy by design.** GNS's key-blinding scheme provides query privacy at the DHT level without requiring Tor or mix networks. No blockchain-based naming system can match this because all on-chain state is public and trivially enumerable.

2. **Chain-agnosticism.** Neither GNS nor IPNS depends on any specific blockchain. Zone creation is permissionless and free. There are no gas fees, no token requirements, and no governance dependencies.

3. **Self-certifying security.** Both systems derive their security from cryptographic key pairs, not from blockchain consensus or token economics. Verification requires no trusted third party.

4. **No squatting incentive (GNS).** GNS's petname system ([[patterns/gns-petname-system]]) eliminates the global human-readable namespace entirely, removing the economic incentive for name squatting by construction. This is the only naming system that solves the squatting problem at the architectural level rather than through economic mechanisms.

5. **RFC standardisation (GNS).** RFC 9498 gives GNS a formally specified wire format, cryptographic routines, and resolution procedure. No other decentralised naming system has achieved IETF standardisation.

### What they lack

1. **Adoption.** GNS has a self-described "tiny" network. IPNS DHT resolution has a median latency of ~11 seconds (August 2025). Neither system has achieved meaningful consumer adoption for naming.

2. **Human-readable global names.** GNS deliberately rejects a global human-readable namespace (Zooko's triangle). IPNS names are cryptographic hashes. Neither provides the "type a name and go" experience that DNS and ENS offer.

3. **Record persistence.** IPNS records expire after 48 hours; publishers must republish every ~4 hours. GNS records are republished by the ZONEMASTER daemon. Both require a running node for name availability, unlike blockchain systems where records persist indefinitely.

4. **Economic incentives for participation.** Neither system has a token or fee structure that rewards DHT participants. Network participation is altruistic. This limits the DHT's size and therefore its routing quality and privacy properties.

5. **Key revocation (IPNS).** IPNS has no standardised key revocation mechanism. GNS requires a computationally expensive proof-of-work revocation certificate that takes 4-5 days to compute.

### What blockchain systems get right

1. **Sybil resistance via economic cost.** Registration fees and auction mechanisms (ENS: $5-$640/year; Handshake: Vickrey auction burn) create real anti-squatting friction that DHT-based systems lack.

2. **Persistent records.** Once written on-chain, name records persist until explicitly updated or expired. There is no republishing burden.

3. **Human-readable global names.** ENS `.eth`, Handshake TLDs, and SPACE ID `.bnb`/`.arb` all provide globally unique, human-readable names.

4. **Ecosystem integrations.** ENS has 500+ integrations. Blockchain-based names compose with DeFi, NFT marketplaces, and wallet infrastructure.

### What blockchain systems lack

1. **Query privacy.** All on-chain state is public. ENS names, Handshake TLDs, and Namecoin `.bit` records are trivially enumerable. A 2025 ENS grant proposal for privacy-preserving resolution exists but has no production implementation.

2. **Chain-agnosticism.** ENS depends on Ethereum. Handshake has its own dedicated chain. SPACE ID uses a cross-chain oracle (Yoda) whose decentralisation is undocumented. None is truly chain-independent.

3. **Registration privacy.** Domain registrations on public blockchains link wallet addresses to name ownership permanently.

### The unsolved problem

**Globally unique human-readable names without a chain** remains an open design challenge. GNS resolves Zooko's triangle honestly (local petnames + global zTLDs), but this requires out-of-band key exchange for discovery. IPNS composes with ENS or DNSLink for human readability, but both introduce external dependencies.

A potential synthesis: a DHT-based naming system that supports multiple "registrar zones" (similar to GNS's FCFS registrar model) where human-readable names are registered within voluntarily chosen registrar namespaces. Each registrar zone is a trust delegation, not a protocol-level authority. This preserves chain-agnosticism and privacy while offering human-readable names within chosen trust contexts.

## Privacy analysis

| Privacy dimension | GNS | IPNS | ENS | Handshake | Unstoppable Domains | SPACE ID |
|-------------------|-----|------|-----|-----------|---------------------|----------|
| **Query privacy** (resolver cannot learn query) | Strong (key blinding) | Weak (DHT peers see key) | None (public chain) | Weak (SPV leaks TLD to full nodes) | None (public chain) | None (API sees all) |
| **Registration privacy** (who registered what) | Strong (no public registry) | Moderate (key hash only) | None (wallet-to-name public) | Moderate (blinded bids, pseudonymous) | None (wallet-to-name public) | None (wallet-to-name public) |
| **Zone enumeration resistance** | Strong (brute force required) | N/A (no zones) | None (all names on-chain) | None (all TLDs on-chain) | None (all names on-chain) | None (all names on-chain) |
| **Record confidentiality** | Yes (AES-256-CTR / ChaCha20) | No (plaintext records) | No (on-chain plaintext) | No (plaintext DNS data) | No (on-chain plaintext) | No (on-chain plaintext) |
| **Reverse resolution privacy** | N/A (no reverse resolution) | N/A | Weak (primary name is public opt-in deanonymisation) | N/A | Weak (on-chain ownership public) | Weak (on-chain ownership public) |

GNS is the clear privacy leader across every dimension. Its key-blinding scheme ([[patterns/gns-query-privacy]]) provides query privacy without an anonymity overlay, record encryption prevents DHT peers from reading stored data, and zone enumeration resistance makes offline namespace discovery computationally expensive.

The critical gap: GNS does not provide anonymity for the zone owner. A strong adversary monitoring P2P traffic can identify which peer publishes records for a given zone (RFC 9498, Section 10).

For the RFP, the minimum privacy baseline should be GNS-level query privacy (key blinding for DHT storage) and record encryption. Anonymity for zone publishers is a desirable but unsolved property.

## Problem data

### MEV and frontrunning in name registration

ENS mitigates frontrunning with a two-step commit/reveal process (1-minute wait between commit and reveal). Handshake uses blinded Vickrey auctions. Namecoin uses salted hash commitments. IPNS and GNS are not subject to frontrunning because name creation is permissionless and instant (any keypair is valid).

### Privacy leaks

- **ENS reverse resolution** ([[patterns/ens-reverse-resolution]]): setting a primary name publicly links a wallet address to a human-readable identity. Block explorers display this automatically.
- **CCIP-Read gateways** ([[patterns/ccip-read-offchain-resolution]]): gateway operators see every resolution query. The gateway URL is publicly visible on-chain.
- **Handshake SPV clients**: hnsd leaks queried TLD names to connected full nodes in plaintext. The hnsquery library partially mitigates this by sending hashed queries.
- **DNS2GNS gateways**: public GNS resolution gateways break the cryptographic chain of trust and expose queries to the gateway operator.

### Centralisation risks

| System | Centralisation vector | Impact |
|--------|----------------------|--------|
| ENS | CCIP-Read gateway operators; ENS DAO governance (token-weighted) | Gateway can censor queries; DAO can change protocol parameters |
| Unstoppable Domains | Whitelisted minting (company controls new issuance); Resolution Service API | Company can block new registrations; API can censor resolution |
| SPACE ID | Yoda oracle (cross-chain uniqueness, decentralisation undocumented) | Oracle failure blocks new registrations across all chains |
| D3/Doma | Initial validator set (D3 + Conduit + early partners); Compliance Module | Validators control chain; Compliance Module enables forced revocation |
| Handshake | Namebase marketplace (ownership unknown post Jan 2026 sale) | Ecosystem infrastructure depends on single marketplace |
| GNS | FCFS registrar (single operator for .pin.gns.alt discovery zone) | Discovery depends on registrar; record integrity is not compromised |

### Name squatting

Namecoin is the definitive case study ([[patterns/name-squatting-prevention]]): with a 0.01 NMC fixed registration fee, 99.985% of registered names were squatted or inactive (28 of 196,023 had non-trivial content, per the 2015 Princeton study). Effective anti-squatting requires either meaningful registration/renewal fees (ENS), auction-based pricing (Handshake), or elimination of the global human-readable namespace entirely (GNS).

### Name takeover risks

- ENS names that expire during the 90-day grace period can be re-registered, enabling reputation hijacking. Academic research documents this attack vector ([arXiv: ENS Good, Bad and Ugly](https://arxiv.org/pdf/2104.05185)).
- Key loss in any system (GNS, IPNS, ENS, Handshake, Namecoin) results in permanent, irrecoverable loss of the name.
- GNS requires pre-computed revocation certificates stored offline; failure to pre-compute means a compromised key cannot be revoked.

## Gaps and open questions

### For DHT-based approaches

1. **DHT incentive design.** Neither GNS nor IPNS provides economic incentives for DHT participants. How can a naming DHT sustain sufficient node count for routing quality and privacy without a token? Research into altruistic participation models (e.g., reciprocal storage agreements, reputation systems) is needed.

2. **Record persistence without republishing.** IPNS's 48-hour expiry and mandatory republishing is a significant operational burden. Can a DHT-based naming system provide DNS-equivalent persistence guarantees (years, not hours)? Hybrid approaches (DHT for routing, persistent storage on a subset of "pinning" nodes) need evaluation.

3. **Human-readable name discovery.** GNS's petname model eliminates squatting but requires out-of-band key exchange. What UX patterns can make zone key discovery frictionless? QR codes, social graph bootstrapping, and registrar zone discovery are potential paths.

4. **Resolution latency.** IPNS median resolution is ~11 seconds (August 2025 measurement). GNS's R5N DHT latency has not been formally measured on the current network. For interactive use cases, resolution must be sub-second. Caching, PubSub fast-paths, and delegated routing are known mitigations.

5. **Cross-chain address records in DHT.** Neither GNS nor IPNS natively supports multi-chain address resolution (store ETH, BTC, SOL addresses in a single record). This is a straightforward extension of the record format but has not been implemented.

6. **Post-quantum readiness.** GNUnet 0.26.x introduced post-quantum cryptographic layer work (Dec 2025). IPNS uses Ed25519 exclusively. The transition path for zone keys when post-quantum schemes mature is undefined.

### For the RFP broadly

7. **DANE without browser support.** Handshake demonstrates that DANE can replace CAs when anchored in a decentralised root of trust ([[patterns/dnssec-dane-integration]]), but Chrome does not support DANE. Is DANE relevant for application-layer naming (Logos/Waku apps) where the application controls TLS verification?

8. **Governance model.** ENS uses token-weighted DAO governance ([[patterns/ens-governance]]) with known plutocratic capture risks. GNS has no governance (each user is their own root). What governance model suits a Logos naming system?

9. **DNS migration path.** GNS provides AXFR/IXFR mirroring (Ascension), zonefile import, and live DNS zone scraping ([[patterns/gns-dns-bridge]]). Any production naming system needs comparable migration tooling for operators transitioning from legacy DNS.

## Recommendations for the RFP

Based on the evidence from all 8 projects, the RFP should address the following design decisions:

### 1. Adopt a DHT-based architecture with GNS-inspired privacy

GNS's key-blinding scheme is the strongest privacy mechanism observed across all systems. The RFP should require:
- Records stored under blinded keys in the DHT (zone enumeration resistant).
- Record data encrypted at rest in the DHT.
- Expiry enforced cryptographically (baked into the encryption IV, per GNS).
- No blockchain dependency for name resolution.

### 2. Layer human-readable names as optional registrar zones

Rather than choosing between "no global names" (GNS) and "chain-anchored names" (ENS), the RFP should specify a layered architecture:
- **Base layer:** self-certifying names (public key hashes), similar to IPNS and GNS zTLDs.
- **Discovery layer:** optional registrar zones (similar to GNS FCFS, but with anti-squatting mechanisms) where human-readable names map to zone keys. Multiple registrar zones can coexist; users choose which to trust.
- **Bridge layer:** DNS compatibility via dns2gns-style proxies and `.alt` TLD integration.

### 3. Solve record persistence

The 48-hour DHT expiry model (IPNS) is inadequate for a DNS replacement. The RFP should explore:
- Extended DHT TTLs with cryptographic expiry enforcement (GNS approach, but with longer periods).
- Persistent "pinning" nodes that commit to long-term record storage.
- Optional on-chain anchoring for high-value names (a name owner can optionally commit a record hash to a blockchain for persistence, without requiring resolution to go through the chain).

### 4. Require anti-squatting mechanisms for registrar zones

Namecoin's catastrophic squatting failure (99.985% inactive) demonstrates that fixed, low registration fees guarantee namespace pollution. For any human-readable registrar zone, the RFP should require:
- Meaningful registration and renewal fees (ENS model) or auction-based pricing (Handshake Vickrey model).
- Name expiry with grace period (ENS's 90-day grace period is a reasonable default).
- Clear dispute resolution path or explicit acceptance of squatting as a trade-off.

### 5. Design for multi-transport resolution

IPNS demonstrates that the same record format can be distributed via DHT, PubSub, HTTP, or custom transport. The RFP should:
- Define a single record format (protobuf or CBOR) with cryptographic verification independent of the transport.
- Support at minimum: DHT routing, Waku messaging, and HTTP delegated routing.
- Enable PubSub as a fast-path for real-time updates (IPNS's GossipSub model).

### 6. Provide DNS compatibility from day one

Every naming system that omitted DNS compatibility at launch (Namecoin, GNS, Handshake) faced adoption barriers. The RFP should require:
- A dns2gns-style DNS proxy for legacy applications.
- A GNS2DNS-style delegation record for handing off to legacy DNS.
- BOX record support for co-locating TLSA/SRV records to avoid extra DHT lookups.
- Use of the `.alt` pseudo-TLD (RFC 9476) for namespace disambiguation.

### 7. Address key management and revocation

Key loss is permanent and irrecoverable in every system studied. The RFP should:
- Require key rotation support (IPNS lacks this; GNS supports it via zone delegation).
- Define a revocation mechanism (GNS's PoW revocation certificate or a lighter alternative).
- Specify tooling for revocation certificate pre-computation and secure offline storage.
- Consider social recovery or multisig zone ownership as extensions.

### 8. Specify an SDK and integration path

SPACE ID's Web3 Name SDK (auto-discovery of new TLDs, zero-configuration resolution) and ENS's Resolution libraries demonstrate the importance of developer experience. The RFP should require:
- A resolution SDK (JavaScript/TypeScript at minimum) with zero-configuration defaults.
- Auto-discovery of new registrar zones.
- Support for reverse resolution (address to name) with explicit opt-in and privacy warnings ([[patterns/ens-reverse-resolution]]).

## Related notes

- [[patterns/gns-dht-name-resolution]] — core DHT resolution with key blinding
- [[patterns/gns-query-privacy]] — privacy analysis of key blinding
- [[patterns/gns-petname-system]] — petnames and Zooko's triangle
- [[patterns/self-certifying-names]] — IPNS self-certifying architecture
- [[patterns/ccip-read-offchain-resolution]] — ENS off-chain resolution standard
- [[patterns/name-squatting-prevention]] — anti-squatting mechanisms
- [[patterns/dht-record-expiry]] — DHT record persistence
- [[patterns/gns-dns-bridge]] — DNS compatibility
- [[metrics/dht-vs-blockchain-naming]] — architectural comparison
- [[metrics/naming-protocol-adoption]] — adoption metrics
