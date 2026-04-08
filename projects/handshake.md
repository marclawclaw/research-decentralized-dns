---
tags: [project, pow-blockchain, utxo, naming, dns, tld-auction, handshake]
project: Handshake
ticker: HNS
ecosystem: Dedicated PoW blockchain (Bitcoin-derived UTXO)
status: active
researched: 2026-04-08
---

# Handshake (HNS)

Handshake is a decentralised, permissionless naming protocol built on a dedicated proof-of-work blockchain. Every participant validates and manages the root DNS naming zone, replacing the ICANN-controlled root zone file and root servers with a blockchain-based system. The goal is to eliminate dependence on certificate authorities (CAs) and DNS gatekeepers.

- Website: https://handshake.org
- Developer docs: https://hsd-dev.org/
- Full node (hsd): https://github.com/handshake-org/hsd
- SPV resolver (hnsd): https://github.com/handshake-org/hnsd
- Primary marketplace (post-Namecheap): https://www.namebase.io

---

## Adoption Metrics

| Metric | Value | Date | Source |
|--------|-------|------|--------|
| Total domain registrations | 11.3 million | 31 Jul 2024 (data now 21 months old; no newer public figure found) | [Namebase stats](https://www.namebase.io/stats/) (caution: stats URL reliability uncertain after Namebase sold to undisclosed buyer Jan 2026) via [HIP-68 discussion](https://github.com/handshake-org/HIPs/discussions/68) |
| HNS burned in auctions | 62.5 million HNS | 31 Jul 2024 (data now 21 months old) | [HIP-68 discussion](https://github.com/handshake-org/HIPs/discussions/68) |
| Circulating supply | 677.7 million HNS | Apr 2026 | [CoinMarketCap](https://coinmarketcap.com/currencies/handshake/) |
| Market cap | ~$3.3 million USD | Apr 2026 | [CoinMarketCap](https://coinmarketcap.com/currencies/handshake/) |
| HNS price | ~$0.0049 USD | Apr 2026 | [CoinMarketCap](https://coinmarketcap.com/currencies/handshake/) |
| HNS all-time high (ATH) | ~$0.85 USD | 5 May 2021 | [ath.ooo/hns](https://ath.ooo/hns); [CoinGecko](https://www.coingecko.com/en/coins/handshake) |
| CMC rank | #1369 | Apr 2026 | [CoinMarketCap](https://coinmarketcap.com/currencies/handshake/) |
| Fastest growth period | [DATA NEEDED] | [DATA NEEDED] | Claim "Jul to Nov 2023" could not be verified in the cited BestDapps article; no alternative source found |
| FOSS developer airdrop recipients | ~250,000 GitHub users (15+ followers) + ~30,000 PGP WOT keys | 2020 | [hs-airdrop GitHub](https://github.com/handshake-org/hs-airdrop) |
| FOSS airdrop claimed | ~25 million HNS (1.83% of initial supply) | 2020 | [hs-airdrop GitHub](https://github.com/handshake-org/hs-airdrop) |
| HNS V2 airdrop to TLD owners (proposed) | 45 million HNS | cutoff Feb 2025 | [HIP-68 discussion](https://github.com/handshake-org/HIPs/discussions/68) |

---

## Token Economics

| Parameter | Value | Source |
|-----------|-------|--------|
| Initial genesis supply | 1.36 billion HNS | [Namebase HNS economics](https://learn.namebase.io/about-handshake/handshake-coin) |
| Total mineable (PoW) | 680 million HNS | [Namebase HNS economics](https://learn.namebase.io/about-handshake/handshake-coin) |
| Fully diluted max supply | 2.04 billion HNS | [CoinMarketCap](https://coinmarketcap.com/currencies/handshake/) |
| Initial block reward | 2,000 HNS per block | [hsd-dev protocol summary](https://hsd-dev.org/protocol/summary.html) |
| Block time | 10 minutes | [hsd-dev protocol summary](https://hsd-dev.org/protocol/summary.html) |
| Halving interval | Every 170,000 blocks (~3.25 years) | [ViaBTC HNS guide](https://www.viabtc.com/en/blog/Beginner-what-is-handshake-hns-how-to-mine-handshake-hns-458) |
| Mining ceases (projected) | Block 5,270,000, ~year 2119 | [ViaBTC HNS guide](https://www.viabtc.com/en/blog/Beginner-what-is-handshake-hns-how-to-mine-handshake-hns-458) |
| Investor allocation | 7.5% of initial 1.36B | [Namebase HNS economics](https://learn.namebase.io/about-handshake/handshake-coin) |
| FOSS developer allocation | 70% of initial 1.36B (952 million HNS) | [Namebase HNS economics](https://learn.namebase.io/about-handshake/handshake-coin) |
| No ICO | Yes (no token sale to public) | [Kraken HNS overview](https://www.kraken.com/learn/what-is-handshake-hns) |
| Seed funding raised | $10.2 million (2018) | [Messari](https://messari.io/project/handshake) |
| Seed investors | a16z, Founders Fund, Polychain Capital, Draper Associates (67 total) | [Messari](https://messari.io/project/handshake) |

---

## How It Works

Handshake replaces the DNS root zone (the file that maps TLDs to their authoritative name servers) with a PoW blockchain. The key departure from traditional DNS: instead of ICANN owning the root, every full node in the Handshake network independently validates and replicates the complete root zone state.

### Layer 1: The blockchain (TLD ownership)

- Bitcoin-derived UTXO model; 10-minute blocks; PoW using BLAKE2b + SHA3 (see [[pow-naming]])
- Every full node validates the root zone
- Name state is encoded as **covenants** attached to UTXO outputs
- The 12 covenant types are: NONE, CLAIM, OPEN, BID, REVEAL, REDEEM, REGISTER, UPDATE, RENEW, TRANSFER, FINALIZE, REVOKE
- New covenant types can be added via soft fork
- Max block size: 1,000,000 bytes base / 4,000,000 weight units

### Layer 2: TLD auction (name acquisition, see [[tld-auction-mechanism]])

1. **OPEN**: any participant broadcasts an OPEN covenant to start a 5-day bidding window for an unclaimed TLD
2. **BID**: participants submit blinded bids (bid amount + blind amount, only the combined lockup is visible on-chain)
3. **REVEAL**: 10-day reveal period; bidders publish their actual bid amounts; unrevealed bids are forfeited
4. **REGISTER**: winner pays second-highest bid (Vickrey mechanism); name is registered and locked to the winner's address
5. Renewal required every 2 years via RENEW covenant (small miner fee; no auction fee on renewal)

### Layer 3: DNS record storage (UPDATE covenant)

- The TLD owner publishes DNS records directly on-chain via the UPDATE covenant
- Up to 512 bytes of arbitrary DNS data per UPDATE, at most once per block
- Typically stores an NS record pointing to an authoritative name server, or a DS record for DNSSEC delegation
- SLDs (second-level domains) are managed off-chain by the TLD owner's name server; the blockchain only tracks TLDs

### Layer 4: Resolution (see [[handshake-resolution-stack]])

- **hsd full node**: runs the complete chain; acts as a recursive resolver and authoritative root server; most private option
- **hnsd SPV light client**: downloads only block headers + Merkle proofs; translates HNS name data into DNS responses; connects to full nodes for proofs
- **hnsquery**: cross-platform SPV library for embedding HNS resolution in apps; routes DNS lookups over DoH (DNS-over-HTTPS), concealing query content within HTTPS traffic
- **Third-party gateways**: hns.to, hns.is, rsvr.xyz act as DNS proxies; convenient but trust-dependent
- Standard stub resolvers (e.g., browsers) are pointed at a local hsd/hnsd instance or a gateway

### Layer 5: DANE/TLSA (CA replacement, see [[dnssec-dane-integration]])

- Handshake replaces the CA root of trust by combining DNS security with PoW
- TLD owners publish TLSA records (RFC 6698) in their DNS zone
- Because the TLSA record is committed to by the blockchain's PoW, TLS certificates can be self-signed; no CA needed
- Currently limited by browser support: Chrome does not implement DANE; Firefox support is partial

---

## Key Behaviours (RFP relevance)

| Behaviour | Detail |
|-----------|--------|
| Root zone decentralisation | Every full node independently validates the entire root zone; no single party can unilaterally censor or modify TLD records |
| Vickrey auction pricing | Second-price sealed-bid auction; winner pays second-highest bid, not their own bid; discourages strategic overbidding |
| Bid privacy (partial) | Bid amounts are blinded on-chain; only the lockup (bid + blind) is visible to competitors; true bid is revealed after close |
| UTXO-native name state | Name ownership is a UTXO; transfer is a coin spend; no separate smart contract runtime needed |
| TLD-only scope | The blockchain governs only TLDs; SLD management is fully delegated to off-chain DNS infrastructure chosen by the TLD owner |
| Renewal required | Names expire if not renewed every 2 years; prevents permanent squatting; costs only a miner fee (no auction burn on renewal) |
| REVOKE covenant | A name can be permanently revoked, preventing transfer during dispute; irreversible |
| TRANSFER + FINALIZE | Two-step transfer with a ~2-day lock; original owner can REVOKE during the lock window |
| Chain-agnostic record data | UPDATE covenant stores up to 512 bytes of raw DNS data; any DNS record type can be stored |
| No DHT layer | All name data is on-chain; no distributed hash table; resolution always queries the blockchain |
| Censorship resistance | Censoring a name requires sustained majority hash-rate attack; no administrative revocation path |
| Privacy limitation | SPV light clients leak TLD queries to the full nodes they connect to; hnsquery partially mitigates this by routing DNS lookups over DoH (DNS-over-HTTPS), which hides query content in HTTPS traffic rather than exposing it on port 53 ([hnsquery README](https://github.com/imperviousinc/hnsquery/blob/main/README.md)) |

---

## Architecture Decisions

| Decision | Rationale | Trade-off |
|----------|-----------|-----------|
| PoW consensus (not PoS) | Known security parameters; enables compact SPV light-client proofs | High energy consumption; ASIC centralisation risk |
| UTXO model (not account model) | Stateless verification; parallelism; aligns with Bitcoin security assumptions | More complex covenant logic than EVM smart contracts |
| TLD-only scope | Keeps blockchain state minimal; scales to millions of SLDs without chain bloat | TLD owners must run off-chain DNS infrastructure; adds complexity for end users |
| Vickrey auction (not first-price) | Encourages honest bidding; the winner does not overpay relative to competition | Requires two-step commit/reveal, adding ~15 days of latency to name acquisition |
| BLAKE2b + SHA3 PoW | Designed for ASIC resistance (partially achieved); SHA3 adds collision resistance | Non-standard; smaller mining ecosystem than Bitcoin SHA256 |
| No ICO; FOSS airdrop | Bootstraps with technically engaged users; avoids securities risk; broad initial distribution | Low token price if recipients sell; weak financial incentive for speculative buyers |
| DANE for CA replacement | Eliminates trusted third-party CAs entirely | Browser adoption is the bottleneck; Chrome DANE non-support limits practical use |
| Root zone reserved TLDs | Existing ICANN TLDs are blacklisted from auction to avoid conflicts | Incomplete: some pending ICANN TLDs (e.g., .MUSIC, .WEB) were missed at launch |

---

## Differentiators

1. **Only production system that replaces the DNS root zone** (not just a namespace overlay): Handshake is the only deployed system where the blockchain IS the root zone, not a separate namespace requiring a custom resolver to bridge to.
2. **TLD ownership, not SLD ownership**: all other major systems (ENS, Unstoppable, SPACE ID) register second-level or lower domains; Handshake owners control an entire TLD.
3. **CA elimination via DANE**: the combination of PoW root trust + DANE means a correctly set up Handshake site does not require any trusted third party for TLS, not even a CA.
4. **FOSS-first distribution**: no ICO; majority of genesis supply airdropped to open source contributors; investor allocation capped at 7.5%.
5. **Extensible covenant system**: new covenant types can be added via soft fork without changing the core UTXO model.

---

## Limitations and Criticisms

| Limitation | Detail |
|-----------|--------|
| Browser support absent | No major browser supports HNS natively; users need a plugin, gateway, or local hsd/hnsd instance |
| DANE not supported by Chrome | Chrome deliberately does not implement DANE; CA replacement benefit is inaccessible to most users |
| HNS required to participate | Must hold HNS coins to bid on names; barrier to non-crypto users |
| ICANN conflict risk | Future ICANN TLDs may collide with existing HNS TLDs; resolvers must choose one; some upcoming ICANN TLDs missed in the initial reservation list |
| Declining ecosystem interest | Namebase acquired by Namecheap (2022), then sold by Namecheap (Jan 2026) to undisclosed buyer; token price near historic lows (~$0.005 Apr 2026) |
| TLD owner DNS infrastructure burden | TLD owners must operate their own authoritative name servers for SLD management; not a simple web3 product |
| Key loss is permanent | Unlike traditional DNS, a lost private key means permanent loss of the TLD name; no recovery mechanism |
| Airdrop audit concern | The airdrop merkle tree uses a novel cryptographic blinding factor that makes the tree unverifiable by external auditors; the root is hard-coded into consensus rules |
| Privacy leakage at resolution | SPV clients leak queried TLD names to the full nodes they connect to |
| Low adoption vs. registrations | 11.3M registrations include speculative/bulk registrations; actual usage (resolving HNS sites) is a fraction of that |

---

## Ecosystem and Tooling

| Component | Description | Status |
|-----------|-------------|--------|
| hsd | Full node daemon (Node.js); includes wallet, auction management, recursive resolver | Active |
| hnsd | SPV light client in C; acts as authoritative root server for DNS; minimal footprint | Active |
| hnsquery | Cross-platform SPV resolution library; uses DoH for DNS query privacy | Active |
| hdns | Handshake-capable DNS module for Node.js | Active |
| Bob Wallet | Desktop GUI wallet + hsd full node; DNS record management and auctions | Active |
| Namebase | Primary HNS marketplace and registrar (acquired by Namecheap 2022; sold Jan 2026) | Ownership change |
| Namecheap | Registered HNS SLDs under its own TLDs; sold Namebase Jan 2026 | Active but reduced HNS focus |
| hns.to / hns.is / rsvr.xyz | Third-party DNS gateways for HNS resolution without local setup | Active (trust-dependent) |
| DNSimple | Added HNS domain hosting support (Apr 2022) | Active |
| Porkbun | Provides information/support for HNS TLD resolution | Active |

---

## Related Patterns

- [[tld-auction-mechanism]] — Vickrey sealed-bid auction for TLD allocation
- [[dnssec-dane-integration]] — DANE/TLSA as CA replacement secured by PoW
- [[pow-naming]] — proof-of-work blockchain as root of trust for naming
- [[root-zone-replacement]] — replacing ICANN root zone with a blockchain

---

## Related Projects

- [[namecoin]] — original blockchain naming (2011); .bit TLD; merged mining with Bitcoin
- [[ens]] — smart-contract naming on Ethereum; SLD model; ENSv2 cross-chain
- [[gns]] — DHT-based naming; no blockchain; RFC 9498

---

## Sources

- [Handshake official site](https://handshake.org)
- [Handshake developer docs (hsd-dev.org)](https://hsd-dev.org/protocol/summary.html)
- [hsd GitHub](https://github.com/handshake-org/hsd)
- [hnsd GitHub](https://github.com/handshake-org/hnsd)
- [Namebase stats](https://www.namebase.io/stats/) (caution: reliability uncertain post-Jan 2026 ownership change)
- [HIP-68: Handshake V2 proposal](https://github.com/handshake-org/HIPs/discussions/68)
- [Namebase HNS coin economics](https://learn.namebase.io/about-handshake/handshake-coin)
- [Namebase auction tutorial](https://www.namebase.io/blog/tutorial-3-basics-of-handshake-auction-and-bidding/)
- [CoinTelegraph: What are HNS domains](https://cointelegraph.com/news/what-are-handshake-hns-domains-and-how-do-they-work)
- [Kraken: What is Handshake HNS](https://www.kraken.com/learn/what-is-handshake-hns)
- [CoinMarketCap HNS](https://coinmarketcap.com/currencies/handshake/)
- [CoinGecko HNS](https://www.coingecko.com/en/coins/handshake)
- [ath.ooo HNS all-time high](https://ath.ooo/hns)
- [BestDapps deepdive 2025](https://bestdapps.com/blogs/news/a-deepdive-into-hns-2025)
- [hs-airdrop GitHub (FOSS airdrop)](https://github.com/handshake-org/hs-airdrop)
- [Matthew Zipkin: HNS TLSA security](https://matthewzipkin.medium.com/using-hns-websites-securely-69959ae02052)
- [Matthew Zipkin: Building secure HNS site](https://matthewzipkin.medium.com/building-a-secure-website-on-your-handshake-tld-a8922a950a4f)
- [Bob Wallet](https://bobwallet.io/)
- [hnsquery library](https://github.com/imperviousinc/hnsquery)
- [Domain Name Wire: Namecheap sells Namebase](https://domainnamewire.com/2026/01/28/namecheap-sells-handshake-marketplace-namebase/)
- [GeeksforGeeks HNS overview](https://www.geeksforgeeks.org/blogs/handshake-an-peer-to-peer-naming-system/)
- [ViaBTC: How to mine HNS](https://www.viabtc.com/en/blog/Beginner-what-is-handshake-hns-how-to-mine-handshake-hns-458)
- [IQ.wiki Handshake](https://iq.wiki/wiki/handshake)
- [hsd-dev: Auction guide](https://hsd-dev.org/guides/auctions.html)
