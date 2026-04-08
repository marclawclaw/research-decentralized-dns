# D3 / Doma Protocol

## Summary

D3 Global is the company building the Doma Protocol, described as the first purpose-built blockchain for internet domain tokenisation. Doma Chain is an EVM-compatible Layer 2 built on Optimism's OP Stack, deployed via Conduit, with Celestia for data availability and LayerZero for cross-chain messaging. The protocol bridges ICANN-governed DNS domains (.com, .net, .ai, .org) and Web3-native TLDs (.shib, .near, .ape) by minting on-chain representations while keeping the DNS record authoritative. Its stated differentiator is ICANN compliance by design rather than as an afterthought.

## Key Links

- Website: https://d3.com / https://doma.xyz
- Docs: https://docs.d3.app / https://docs.doma.xyz
- Blog: https://blog.doma.xyz
- Mainnet launch post: https://blog.doma.xyz/doma-mainnet-is-live/
- ENS integration: https://ens.domains/blog/post/d3-doma

## Classification

- **Category:** Domain name tokenisation / RWA / DomainFi
- **Ecosystem:** Purpose-built L2 (Doma Chain); cross-chain to Ethereum, Base, Solana, Avalanche
- **Stage:** Mainnet live (launched 25 November 2025)
- **Chain type:** EVM-compatible optimistic rollup (OP Stack via Conduit)

## Team

- **CEO / Co-founder:** Fred Hsu — previously co-founder and CTO of Oversee.net (sold to Oak Hill Capital after $150M Series A); co-founder and CEO of Manage.com (acquired by Criteo). [Source: https://techstartups.com/2024/02/06/founder-story-interview-with-fred-hsu-co-founder-and-ceo-of-d3-on-web3-domains/, February 2024]
- **Co-founder:** Paul Stahura — previously founder, CEO, and chairman of eNom (major domain registrar). [Source: same]

## Funding

| Round | Date | Amount | Lead Investor | Source |
|-------|------|--------|---------------|--------|
| Seed | September 2023 | $5M | Shima Capital | https://domainnamewire.com/2025/01/29/d3-raises-25-million-series-a/ |
| Series A | 29 January 2025 | $25M | Paradigm | https://www.prnewswire.com/news-releases/d3-raises-25m-series-a-led-by-paradigm-announces-the-first-blockchain-for-internets-362m-domain-names-302362780.html |

**Total raised:** $30M (as of January 2025)

**Series A co-investors:** Coinbase Ventures, Sandeep Nailwal (Polygon Labs co-founder), Dharmesh Shah (HubSpot founder), Richard Kirkendall (Namecheap CEO). [Source: https://www.prnewswire.com/news-releases/d3-raises-25m-series-a-led-by-paradigm-announces-the-first-blockchain-for-internets-362m-domain-names-302362780.html, 29 January 2025]

## Architecture

### Doma Chain

Doma Chain is an EVM-compatible Layer 2:

- **Execution layer:** OP Stack (Optimism), deployed via Conduit. Conduit's G2 sequencer claims 100 Mgas/s throughput (10x standard OP Stack sequencers). [Source: https://www.conduit.xyz, 2025]
- **Data availability:** Celestia (modular DA layer, minimises cost for high-volume minting). [Source: https://messari.io/report/doma-domains-galore]
- **Cross-chain messaging:** LayerZero v2, enabling Omnichain Fungible Tokens (OFTs) on Ethereum, Base, Solana. [Source: https://messari.io/report/doma-domains-galore]
- **Wallet infrastructure:** Privy
- **RPC:** dRPC (decentralised RPC)
- **Initial validator set (Genesis, 25 November 2025):** D3, Conduit, and early registrar partners. [Source: https://messari.io/report/doma-domains-galore]

### Token Model

Doma introduces two ERC-20/NFT token primitives per domain:

1. **Domain Ownership Token (DOT):** NFT representing full ownership and transfer rights. Tradeable, collaterable, fractionalisable. Minted on the user's chosen chain; the Doma Chain holds the authoritative record.
2. **Domain Service Token (DST):** Controls ongoing DNS operations (DNS record edits, email routing). Separating ownership from DNS control allows a domain to be traded while continuing to function normally.

[Source: https://docs.doma.xyz/readme/protocol-overview; https://messari.io/report/doma-domains-galore]

### Protocol Modules

| Module | Function |
|--------|----------|
| **Domain Tokenisation Module** | APIs and smart contracts for ICANN-accredited registrars to mint DOTs |
| **Domain Partitioning Module** | Fractionalises DOTs into fungible tokens for partial ownership and liquidity pools |
| **Compliance Module** | UDRP support: Transfer Lock (freeze during investigation), Forced Detokenisation (return to registrar after ruling) |
| **Custodian Module** | Solves ICANN registrant data requirement: when a DOT is transferred, the registrar places domain in a proxy account; new owner completes a KYC/contact claim before full control transfers |
| **Bridging Module** | Moves DOTs and synthetic tokens across L1/L2 networks via LayerZero |

[Source: https://messari.io/report/doma-domains-galore; https://docs.doma.xyz/readme/protocol-overview]

### ENS Integration

Announced October 2025. D3 updated the Doma registry smart contract to emit ENS-compatible records. ENS whitelisted the Doma registry, so tokenised DNS domains behave as first-class ENS names. When the DOT NFT is transferred, the ENS profile (linked address, avatar) updates automatically without a separate transaction. No DNSSEC setup required for domains tokenised through Doma. [Source: https://ens.domains/blog/post/d3-doma, October 2025]

## Supported Domains

- **Web2 TLDs available at mainnet:** .com, .net, .ai, .org, .xyz, and others managed by registrar partners
- **Web3-native TLDs (planned, pending ICANN approval):** .shib, .near, .core, .ape, .doge, .sol, .solana, .avax, .magic — expected Summer 2026
- **Registrar claim at mainnet:** 510+ communities and TLDs supported via D3.app
- **Total addressable market claimed:** 371M+ existing internet domains; $360B domain industry

[Source: https://d3.app; https://www.globenewswire.com/news-release/2025/12/11/3203728/0/en/D3-and-InterNetX-Partner-to-Bring-Over-46-Million-Domains-to-Solana-as-Tokenized-RWAs.html]

## Applications Built on Doma

### Interstellar (formerly D3.app)
Next-generation domain marketplace. Purchase domains with USDC or ETH; instant on-chain settlement; no escrow/waiting periods. Bridges traditional DNS domains with Web3 identity. [Source: https://docs.d3.app/d3-is-now-interstellar]

### Mizu Launchpad
DeFi platform for fractionalized domain trading. Users can: trade fractional ownership of domains, provide liquidity to earn trading fees, stake for utility features. Demonstrated with flagship domains software.ai and Brag.com at mainnet launch. [Source: https://www.theblock.co/press-releases/371586/d3-unveils-mizu-launchpad-on-doma-protocol-unlocking-defi-potential-for-internet-domains]

### Doma Names Marketplace on Base (December 2025)
Launched 18 December 2025. Brings 40M+ DNS domains to the Base app. Users browse, purchase (USDC or ETH), and manage domain portfolios without leaving the Base app. InterNetX (22M+ active domains, 24M+ premium listings) is anchor partner. [Source: https://www.globenewswire.com/news-release/2025/12/18/3207950/0/en/Doma-Protocol-Launches-Names-Marketplace-on-Base-to-Bring-Over-40-Million-Domains-Onchain.html, 18 December 2025]

## Adoption Metrics

| Metric | Value | Date | Source |
|--------|-------|------|--------|
| Testnet transactions | 35M+ | At mainnet launch (Nov 2025) | https://blog.doma.xyz/doma-mainnet-is-live/ |
| Testnet addresses | 1.45M | At mainnet launch (Nov 2025) | https://blog.doma.xyz/doma-mainnet-is-live/ |
| Testnet domains tokenised | 200,000+ | At mainnet launch (Nov 2025) | https://blog.doma.xyz/doma-mainnet-is-live/ |
| Testnet transactions (earlier count) | 2.9M | August 2025 | https://www.ainvest.com/news/d3-mizu-launchpad-future-defi-enabled-internet-domains-2509/ |
| Testnet active wallets | 115,000 | August 2025 | https://www.ainvest.com/news/d3-mizu-launchpad-future-defi-enabled-internet-domains-2509/ |
| Testnet domains tokenised (earlier) | 705,000 | August 2025 | https://www.ainvest.com/news/d3-mizu-launchpad-future-defi-enabled-internet-domains-2509/ |
| Registrar partner domains accessible | 46M+ | December 2025 | https://www.globenewswire.com/news-release/2025/12/11/3203728/0/en/D3-and-InterNetX-Partner-to-Bring-Over-46-Million-Domains-to-Solana-as-Tokenized-RWAs.html |
| Developer grant pool | $1M USDC | June 2025 | https://thedefiant.io/news/press-releases/d3-unveils-1m-usdc-developer-fund-to-fuel-doma-testnet-momentum-and-domainfi-innovation |

Note: testnet metrics (705k domains tokenised in August vs 200k+ at mainnet launch) appear contradictory; the August figure may include testnet recycling or a different counting method. [NOT CONFIRMED — data point needs reconciliation]

## Key Partnerships

| Partner | Role | Domains | Date |
|---------|------|---------|------|
| InterNetX (IONOS Group) | Registrar; anchor for Base marketplace | 22M+ active, 24M+ premium across 30,000 partners | Dec 2025 |
| NicNames | Registrar partner | Part of the 30M+ at mainnet | Nov 2025 |
| Rumahweb | Registrar partner | Part of the 30M+ at mainnet | Nov 2025 |
| EnCirca | Registrar partner | Part of the 30M+ at mainnet | Nov 2025 |
| Solana Foundation | Distribution channel (Breakpoint 2025) | 46M via InterNetX bridge to Solana | Dec 2025 |
| Base (Coinbase) | Consumer distribution channel | 40M+ domains in Base app marketplace | Dec 2025 |

[Source: https://www.globenewswire.com/news-release/2025/12/11/3203728/0/en/D3-and-InterNetX-Partner-to-Bring-Over-46-Million-Domains-to-Solana-as-Tokenized-RWAs.html; https://www.globenewswire.com/news-release/2025/12/18/3207950/0/en/Doma-Protocol-Launches-Names-Marketplace-on-Base-to-Bring-Over-40-Million-Domains-Onchain.html]

## Key Behaviours (Potential RFP Requirements)

1. **Dual-token ownership/control separation:** A name system can separate ownership rights (tradeable NFT) from operational control (DNS settings, routing) using two distinct token types. See [[patterns/dual-token-name-ownership]].

2. **ICANN-compliant tokenisation with custodian handshake:** Transferring a tokenised domain triggers a registrar notification and a proxy/custodian step requiring the new owner to submit contact information before gaining full control. This preserves ICANN's registrant data requirements. See [[patterns/icann-compliant-decentralised-naming]].

3. **Compliance enforcement on-chain:** UDRP disputes can trigger a Transfer Lock that freezes domain assets on-chain; a legal ruling can force on-chain detokenisation (burning the DOT). This means a "decentralised" name system can be subject to legal forced revocation.

4. **Cross-chain resolution via ENS whitelisting:** Tokenised traditional DNS domains can resolve as native ENS names without DNSSEC setup, by having the naming chain's registry whitelisted in ENS. See [[patterns/cross-chain-name-resolution]].

5. **Fractionalization / DomainFi:** A single domain can be split into fungible ERC-20 tokens for partial ownership, liquidity provision, and DeFi collateral, while the underlying DNS record remains intact and functional.

6. **Registrar as integration point:** Traditional ICANN-accredited registrars act as the bridge between off-chain DNS records and on-chain tokens, rather than the protocol disintermediating them.

## Limitations and Risks

### Centralisation

- DNS root zone is administered by ICANN; tokenising a domain does not remove upstream ICANN/registry/registrar power. If a government or court orders seizure, the Compliance Module is designed to execute it on-chain.
- Initial validator set (Genesis) is composed only of D3, Conduit, and early registrar partners — not a permissionless validator set. Plans for broader decentralisation [NOT FOUND] in public docs as of April 2026.
- Custodian Module requires KYC contact submission, meaning anonymous domain ownership is not possible for tokenised domains.

### Privacy

- ICANN's Whois/registrant data requirements mean on-chain domain ownership cannot be fully pseudonymous through Doma. The Custodian Module explicitly enforces contact information collection.
- DNS resolution itself remains on traditional infrastructure — privacy properties of DNS (e.g., DNS-over-HTTPS, DNS-over-TLS) are not addressed by the protocol.

### Censorship Resistance

- The Compliance Module's Forced Detokenisation mechanism directly enables state-level or court-ordered censorship of on-chain domain assets. This is by design for ICANN compliance but is antithetical to censorship-resistant naming.
- Domain names tokenised via Doma can be seized through the same legal mechanisms as traditional DNS domains (UDRP, court orders, government seizure).

### Dependency on Registrar Cooperation

- The protocol requires ICANN-accredited registrars to integrate. If a registrar withdraws or loses accreditation, the domains they manage could lose on-chain/off-chain synchronisation.

### Token Value and Speculation Risk

- Testnet metrics (705k domains in August 2025 vs 200k+ at mainnet) suggest significant testnet activity may have been synthetic or incentivised. The gap between testnet scale and mainnet activity is [NOT CONFIRMED].

[Sources: https://messari.io/report/doma-domains-galore; https://docs.doma.xyz/readme/protocol-overview; https://ens.domains/blog/post/d3-doma]

## Comparators

- [[projects/ens]] — Web3-native naming; no ICANN compliance; no DNS RWA tokenisation
- [[projects/unstoppable-domains]] — Web3 TLDs; not ICANN-approved; no DNS sync
- [[projects/handshake]] — decentralised DNS root alternative; not ICANN-compliant
- [[projects/sns]] — Solana Name Service; single-chain; no traditional DNS integration

## Tags

#naming #rwa #domainfi #icann #dns #layer2 #op-stack #cross-chain #ens-integration #defi
