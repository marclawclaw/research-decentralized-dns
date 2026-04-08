# Research Index: Decentralised DNS and Naming Systems

Research coordinator discovery pass, April 2026.
Research focus: decentralised naming systems with emphasis on DHT-based and chain-agnostic architectures as alternatives to smart-contract-only incumbents.

---

## Architectural Categories

Projects are grouped by their primary resolution mechanism:

| Category | Description |
|----------|-------------|
| **Smart Contract** | Name records stored on-chain; resolution queries the blockchain |
| **UTXO/PoW Blockchain** | Name records in a dedicated PoW blockchain (Bitcoin-derived) |
| **DHT/P2P** | Name records distributed via a distributed hash table; no single chain |
| **Hybrid** | Combines on-chain ownership with traditional DNS or off-chain resolution |
| **Messaging-Layer** | Experimental naming over a P2P messaging stack |

---

## Discovered Projects

### 1. Ethereum Name Service (ENS)

| Field | Value |
|-------|-------|
| Ecosystem | Ethereum (mainnet + L2 via Namechain rollup) |
| Approach | Smart contract (registry + resolver pattern) |
| Primary metric | ~2.5M total registered names; ~910K active domains (late 2025) |
| Status | Active; ENSv2 / Namechain ZK-rollup launched Nov 2025 |
| Source | [Gate Learn ENS 2026](https://www.gate.com/en-us/learn/articles/ens-domains-explained-how-ethereum-name-service-is-powering-web3-identity-in-2026/12406); [The Block: Namechain](https://www.theblock.co/post/325381/ens-labs-introduces-own-l2-agnostic-rollup-namechain-aiming-for-launch-by-end-of-2025) |

How it works: two core smart contracts (Registry and Resolver) on Ethereum mainnet. The Registry maps each name to an owner, resolver address, and TTL. The Resolver holds the actual records (addresses, content hashes, text). ENSv2 / Namechain is a ZK-rollup built on the Taiko stack, enabling sub-$0.01 gas fees and sub-10-second cross-chain resolution via CCIP-Read. The Namechain approach was partially reverted by governance in Feb 2026 in favour of a mainnet-only ENSv2 deployment.

---

### 2. Unstoppable Domains

| Field | Value |
|-------|-------|
| Ecosystem | Ethereum + Polygon (L2) |
| Approach | Smart contract; NFT domains (ERC-721) |
| Primary metric | >4.2M registered Web3 domains; 750,000+ traditional DNS domains under management (March 2026) |
| Status | Active; ICANN-accredited registrar since August 2024; DNS registrar operations from October 2024 |
| Source | [Unstoppable Domains Wikipedia](https://en.wikipedia.org/wiki/Unstoppable_Domains); [Domain Name Wire growth](https://domainnamewire.com/2026/03/12/unstoppable-domains-growth-the-past-and-what-happens-in-the-future/) |

How it works: two smart contract architectures (legacy CNS on Ethereum L1; current UNS on Polygon). Domains minted as ERC-721 NFTs on Polygon for $0 gas. One-time purchase with no renewal fee. Supports 20+ Web3 TLDs including .crypto, .wallet, .nft, .x, .web3 and others. ICANN accreditation (August 2024) lets Unstoppable also sell traditional DNS TLDs (.com, etc.); DNS operations launched October 2024.

---

### 3. Handshake (HNS)

| Field | Value |
|-------|-------|
| Ecosystem | Dedicated PoW blockchain (Bitcoin-derived UTXO) |
| Approach | UTXO/PoW blockchain; TLD auction via Vickrey mechanism |
| Primary metric | 11.3M domain registrations; 62.5M HNS burned in auctions (as at 31 Jul 2024) |
| Status | Active; Namebase acquired by Namecheap |
| Source | [Namebase stats](https://www.namebase.io/stats/); [GeeksforGeeks HNS](https://www.geeksforgeeks.org/blogs/handshake-an-peer-to-peer-naming-system/) |

How it works: a dedicated PoW blockchain (10-minute blocks, UTXO model like Bitcoin) where every participant validates the root DNS zone. TLDs are auctioned via blind Vickrey auctions; the winner pays the second-highest bid. Name records use Bitcoin-style covenants (OPEN, BID, REVEAL, REGISTER, UPDATE, TRANSFER, FINALIZE, REVOKE). DNS record data lives in the blockchain; there is no DHT layer. Resolution requires a custom resolver (e.g. the hnsd light client or a HNS-aware DNS proxy). Namecoin-style merged mining is not used; Handshake uses its own PoW function (blake2b + sha3).

---

### 4. Namecoin

| Field | Value |
|-------|-------|
| Ecosystem | Dedicated PoW blockchain (Bitcoin fork) |
| Approach | UTXO/PoW blockchain; first blockchain naming system |
| Primary metric | Market cap ~$13.2M USD; low active usage of .bit domains [NOT FOUND: current domain count] |
| Status | Maintained but low adoption; original proof-of-concept |
| Source | [Namecoin.org](https://www.namecoin.org/); [Wikipedia Namecoin](https://en.wikipedia.org/wiki/Namecoin) |

How it works: Bitcoin fork (merged mining with Bitcoin) that adds a key-value namespace to the blockchain. The `d/` namespace provides .bit TLD DNS records. Records expire after ~36,000 blocks (~200 days) unless renewed; fee is 0.01 NMC per record. No built-in DHT; resolution requires a local node or an external proxy (e.g. ncdns daemon). Limited browser support is the primary adoption barrier.

---

### 5. GNU Name System (GNS) via GNUnet

| Field | Value |
|-------|-------|
| Ecosystem | GNUnet P2P network (chain-independent) |
| Approach | DHT-based; zone records signed by user keypairs |
| Primary metric | [NOT FOUND: adoption/node count] |
| Status | Active research/production; RFC 9498 standardised in 2023 |
| Source | [GNUnet GNS docs](https://docs.gnunet.org/latest/users/gns.html); [GNUnet GNS page](https://www.gnunet.org/en/gns.html) |

How it works: each user administers their own root zone, identified by a public key. Zone records are signed with the corresponding private key and published into GNUnet's Kademlia-variant DHT (a randomised Kademlia that remains efficient in small-world networks). Lookups traverse the DHT; the zonemaster service pushes updates. Records default to private (not DHT-published). DNS compatibility via DNS2GNS bridge, SOCKS proxy, and NSS plugin. Standardised in RFC 9498. Petnames allow user-assigned aliases to public zones. No blockchain involved; no token.

---

### 6. IPNS (InterPlanetary Name System) via IPFS/libp2p

| Field | Value |
|-------|-------|
| Ecosystem | IPFS / libp2p (chain-independent) |
| Approach | DHT-based (Kademlia); PKI namespace |
| Primary metric | [NOT FOUND: specific resolution query counts] |
| Status | Active; integral to IPFS; specification updated Nov 2025 |
| Source | [IPFS IPNS docs](https://docs.ipfs.tech/concepts/ipns/); [IPFS Kademlia DHT spec](https://specs.ipfs.tech/routing/kad-dht/) |

How it works: IPNS names are the hash of a public key (self-certifying). Records are signed by the private key holder and published to the Kademlia DHT (or PubSub for lower latency). Records expire after 48 hours and must be re-published. Resolution returns a pointer to a CID (content identifier), enabling mutable content addressing. IPNS is not a human-readable naming system on its own; ENS or DNSLink provide the human-readable layer on top. libp2p protocol ID format: `/<swarm-prefix>/kad/<version>`.

---

### 7. D3 (Doma Protocol)

| Field | Value |
|-------|-------|
| Ecosystem | Doma Protocol (purpose-built ICANN-compliant blockchain) |
| Approach | Hybrid: ICANN DNS + blockchain tokenisation; chain-agnostic ENS records |
| Primary metric | Targeting 362M+ existing DNS domains; $25M Series A (Jan 2025, led by Paradigm) |
| Status | Testnet phase (first registrar NicNames on testnet); ENS integration launched Oct 2025 |
| Source | [PR Newswire D3 Series A](https://www.prnewswire.com/news-releases/d3-raises-25m-series-a-led-by-paradigm-announces-the-first-blockchain-for-internets-362m-domain-names-302362780.html); [Messari D3](https://messari.io/report/d3-bridging-domains-to-web3) |

How it works: Doma Protocol is a blockchain purpose-built to meet ICANN compliance requirements, enabling traditional domain ownership to be mirrored on-chain as tokens. When an NFT representing a domain is transferred, domain ownership is updated simultaneously in both the DNS registrar and the blockchain. Name records are chain-agnostic at the ENS resolver level: a single name can reference addresses on Ethereum, L2s, Bitcoin, and Solana. Invested in by Coinbase Ventures, Polygon co-founder, and Namecheap CEO.

---

### 8. 3DNS

| Field | Value |
|-------|-------|
| Ecosystem | Ethereum (ERC-721 NFTs) + ICANN registrar back-end |
| Approach | Hybrid: ICANN DNS + on-chain smart contract ownership |
| Primary metric | [NOT FOUND: registered domain count] |
| Status | Active; first ICANN-integrated on-chain domain provider |
| Source | [3DNS website](https://3dns.box/); [3DNS comparison post](https://3dns.box/blog/posts/comparing-decentralized-dns-providers/) |

How it works: 3DNS partners with an ICANN-accredited registrar to register .com, .xyz, .io, .box, etc. traditionally, then tokenises each domain as an ERC-721 NFT on Ethereum. Domain ownership transfer = NFT transfer; both the DNS record and the blockchain state update atomically. Compatible with ENS protocol for Web3 resolution. Domains resolve in ordinary browsers (ICANN-compliant) without extensions, while also functioning as decentralised identities.

---

### 9. SPACE ID

| Field | Value |
|-------|-------|
| Ecosystem | BNB Chain, Arbitrum, Ethereum; 24+ blockchains |
| Approach | Smart contract; multi-chain name service |
| Primary metric | 6.7M+ registered domains; 330+ platform integrations (2025) |
| Status | Active; AI Agent Domains roadmap item for 2025 |
| Source | [CoinMarketCap SPACE ID](https://coinmarketcap.com/cmc-ai/space-id/what-is/); [SPACE ID docs](https://docs.space.id) |

How it works: universal name service deploying chain-specific registrar contracts on each supported chain (.bnb on BNB, .arb on Arbitrum, .eth via ENS delegation, etc.). A Name Service Protocol SDK/API abstracts multi-chain resolution for developers. Records are stored on each respective chain's smart contracts. The $ID token governs the protocol.

---

### 10. ZNS Connect

| Field | Value |
|-------|-------|
| Ecosystem | 50+ EVM-compatible chains |
| Approach | Smart contract; cross-chain domain service |
| Primary metric | 248,000+ domains minted; $762K revenue pre-token |
| Status | Active; token ($ZNS) pending launch |
| Source | [ZNS Connect website](https://znsconnect.io/); [Medium 2025 article](https://znsconnect.medium.com/web3-domains-in-2025-why-zns-connect-is-the-real-challenger-to-ens-space-id-sns-ba2ae923cd8b) |

How it works: deploys domain registrar contracts across 50+ EVM chains; a single domain can reference records on 100+ networks. Supports .hood (Robinhood network), .ink (Kraken L2), .defi (Unichain), .light (Pharos Network) and others. On-chain NFT ownership with cross-chain resolution via unified resolver API.

---

### 11. Solana Name Service (SNS / Bonfida)

| Field | Value |
|-------|-------|
| Ecosystem | Solana |
| Approach | Smart contract (Solana program) |
| Primary metric | 283,000+ registered .sol domains; 129,000+ unique holders; 150+ ecosystem partners |
| Status | Active; new SNS governance token launched May 2025 |
| Source | [Bonfida SNS docs](https://naming.bonfida.org/); [Gate crypto wiki SNS](https://dex.gate.com/crypto-wiki/article/exploring-sns-id-insights-into-blockchain-name-services-20251219) |

How it works: on-chain Solana program (name-service) maps .sol domains to Solana public keys and other records. Bonfida operates the primary registrar. Cross-chain via Wormhole bridge integration; browser integration via Brave. Airdrop of 40% of SNS token supply to early .sol domain holders (May 2025).

---

### 12. EmerDNS (Emercoin)

| Field | Value |
|-------|-------|
| Ecosystem | Emercoin blockchain (PoW+PoS) |
| Approach | UTXO blockchain (NVS name-value store) |
| Primary metric | [NOT FOUND: registered domain count] |
| Status | Active but niche; supports .emc, .lib, .coin, .bazar |
| Source | [EmerDNS page](https://emercoin.com/en/emerdns/); [Medium EmerDNS features](https://medium.com/@emer.tech/5-features-making-emerdns-the-only-truly-decentralized-dns-4c513bb13850) |

How it works: Emercoin's Name-Value Storage (NVS) extends the blockchain with arbitrary key-value records. The `dns` NVS prefix provides DNS records. Records can persist for up to 30 years (longer than Namecoin's 200-day maximum). Each wallet includes a built-in DNS server supporting RFC 1034 UDP DNS protocol. Supports four reserved TLD zones plus any custom zone.

---

### 13. Freename

| Field | Value |
|-------|-------|
| Ecosystem | Multi-chain (Polygon and others) |
| Approach | Smart contract; Web3 TLD registrar |
| Primary metric | [NOT FOUND: total registered domain count]; $6.5M Series A (Jul 2025) |
| Status | Active; ICANN-accredited; first ICANN-accredited Web3 namespace |
| Source | [Freename Series A news](https://www.thedomains.com/2025/07/23/freename-raises-6-5m-series-a-to-power-the-future-of-web3-domains-digital-identity/); [Freename Wikipedia](https://en.wikipedia.org/wiki/Freename) |

How it works: allows registration of custom TLDs and SLDs on-chain with no renewal fees. ICANN accreditation (shared with Unstoppable Domains) lets Freename also offer traditional DNS names. U.S. patent held for its Web3+DNS integration protocol.

---

### 14. Starknet ID

| Field | Value |
|-------|-------|
| Ecosystem | Starknet (Ethereum L2 ZK-rollup) |
| Approach | Smart contract (Cairo); .stark domain service |
| Primary metric | 239,000+ on-chain registered domains |
| Status | Active; primary identity layer for Starknet dApps |
| Source | [LFG Labs Starknet ID stats](https://lfglabs.dev/posts/starknetid); [Starknet ID website](https://starknet.id/) |

How it works: Cairo smart contracts resolve .stark names to Starknet addresses and arbitrary identity data. Integrated with Argent X, Braavos wallets and major Starknet DEXs. Interoperability via SNS (Solana Name Service) cross-chain matching.

---

### 15. Waku Naming Service (WNS) [Experimental]

| Field | Value |
|-------|-------|
| Ecosystem | Waku P2P messaging network (Logos stack) |
| Approach | P2P messaging layer; privacy-preserving naming |
| Primary metric | Experimental/hackathon prototype only |
| Status | Conceptual prototype; emerged from Waku internal hackathon (Sep 2025) |
| Source | [Waku hackathon blog](https://blog.waku.org/what-we-built-at-the-first-waku-internal-hackathon/) |

How it works: maps domain names to public keys (not wallet addresses or IPs); further communication is negotiated privately via Waku messaging. Avoids linking names to wallet addresses, preserving user privacy. Intended as a pseudonymous identity layer for Waku's marketplace infrastructure. No DHT or blockchain; resolution is mediated by the Waku messaging layer. Currently a proof-of-concept with no production deployment.

---

### 16. W3C Decentralised Identifiers (DID)

| Field | Value |
|-------|-------|
| Ecosystem | Cross-stack standard (W3C); multiple DID methods |
| Approach | Specification/standard; resolution depends on DID method |
| Primary metric | DID v1.0 published 2022; DID v1.1 in progress 2025 |
| Status | W3C Recommendation (v1.0); actively deployed across identity ecosystems |
| Source | [W3C DID 1.0](https://www.w3.org/TR/did-1.0/); [W3C DID 1.1](https://www.w3.org/TR/did-1.1/) |

How it works: DIDs are URIs in the form `did:<method>:<identifier>`. Resolution returns a DID Document containing public keys, service endpoints, and other metadata. DID methods vary: `did:web` uses traditional HTTPS/DNS; `did:ion` uses Bitcoin; `did:ethr` uses Ethereum; `did:plc` (used by AT Protocol/Bluesky) is self-certifying. Relevant to naming as a chain-agnostic identifier layer that can sit above any underlying resolution mechanism.

---

## Ranking by Relevance

Ranked by (a) architectural relevance to DHT/chain-agnostic design goal and (b) adoption/maturity.

| Rank | Project | Relevance reason | Adoption |
|------|---------|-----------------|----------|
| 1 | [[ens]] | Gold-standard smart-contract incumbent; ENSv2 crosschain architecture | High (2.5M+ names) |
| 2 | [[handshake]] | PoW blockchain replacing root DNS; closest to chain-native TLD model | Medium (11.3M domains) |
| 3 | [[gnu-name-system]] | Only production DHT-based system; RFC 9498 standard; directly relevant architecture | Low adoption but mature |
| 4 | [[ipns]] | DHT-based mutable naming over libp2p/IPFS; foundational infrastructure | Very high (IPFS ecosystem) |
| 5 | [[d3-doma]] | Chain-agnostic records; ICANN-compliant; $25M Paradigm-led; bridging Web2+Web3 | Early stage |
| 6 | [[unstoppable-domains]] | Major incumbent; 4.2M domains; one-time ownership model | High |
| 7 | [[space-id]] | Multi-chain smart contract incumbent; 6.7M domains | High |
| 8 | [[namecoin]] | Original blockchain naming system; first DHT-adjacent PoW DNS | Low (historical reference) |
| 9 | [[waku-naming-service]] | Logos-stack project; privacy-first; messaging-layer naming | Prototype only |
| 10 | [[3dns]] | Hybrid ICANN+on-chain; best Web2 compatibility | Early |

---

## Selected for Deep Research

The following 8 projects are selected for atomic-note deep research, covering the key architectural archetypes:

### Tier 1: Must research (highest signal)

1. **[[ens]]** - Ethereum Name Service
   - Rationale: dominant smart-contract incumbent; ENSv2 cross-chain architecture is the primary comparison point for any new naming RFP; rich public data

2. **[[handshake]]** - HNS
   - Rationale: only production system that replaces the DNS root zone; PoW-native TLD model; 11.3M domain registrations; important design contrast

3. **[[gnu-name-system]]** - GNUnet GNS
   - Rationale: the most mature pure-DHT naming system; RFC 9498 standardised; directly maps to the DHT-over-smart-contracts angle; no token/blockchain dependency

4. **[[ipns]]** - InterPlanetary Name System
   - Rationale: the DHT naming layer that IPFS billions of files rely on; Kademlia implementation details are directly reusable; active spec (updated Nov 2025)

### Tier 2: Strong supporting research

5. **[[d3-doma]]** - D3 / Doma Protocol
   - Rationale: only purpose-built ICANN-compliant blockchain for naming; chain-agnostic ENS records; $25M Paradigm investment signals institutional confidence in the space

6. **[[unstoppable-domains]]** - Unstoppable Domains
   - Rationale: second-largest smart-contract naming system; one-time ownership model is a meaningful design difference from ENS; ICANN accreditation path

7. **[[space-id]]** - SPACE ID
   - Rationale: largest multi-chain naming system (6.7M domains, 24+ chains); demonstrates how multi-chain resolution actually works in production

8. **[[namecoin]]** - Namecoin
   - Rationale: original blockchain naming system (2011); design choices and failure modes are directly informative for the RFP; merged PoW with Bitcoin

### Not selected (lower priority)

- **ZNS Connect**: similar to SPACE ID but smaller; covered by SPACE ID research
- **Freename**: ICANN-accredited Web3 TLD registrar; limited technical differentiation from Unstoppable Domains
- **Starknet ID**: chain-specific (Starknet only); limited architectural novelty
- **EmerDNS**: niche; low activity data available; covered by Namecoin and Handshake comparisons
- **Waku Naming Service**: prototype only; will be covered in the Logos context separately
- **W3C DID**: standard not a product; relevant background but not a primary research target

---

## Notable Findings

### DHT-based systems are underrepresented in production

Only GNUnet GNS and IPNS are genuine DHT-based naming systems with production deployments. All other major systems with significant adoption (ENS, Handshake, Namecoin, Unstoppable, SPACE ID, SNS) use dedicated blockchains or smart contracts. The DHT architecture is proven (RFC 9498, libp2p's global deployment) but has not achieved consumer adoption in naming.

### Chain-agnostic naming is an active design space (2025)

Three distinct approaches to chain-agnostic naming have emerged:
- **ENSv2 CCIP-Read**: off-chain resolution with on-chain verification; resolves names across any chain via a resolver gateway
- **SPACE ID / ZNS Connect**: deploy registrar contracts on each chain separately; resolution queries the target chain directly
- **D3 / Doma Protocol**: a purpose-built chain acts as the canonical record store; records reference multi-chain addresses

### Hybrid ICANN+blockchain is gaining momentum

3DNS and D3 both demonstrate the market appetite for ICANN-compatible blockchain names (i.e. domains that work in ordinary browsers). Freename and Unstoppable Domains both hold ICANN accreditation. This is the fastest-growing segment as of 2025.

### Privacy is almost completely absent from naming systems

With the exception of GNUnet GNS (query privacy by design) and the experimental Waku Naming Service, no production naming system addresses resolver-level privacy. DNS-over-HTTPS resolves the transport but not the resolver-trust problem. This is a significant gap.

### Logos/Waku relevance

The Waku Naming Service is the only project directly on the Logos stack. It is a prototype only; there is no production system. The RFP should consider both the GNS DHT architecture and IPNS as the closest prior art for a chain-agnostic, DHT-based naming layer.

---

## Sources

- [ENS Gate Learn 2026](https://www.gate.com/en-us/learn/articles/ens-domains-explained-how-ethereum-name-service-is-powering-web3-identity-in-2026/12406)
- [ENS Namechain The Block](https://www.theblock.co/post/325381/ens-labs-introduces-own-l2-agnostic-rollup-namechain-aiming-for-launch-by-end-of-2025)
- [Unstoppable Domains Wikipedia](https://en.wikipedia.org/wiki/Unstoppable_Domains)
- [Handshake GeeksforGeeks](https://www.geeksforgeeks.org/blogs/handshake-an-peer-to-peer-naming-system/)
- [Namebase Stats](https://www.namebase.io/stats/)
- [GNUnet GNS docs](https://docs.gnunet.org/latest/users/gns.html)
- [IPFS IPNS docs](https://docs.ipfs.tech/concepts/ipns/)
- [D3 Series A PR Newswire](https://www.prnewswire.com/news-releases/d3-raises-25m-series-a-led-by-paradigm-announces-the-first-blockchain-for-internets-362m-domain-names-302362780.html)
- [3DNS comparison post](https://3dns.box/blog/posts/comparing-decentralized-dns-providers/)
- [SPACE ID CoinMarketCap](https://coinmarketcap.com/cmc-ai/space-id/what-is/)
- [ZNS Connect website](https://znsconnect.io/)
- [Bonfida SNS naming](https://naming.bonfida.org/)
- [LFG Labs Starknet ID stats](https://lfglabs.dev/posts/starknetid)
- [Waku hackathon blog](https://blog.waku.org/what-we-built-at-the-first-waku-internal-hackathon/)
- [W3C DID 1.0](https://www.w3.org/TR/did-1.0/)
- [Namecoin.org](https://www.namecoin.org/)
- [EmerDNS](https://emercoin.com/en/emerdns/)
- [Freename Wikipedia](https://en.wikipedia.org/wiki/Freename)
- [Freename Series A](https://www.thedomains.com/2025/07/23/freename-raises-6-5m-series-a-to-power-the-future-of-web3-domains-digital-identity/)
- [Oxford/AFNIC blockchain domain mapping](https://www.oxil.uk/blog/mapping-blockchain-domain-providers-10-key-findings-from-oxford-information-labs-and-afnic)
- [MDPI survey on DNS and blockchain-based DNS](https://www.mdpi.com/2076-3417/16/2/598)
