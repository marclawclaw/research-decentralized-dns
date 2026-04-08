---
tags: [metrics, comparison, handshake, naming-systems, adoption]
metric: Handshake vs. Smart-Contract Naming Systems
last-updated: 2026-04-08
---

# Metrics: Handshake vs. Smart-Contract Naming Systems

Cross-project comparison of key adoption and technical metrics for decentralised naming systems, with Handshake (HNS) as the primary subject.

---

## Registered Domain / Name Count

| Project | Count | Date | Source |
|---------|-------|------|--------|
| Handshake (TLD registrations) | 11.3 million | 31 Jul 2024 | [Namebase stats](https://www.namebase.io/stats/) via [HIP-68](https://github.com/handshake-org/HIPs/discussions/68) |
| Unstoppable Domains (Web3) | >4.2 million | 2025 | [Wikipedia](https://en.wikipedia.org/wiki/Unstoppable_Domains) |
| SPACE ID | 6.7 million+ | 2025 | [CoinMarketCap](https://coinmarketcap.com/cmc-ai/space-id/what-is/) |
| ENS | ~2.5 million total; ~910K active | late 2025 | [Gate Learn ENS 2026](https://www.gate.com/en-us/learn/articles/ens-domains-explained-how-ethereum-name-service-is-powering-web3-identity-in-2026/12406) |
| Solana Name Service (SNS) | 283,000+ .sol domains | 2025 | [Bonfida SNS](https://naming.bonfida.org/) |
| Namecoin (.bit) | [NOT FOUND: current count] | | |

Note: Handshake counts are TLD registrations (not SLD registrations). A single HNS TLD owner may or may not be operating any SLDs beneath their TLD. The 11.3M figure likely includes speculative and bulk registrations with minimal active usage.

---

## Token Economics Comparison

| Project | Token | Max Supply | Market Cap (Apr 2026) | Source |
|---------|-------|-----------|----------------------|--------|
| Handshake | HNS | 2.04 billion | ~$3.3 million | [CoinMarketCap](https://coinmarketcap.com/currencies/handshake/) |
| ENS | ENS | 100 million | [NOT FOUND: Apr 2026 figure] | |
| SPACE ID | $ID | [NOT FOUND] | [NOT FOUND: Apr 2026 figure] | |
| Namecoin | NMC | 21 million | ~$13.2 million | [Wikipedia Namecoin](https://en.wikipedia.org/wiki/Namecoin) |

---

## Name Acquisition Model

| Project | Model | Cost structure | Renewal required? |
|---------|-------|---------------|-------------------|
| Handshake | TLD auction (Vickrey, 15-day cycle) | Bid burn (second-highest bid burned); miner fee only for renewal | Yes, every 2 years |
| ENS | Instant registration at fixed price | Annual fee (price-based on length/demand) | Yes, annual |
| Unstoppable Domains | Instant registration at fixed price | One-time fee; no renewal | No |
| Namecoin | Fixed fee per registration | 0.01 NMC per record | Yes, ~200 days |
| SNS | Instant registration | One-time fee (varies by length) | No |

---

## Resolution Stack Complexity

| Project | Requires custom resolver? | Light client available? | Browser native? |
|---------|--------------------------|------------------------|----------------|
| Handshake | Yes (hnsd, hsd, or gateway) | Yes (hnsd SPV) | No |
| ENS | No (for .eth via MetaMask/ENS plugin); yes for on-chain resolution | Yes (CCIP-Read via any RPC) | No (plugin required) |
| Unstoppable Domains | No (gateway available) | [NOT FOUND] | No |
| Namecoin | Yes (ncdns or local node) | No native SPV for DNS | No |
| GNS (GNUnet) | Yes (GNUnet daemon) | No | No |

---

## Privacy Properties

| Project | Query privacy | Registration privacy |
|---------|--------------|---------------------|
| Handshake | Weak: SPV clients leak TLD queries to full nodes; hnsquery hashes queries but full node still observes patterns | Moderate: bid amounts blinded on-chain; ultimate owner of private key is pseudonymous |
| ENS | Weak: on-chain lookups are public; ENS subgraph is queryable | Weak: wallet address to name mapping is public |
| GNS (GNUnet) | Strong: DHT query routing designed for privacy; zone records are private by default | Strong: no token; no on-chain trace |
| IPNS | Weak: DHT queries are observable by routing peers | Moderate: IPNS name is hash of public key |

---

## Censorship Resistance Assessment

| Project | Censorship mechanism | Attack cost |
|---------|---------------------|-------------|
| Handshake | Requires majority hash-rate attack (51%) | Economic; scales with network hash rate (~low, given $3.3M market cap) |
| ENS | Requires majority Ethereum stake or smart contract governance attack | High (Ethereum is large); governance veto via ENS DAO is a soft attack vector |
| Namecoin | Merged-mined with Bitcoin; 51% attack is Bitcoin 51% attack | Very high |
| Unstoppable Domains | Company can stop issuing; but existing NFTs are on-chain and cannot be revoked | Social/legal: company is US-based |
| GNS (GNUnet) | Requires attacking DHT routing; no single point | Low (small network); but records are self-certifying; routing attack only delays, not forges |

---

## Ecosystem Health Indicators (Apr 2026)

| Indicator | Handshake | Notes |
|-----------|-----------|-------|
| Primary marketplace | Namebase.io (ownership unknown post Jan 2026 sale) | Sold by Namecheap to undisclosed buyer Jan 2026 |
| Active development | hsd GitHub (last commit [NOT FOUND: exact date]) | |
| HIP proposals activity | HIP-68 (V2 proposal) active as of Feb 2025 | [GitHub HIPs discussion](https://github.com/handshake-org/HIPs/discussions/68) |
| Bob Wallet | Active | Desktop GUI with integrated full node |
| Token price trend | Near historic lows (~$0.005 Apr 2026 vs. ~$0.50+ peak) | [CoinMarketCap](https://coinmarketcap.com/currencies/handshake/) |

---

## Sources

- [Namebase stats](https://www.namebase.io/stats/)
- [HIP-68: Handshake V2](https://github.com/handshake-org/HIPs/discussions/68)
- [CoinMarketCap HNS](https://coinmarketcap.com/currencies/handshake/)
- [Wikipedia Namecoin](https://en.wikipedia.org/wiki/Namecoin)
- [Gate Learn ENS 2026](https://www.gate.com/en-us/learn/articles/ens-domains-explained-how-ethereum-name-service-is-powering-web3-identity-in-2026/12406)
- [Unstoppable Domains Wikipedia](https://en.wikipedia.org/wiki/Unstoppable_Domains)
- [SPACE ID CoinMarketCap](https://coinmarketcap.com/cmc-ai/space-id/what-is/)
- [Bonfida SNS](https://naming.bonfida.org/)
- [Domain Name Wire: Namecheap sells Namebase](https://domainnamewire.com/2026/01/28/namecheap-sells-handshake-marketplace-namebase/)
