# SPACE ID

## Summary

SPACE ID is a multi-chain Web3 name service protocol and identity platform. It enables users to register human-readable domain names (e.g. `alice.bnb`, `alice.arb`) on 24+ blockchains, resolving them to wallet addresses and other records. As of H1 2025, it is the largest multi-chain naming system by registered domain count.

---

## Key Metrics

| Metric | Value | Source | Date |
|--------|-------|--------|------|
| Registered domains | 6.7M+ | [CoinMarketCap CMC AI](https://coinmarketcap.com/cmc-ai/space-id/what-is/) | H1 2025 |
| Unique domain owners | 1.3M+ | [GlobeNewsWire](https://www.globenewswire.com/news-release/2025/01/08/3006371/0/en/Coinstats-Partners-With-SPACE-ID-as-Web3-Domain-Name-Usage-Rises.html) | Jan 2025 |
| dApp integrations | 330+ | [CoinMarketCap CMC AI](https://coinmarketcap.com/cmc-ai/space-id/what-is/) | 2025 |
| Supported chains | 24+ | [CoinMarketCap CMC AI](https://coinmarketcap.com/cmc-ai/space-id/what-is/) | 2025 |
| Blockchain explorer integrations | 8 | [CoinMarketCap latest updates](https://coinmarketcap.com/cmc-ai/space-id/latest-updates/) | July 2025 |
| Funding raised | $10M (3 rounds) | [CoinDesk](https://www.coindesk.com/business/2023/02/07/decentralized-identity-network-space-id-raises-10m) | Feb 2023 |
| Lead investors | Polychain Capital, dao5 (Binance Labs led Sep 2022 seed; participated in this round) | [CoinDesk](https://www.coindesk.com/business/2023/02/07/decentralized-identity-network-space-id-raises-10m) | Feb 2023 |
| ID token max supply | 2,000,000,000 | [SPACE ID docs](https://docs.space.id/overview/id-token) | — |
| ID token unlocked (as of 2025) | ~430.5M (21.5%) | [TokenInsight](https://tokeninsight.com/en/coins/space-id/tokenomics) | 2025 |
| ID token FDV | ~$147.9M USD | [CoinMarketCap](https://coinmarketcap.com/currencies/space-id/) | 2025 |
| TLD launch stake requirement | 10,000 $ID | [SPACE ID 3.0 Decrypt](https://decrypt.co/204281/space-id-3-0-unveils-id-token-staking-and-game-changing-upgrades-for-its-permissionless-name-service-protocol) | Nov 2023 |
| Protocol fee burn | 2% of $ID spent on each purchase | [SOURCE NEEDED] | — |
| Protocol fee to DAO | 3% of $ID spent on each purchase | [SOURCE NEEDED] | — |

---

## Background

| Field | Value |
|-------|-------|
| Founded | 2022 |
| Headquarters | Singapore |
| Ecosystem | BNB Chain (primary), Arbitrum, Ethereum + 21 more chains |
| Token | $ID (ERC-20 / BEP-20) |
| Token launch | March 2023, Binance Launchpad |
| Governance | SPACE ID DAO (ID-token-weighted voting) |
| Primary TLDs | `.bnb`, `.arb`, `.eth` (via ENS delegation), plus community-created TLDs |
| GitHub | [Space-ID](https://github.com/Space-ID) |
| Docs | [docs.space.id](https://docs.space.id) |

---

## Architecture

SPACE ID's architecture has three layers: the Protocol (smart contracts), the Platform (web app and SDK), and the DAO (governance).

### Protocol Layer: Three Core Components

The cross-chain uniqueness guarantee is the central technical challenge. SPACE ID solves it with three named components:

#### Jedi (chain-specific registrar contracts)

Jedi is a smart contract deployed on each supported blockchain. It handles domain registration and resolution for that chain. Before finalising a registration, Jedi requires a signature from Yoda confirming the name is not already taken on any other chain.

- One Jedi contract per supported chain
- Stores name-to-address records on its native chain
- Checks with Yoda before accepting a new name

#### Yoda (cross-chain uniqueness oracle)

Yoda is a network oracle that aggregates registration events from all Jedi contracts across all supported chains. It enforces global uniqueness: a name registered on BNB Chain cannot be re-registered on Arbitrum. Yoda issues cryptographic authentication signatures to users who want to register or bridge a name on any Jedi.

- Collects registration events from all chains
- Maintains a global name registry
- Issues signatures that Jedi contracts verify before finalising registration
- **Centralisation risk:** Yoda is an oracle layer whose decentralisation is not publicly documented; it is a potential single point of failure for the cross-chain uniqueness guarantee

#### Lucas (cross-chain data store / coordination chain)

Lucas is described as an "ad hoc blockchain" that connects and stores user data across multiple chains. It maintains registration metadata in a transcript managed by Yoda. Lucas acts as the director of the overall system.

- Functions as a shared state layer
- Managed in conjunction with Yoda
- Not the same as any of the 24+ supported end-user chains

### Platform Layer

- **SPACE ID App**: web interface for domain registration, management, and marketplace
- **Domain NFT Marketplace**: consolidated marketplace integrating listings from OpenSea, Element, ENSVision, and others
- **Web3 Name SDK**: JavaScript/TypeScript library for developers (see [[multi-chain-name-resolution]])
- **Web3 Name API**: free HTTP API for on-chain resolution, real-time
- **Payment ID**: `@tld` format identifier for CEX deposit addresses (see below)

---

## Resolution Model

SPACE ID uses a **per-chain deployment** model. Each supported chain hosts its own Jedi registrar contract. Resolution is performed by querying the Jedi contract on the relevant chain directly; there is no single canonical chain that all resolution must go through.

This contrasts with ENS, where resolution always starts on Ethereum mainnet (the single source of truth). SPACE ID's approach means:

- Lower latency: resolving a `.bnb` name queries BNB Chain directly, not Ethereum
- No Ethereum gas costs for non-Ethereum names
- Each chain's names are independent (`.bnb` and `.arb` are separate namespaces)
- Global uniqueness is enforced off-chain by Yoda, not on-chain by a shared contract

### Reverse Resolution

The Web3 Name SDK supports reverse resolution (address to domain name) per chain. Each EVM chain returns a "Chain Primary Name" for that chain.

### Non-EVM Chain Support

The SDK extends beyond EVM chains to support `.sei`, `.inj`, `.sol` and other non-EVM names.

---

## Web3 Name SDK

The [Web3 Name SDK](https://github.com/Space-ID/web3-name-sdk) is an open-source JavaScript/TypeScript library providing:

- **Domain to address resolution**: single call, zero configuration
- **Reverse resolution**: address to domain name, per-chain
- **Batch operations**: retrieve multiple domain names by TLD or by chain ID
- **Record fetch**: retrieve text records by domain name and key
- **Custom RPC support**: optional `rpcUrl` parameter for developer-controlled RPC
- **Auto-discovery**: when a new TLD is launched and verified on SPACE ID, the SDK automatically extends to support it
- **Payment ID support**: resolves `@tld` format identifiers

Supported TLDs include `.eth`, `.bnb`, `.arb`, `.lens`, `.crypto`, `.sol`, `.sei`, `.inj`, and all verified TLDs launched through the SPACE ID Toolkit.

Source: [GitHub Space-ID/web3-name-sdk](https://github.com/Space-ID/web3-name-sdk), [SPACE ID SDK docs](https://docs.space.id/developer-guide/web3-name-api-and-sdk/web3-name-sdk)

---

## Payment ID

Payment ID (`@tld` format) is a SPACE ID product that maps a single human-readable identifier to CEX deposit addresses across all EVM chains, Bitcoin, and Solana. Example: `jane@binance` resolves to the correct deposit address for the user's Binance account on each supported chain.

Authentication uses zkEmail: a zero-knowledge proof generated from the user's Gmail or Yahoo Mail account, enabling email-based on-chain login without exposing the email.

Integrations as of 2025:
- MetaMask
- Binance
- Enkrypt Wallet (PR Newswire, 2025)

Source: [SPACE ID Payment ID docs](https://docs.space.id/payment-id), [Crowdfund Insider](https://www.crowdfundinsider.com/2025/04/238368-space-ids-payment-id-links-centralized-web3-apps/), [PR Newswire Enkrypt](https://www.prnewswire.com/news-releases/enkrypt-integrates-space-ids-payment-id-for-seamless-cex-transfers-302487006.html)

---

## Permissionless TLD Launch (SPACE ID 3.0)

SPACE ID 3.0 (announced November 2023) makes the protocol permissionless: any developer or community can launch a new TLD using the One-Stop Domain Issuance Toolkit.

Requirements:
- Stake 10,000 $ID tokens
- Provide domain parameters, branding, and configuration via a web form
- TLD is then accessible on the SPACE ID platform and via the Web3 Name SDK (auto-discovery)

Source: [Decrypt SPACE ID 3.0](https://decrypt.co/204281/space-id-3-0-unveils-id-token-staking-and-game-changing-upgrades-for-its-permissionless-name-service-protocol), [SPACE ID docs launch overview](https://docs.space.id/launch-your-tlds-with-space-id/overview)

---

## $ID Token

| Field | Value |
|-------|-------|
| Max supply | 2,000,000,000 |
| Launch | March 2023 on Binance Launchpad |
| Primary uses | Staking (marketplace fee discounts, domain registration discounts), payments, governance voting |
| Distribution | Seed Sale 20%, Team 15%, Marketing 13%, Foundation 12%, Strategic Sale 8%, Advisors 7%, Ecosystem Fund 10%, Community Airdrop 10%, Public Sale 5% |
| Vesting period | Extends to 2028 |
| Burn mechanism | 2% of $ID spent on each domain purchase is burned [SOURCE NEEDED] |
| DAO treasury | 3% of $ID spent on each domain purchase goes to DAO treasury [SOURCE NEEDED] |

Source: [SPACE ID token docs](https://docs.space.id/overview/id-token), [Pintu Academy tokenomics](https://pintu.co.id/en/academy/post/space-id-tokenomics), [CoinMarketCap Academy](https://coinmarketcap.com/academy/article/what-is-space-id-features-and-tokenomics)

---

## DAO Governance

The SPACE ID DAO governs protocol parameters, grant allocation, and ecosystem decisions via $ID-weighted voting. Key decisions require on-chain proposals and votes.

Source: [SPACE ID token docs](https://docs.space.id/overview/id-token)

---

## Grant Programme

SPACE ID runs recurring grant seasons to fund ecosystem integrations. The grant programme has run at least 9 seasons (Season 9: January to July 2026). Notable metrics:

- Season 3: $121,650 budget available; $16,450 actually distributed across 25 funded projects
- Season 2: funded projects across exchange, wallet, tooling, gaming, infrastructure, and dApps tracks
- SDK grant: up to $10,000 per project integrating the Web3 Name SDK
- Evaluation criteria: Daily Active Users, Integration Utility, Integration Strength (resolution, reverse resolution, embedded registration)

Source: [SPACE ID Grant Program docs](https://docs.space.id/getting-started/programs/space-id-grant-program), [Season 3 Medium](https://medium.com/@VAHIDNFC/space-id-grant-program-season-3-fueling-innovation-and-ecosystem-growth-472751807d9d), [Season 4 Mirror](https://mirror.xyz/0xc8ddad01455560b8288073fe7052153330cC5C86/z65PqEGNQNYj9rTQwl4LQMc2eyxumLDny9jwag0vdI8)

---

## Roadmap

| Stage | Focus | Status |
|-------|-------|--------|
| Stage 1 | BNB Chain domain service (`.bnb`) | Complete |
| Stage 2 | Multi-chain name service (SPACE ID 2.0, SPACE ID 3.0, permissionless TLDs) | Complete |
| Stage 3 | AI Agent Domains (human-readable names for AI agents, `agent.sid` format) | 2025, ongoing |

Source: [SPACE ID roadmap docs](https://docs.space.id/overview/roadmap), [CoinMarketCap latest updates](https://coinmarketcap.com/cmc-ai/space-id/latest-updates/)

---

## Comparison with ENS

| Dimension | SPACE ID | ENS |
|-----------|----------|-----|
| Multi-chain approach | Per-chain Jedi contracts; global uniqueness via Yoda oracle | Single Ethereum mainnet as source of truth; CCIP-Read gateways for off-chain resolution |
| Uniqueness guarantee | Off-chain oracle (Yoda) aggregates across chains | On-chain registry on Ethereum L1 |
| Gas costs | Native to each chain (e.g. BNB gas for `.bnb`) | Ethereum L1 gas (mainnet) or rollup gas (Namechain) |
| Censorship resistance | On-chain per chain; oracle layer is not fully documented as decentralised | On-chain; ENSv2 DAO-governed |
| Privacy | No built-in query privacy; resolution queries are public | No built-in query privacy |
| Open-source | Yes (GitHub: Space-ID) | Yes (GitHub: ensdomains) |
| Permissionless TLDs | Yes (SPACE ID 3.0, stake 10,000 $ID) | No (controlled by ENS Labs / DAO) |
| Token required | $ID for TLD launch and discounts; not required for basic registration | ETH for registration fees; no protocol token |
| Domain count (2025) | 6.7M+ | ~910K active (late 2025) |

---

## Privacy and Censorship Resistance

- Domain records are stored on-chain; domain ownership cannot be altered by any central authority on the relevant chain
- SPACE ID explicitly describes itself as "censorship-resistant" due to blockchain-based storage
- The Yoda oracle layer is the principal centralisation risk: if Yoda is operated by a single party or is offline, cross-chain name uniqueness guarantees may fail or new registrations may be blocked
- SPACE ID granted a censorship-resistant website builder (1W3) through its grant programme, suggesting privacy-preserving use cases are encouraged but are third-party add-ons rather than protocol-level features
- No built-in resolver-level query privacy (queries to the Web3 Name API expose resolution requests to the API operator)

Source: [Medium coha05_](https://medium.com/@coha05_/space-id-what-is-it-eeeb9848b202), [Medium 1W3 grant](https://medium.com/@coha05_/space-id-grants-1w3-to-develop-censorship-resistant-website-builder-deb285d1240e), [BNB Chain spotlight](https://www.bnbchain.org/en/blog/bnb-chain-spotlight-space-id)

---

## Limitations and Open Questions

1. **Yoda oracle decentralisation**: the degree of decentralisation of the Yoda oracle is not clearly documented. If operated by a single operator or a small team, it is a centralised dependency for global name uniqueness.
2. **Lucas chain documentation**: Lucas is described briefly in overview articles but has limited public technical documentation.
3. **No built-in query privacy**: all name resolution queries via the API are visible to the API provider.
4. **Smart contract audit status**: [NOT FOUND] public audit reports for the Jedi registrar contracts or the Yoda oracle were not located during this research pass.
5. **Per-chain namespace fragmentation**: `.bnb` and `.arb` are separate namespaces; a user wanting the same name on all chains must register it on each chain separately (though Yoda prevents conflicts).
6. **Token dependency for TLD launch**: launching a TLD requires staking $ID, which introduces a financial barrier and ties the permissionless layer to token price.

---

## Behaviours Relevant to a Decentralised Naming RFP

These are observable behaviours that could inform requirements:

- **B1**: A single human-readable name resolves to the correct wallet address on any of 24+ chains without the user specifying the chain.
- **B2**: A name registered on one chain cannot be registered by a different party on another chain (global uniqueness).
- **B3**: Any developer or community can launch a custom TLD without permission from the core team, by staking tokens.
- **B4**: Resolution is performable with zero configuration (SDK resolves names out of the box).
- **B5**: A single identifier (`@tld`) maps to the correct deposit address across CEX platforms, abstracting away chain selection.
- **B6**: New TLDs are automatically discovered by the SDK once verified; integrations do not require manual updates.
- **B7**: Reverse resolution (address to human-readable name) is supported per chain.
- **B8**: Non-EVM chains (Solana, Injective, Sei) are supported within the same SDK interface.

---

## Related Notes

- [[multi-chain-name-resolution]] — pattern note on multi-chain resolution approaches
- [[ens]] — primary comparison: single-chain source of truth vs per-chain deployment
- [[unstoppable-domains]] — similar smart-contract naming system with one-time purchase model
- [[zns]] — similar per-chain deployment approach, smaller scale

---

## Sources

- [SPACE ID docs overview](https://docs.space.id)
- [SPACE ID token docs](https://docs.space.id/overview/id-token)
- [SPACE ID roadmap docs](https://docs.space.id/overview/roadmap)
- [SPACE ID TLD launch overview](https://docs.space.id/launch-your-tlds-with-space-id/overview)
- [SPACE ID Web3 Name SDK docs](https://docs.space.id/developer-guide/web3-name-api-and-sdk/web3-name-sdk)
- [SPACE ID Payment ID docs](https://docs.space.id/payment-id)
- [SPACE ID Grant Program docs](https://docs.space.id/getting-started/programs/space-id-grant-program)
- [GitHub Space-ID/web3-name-sdk](https://github.com/Space-ID/web3-name-sdk)
- [CoinDesk $10M raise](https://www.coindesk.com/business/2023/02/07/decentralized-identity-network-space-id-raises-10m)
- [CoinMarketCap CMC AI — what is SPACE ID](https://coinmarketcap.com/cmc-ai/space-id/what-is/)
- [CoinMarketCap SPACE ID latest updates](https://coinmarketcap.com/cmc-ai/space-id/latest-updates/)
- [CoinMarketCap Academy — features and tokenomics](https://coinmarketcap.com/academy/article/what-is-space-id-features-and-tokenomics)
- [Decrypt SPACE ID 3.0](https://decrypt.co/204281/space-id-3-0-unveils-id-token-staking-and-game-changing-upgrades-for-its-permissionless-name-service-protocol)
- [Pintu Academy tokenomics](https://pintu.co.id/en/academy/post/space-id-tokenomics)
- [TokenInsight tokenomics](https://tokeninsight.com/en/coins/space-id/tokenomics)
- [Crowdfund Insider Payment ID](https://www.crowdfundinsider.com/2025/04/238368-space-ids-payment-id-links-centralized-web3-apps/)
- [PR Newswire Enkrypt Payment ID](https://www.prnewswire.com/news-releases/enkrypt-integrates-space-ids-payment-id-for-seamless-cex-transfers-302487006.html)
- [Yahoo Finance Payment ID MetaMask Binance](https://finance.yahoo.com/news/space-id-launches-payment-id-124300190.html)
- [BNB Chain spotlight SPACE ID](https://www.bnbchain.org/en/blog/bnb-chain-spotlight-space-id)
- [ChainTech Network — what is SPACE ID](https://www.chaintech.network/blog/what-is-space-id-a-multi-chain-name-service/)
- [Pintu Academy — what is SPACE ID](https://pintu.co.id/en/academy/post/what-is-space-id)
- [Gate.com — what is SPACE ID](https://www.gate.com/learn/articles/what-is-space-id/606)
- [Medium coha05_ — what is SPACE ID](https://medium.com/@coha05_/space-id-what-is-it-eeeb9848b202)
- [Medium 1W3 grant](https://medium.com/@coha05_/space-id-grants-1w3-to-develop-censorship-resistant-website-builder-deb285d1240e)
- [BSC News SPACE ID 3.0](https://bsc.news/post/space-id-unveils-space-id-3-0-to-empower-web3-communities)
- [Season 3 grant Medium](https://medium.com/@VAHIDNFC/space-id-grant-program-season-3-fueling-innovation-and-ecosystem-growth-472751807d9d)
- [Season 4 grant Mirror](https://mirror.xyz/0xc8ddad01455560b8288073fe7052153330cC5C86/z65PqEGNQNYj9rTQwl4LQMc2eyxumLDny9jwag0vdI8)
- [Gnosis Chain SPACE ID SDK docs](https://docs.gnosischain.com/tools/web3-name-sdk)
- [Manta Network SPACE ID SDK docs](https://docs.manta.network/docs/manta-pacific/Space%20ID)
