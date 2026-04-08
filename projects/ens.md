---
tags: [project, ethereum, naming]
ecosystem: Ethereum
category: Infrastructure
website: https://ens.domains
docs: https://docs.ens.domains
launched: 2017
---

# Ethereum Name Service (ENS)

ENS is the dominant on-chain naming protocol on Ethereum, mapping human-readable `.eth` names (e.g. `alice.eth`) to machine addresses, content hashes, and arbitrary key-value records. Governed by the ENS DAO since November 2021, it processes approximately 3.2 million resolution requests per day and holds the largest user base of any blockchain naming system. ENSv2, an architectural upgrade delivering a hierarchical registry and improved cross-chain resolution, is under active development and will deploy exclusively on Ethereum mainnet after ENS Labs abandoned its planned Layer 2 rollup (Namechain) in February 2026.

## Adoption metrics

| Metric | Value | Date | Source |
|--------|-------|------|--------|
| Total names ever registered (cumulative) | ~2.8M (2022 peak year) | Jan 2023 | [CoinDesk](https://www.coindesk.com/markets/2023/01/04/ethereum-name-service-recorded-over-28m-domain-registrations-in-2022) |
| Active (non-expired) domains | ~910K | Late 2025 | [CoinMarketCap AI summary](https://coinmarketcap.com/cmc-ai/ethereum-name-service/latest-updates/) |
| Unique owner addresses | ~838K | Jan 2024 | [ChainCatcher](https://www.chaincatcher.com/en/article/2115348) |
| Daily resolution requests | ~3.2M | Feb 2026 | [CryptoNewsNavigator](https://www.cryptonewsnavigator.com/academy/article/why-ens-governance-decides-ethereums-identity-layer-future) |
| ENS DAO annual revenue (2024) | $28.77M | 2024 full year | [ENS DAO Governance Forum](https://discuss.ens.domains/t/ens-revenue-reports/20577/2) |
| ENS DAO Q1 2025 revenue | $4.94M | Q1 2025 | [ENS DAO Governance Forum](https://discuss.ens.domains/t/ens-revenue-reports/20577/2) |
| Total ETH revenue since launch | ~43.8K ETH | Oct 2024 | [Protocol Revenue ENS DAO Basics](https://basics.ensdao.org/protocol-revenue) |
| Endowment initial size | 16,000 ETH | Mar 2023 | [The Block](https://www.theblock.co/post/205418/ethereum-name-service-mulls-initiating-endowment-fund-with-17-million) |
| Ecosystem integrations (wallets, dApps) | 500+ (historical figure) | 2022 | [ENS Domains](https://ens.domains) |

Note: active domain counts differ from cumulative registrations because names expire if not renewed; ~910K active in late 2025 reflects renewals outpacing new registrations during a lower-demand period.

## Fee structure

Registration and renewal fees are denominated in USD and paid in ETH at the current Chainlink oracle price:

| Name length | Annual fee (USD) |
|-------------|-----------------|
| 5+ characters | $5 |
| 4 characters | $160 |
| 3 characters | $640 |
| 1-2 characters | Not available for registration |

Higher fees for short names act as an anti-squatting mechanism and reflect scarcity. An on-chain Chainlink ETH/USD price oracle converts the USD price to ETH at registration time. Gas fees are additional. As of early 2026, Ethereum gas limit growth has reduced registration gas cost by approximately 99% (from ~$5 to ~$0.05 average), making on-chain registration highly accessible.

Source: [ENS fee docs](https://support.ens.domains/en/articles/7900605-fees); [ENS Pricing](https://support.ens.domains/en/articles/12238910-ens-pricing)

## How it works

### User perspective

1. User visits an ENS-integrated app (e.g. app.ens.domains) and searches for an available `.eth` name.
2. User commits a hash of the intended name (two-step registration prevents frontrunning: a 1-minute wait between commit and reveal).
3. User registers the name by paying the annual fee in ETH plus a gas fee. The name is minted as an ERC-721 NFT.
4. User optionally sets a **primary name** (reverse record) so apps display their name instead of a raw address.
5. User sets records: ETH address, other coin addresses (via ENSIP-9 multi-coin), content hash, avatar, email, URL, and arbitrary text key-value pairs.
6. Any app performing address lookup calls the ENS resolver with the namehashed name to retrieve records.
7. Owner must renew before expiry or the name enters a **90-day grace period**, then a **21-day Dutch auction** (decaying premium starting at $2,000), then open re-registration.

### Protocol perspective

Key components:

- **ENS Registry** (EIP-137): a single Ethereum smart contract holding a mapping of `node => {owner, resolver, TTL}`. The node is derived from the name via the [[patterns/ens-namehash]] algorithm.
- **Resolver contracts**: separate contracts that store actual records. The Public Resolver supports address records (multi-coin via ENSIP-9), content hashes, text records, ABI, public keys, and name records. Each name may point to any resolver that implements the required interface.
- **ETH Registrar**: the registrar controller managing `.eth` second-level domain registration and renewal, including the two-step commit-reveal anti-frontrunning flow.
- **Name Wrapper**: an ERC-1155 contract that wraps ENS names as NFTs, enables fusing permissions (making names non-transferable or non-subdomain-creatable), and supports subname management with configurable expiry.
- **Reverse Registrar**: maps Ethereum addresses back to ENS names, enabling primary name (reverse resolution) lookups.

Resolution flow:
1. Input name is normalised per ENSIP-15 (unicode normalisation) and then hashed via the namehash algorithm.
2. The ENS Registry is queried with the node to retrieve the resolver address.
3. The resolver is queried with the node for the requested record type (e.g. `addr(bytes32 node, uint256 coinType)`).
4. For off-chain or L2 names: if the resolver reverts with an `OffchainLookup` error (EIP-3668 / CCIP-Read), the client follows the gateway URL, fetches the off-chain data, and calls the callback with the proof. See [[patterns/ccip-read-offchain-resolution]].

## Key behaviours

- [[patterns/ens-namehash]] - deterministic conversion of human-readable names to 32-byte nodes
- [[patterns/ccip-read-offchain-resolution]] - off-chain data retrieval with on-chain verification via EIP-3668
- [[patterns/ens-name-expiry-renewal]] - name lifecycle, grace period, Dutch auction re-registration
- [[patterns/ens-wildcard-resolution]] - dynamic subname resolution without per-subname on-chain registration
- [[patterns/ens-reverse-resolution]] - address-to-name mapping (primary names) and privacy implications
- [[patterns/ens-governance]] - DAO token-weighted governance over protocol parameters and treasury

## Architecture decisions

- **Registry/Resolver separation:** the Registry only stores ownership and resolver pointer; all records live in swappable Resolver contracts. This allows protocol upgrades without migrating ownership data.
- **Namehash for namespace security:** the node derivation is hierarchical and deterministic. Parent node owners control child nodes, preventing impersonation across levels.
- **Two-step commit-reveal registration:** prevents frontrunners from seeing an intended name in the mempool and registering it first. The 1-minute wait between commit and reveal enforces this.
- **USD-denominated fees paid in ETH:** decouples fee revenue from ETH price volatility. Uses Chainlink oracle on-chain, so the peg is trustless.
- **Wildcard resolution (ENSIP-10):** allows a single resolver to serve dynamic subnames without individual on-chain entries. Enables scalable subname issuance (e.g. `name.company.eth` for any `name`).
- **CCIP-Read (EIP-3668):** standard for off-chain and L2 data retrieval. Allows resolvers to defer lookups to gateway servers while enabling client-side verification. Enables L2 and database-backed names with no on-chain write cost.
- **Abandoned Namechain L2 (February 2026):** ENS Labs had planned a dedicated L2 ZK-rollup (Namechain, built on Taiko/Surge) for sub-$0.01 registration. In February 2026, the decision was reversed: Ethereum L1 gas costs had fallen ~99% (from ~$5 to ~$0.05 per registration) due to gas limit increases (30M to 60M in 2025, targeting 200M in 2026). ENSv2 will deploy exclusively on Ethereum mainnet. Source: [The Block](https://www.theblock.co/post/388932/ens-labs-scraps-namechain-l2-shifts-ensv2-fully-ethereum-mainnet); [CoinTelegraph](https://cointelegraph.com/news/ens-abandons-plan-namechain-will-stay-on-ethereum)
- **ENSv2 hierarchical registry:** each `.eth` name gets its own personal registry contract, enabling fine-grained permission delegation, instant expiry handling, and flexible role management. Source: [ENS Blog ENSv2](https://blog.ens.domains/post/ensv2)

## Differentiators

- **First-mover and de facto standard**: launched 2017, ENS has the deepest wallet and dApp integrations of any naming system. The `.eth` suffix is recognised broadly.
- **DAO governance with on-chain treasury**: unlike Unstoppable Domains (private company) or Handshake (leaderless), ENS DAO governs fee parameters, treasury, and protocol upgrades. The endowment generates additional DeFi yield to sustain the protocol.
- **Composable via CCIP-Read**: any L2 or off-chain system can plug in as a resolver gateway, giving ENS chain-agnostic reach without abandoning Ethereum as the trust anchor.
- **ENSIP improvement process**: an open standards track for protocol extensions (multi-coin addresses, content hash, gasless DNS import, multichain primary names).
- **Gasless DNS import (ENSIP-17)**: traditional DNS domain owners can link their `.com`, `.org`, etc. to ENS records by setting a DNSSEC TXT record, with zero on-chain gas cost. GoDaddy offers one-click integration. Source: [ENS Blog gasless DNSSEC](https://ens.domains/blog/post/gasless-dnssec)
- **Name Wrapper for subname management**: enterprises can issue subnames (`user.company.eth`) with configurable permissions, expiry, and NFT representation.

## Limitations and criticisms

- **Speculative squatting**: premium short names (3-4 chars) and common words are heavily squatted. The decaying-price Dutch auction on expired names mitigates re-registration speculation but does not address existing hoarding.
- **Centralisation in CCIP-Read gateways**: off-chain resolver gateways are URLs embedded in smart contracts. The gateway operator controls data availability; if a gateway goes offline or censors queries, name resolution fails. The URL in the contract can be changed by the resolver owner, introducing a trust dependency. Source: [ENS Docs offchain resolvers](https://docs.ens.domains/resolvers/ccip-read/)
- **Resolver privacy**: traditional name resolution (including ENS) exposes every lookup to the resolver operator. The resolver can log which addresses query which names. A grant proposal for verifiable privacy-preserving ENS resolution was submitted in 2024 but no production implementation exists as of April 2026. Source: [ENS DAO Forum privacy grant](https://discuss.ens.domains/t/grant-proposal-verifiable-and-privacy-preserving-ens-resolution/21108)
- **Unicode homograph attacks**: ENS names support unicode, enabling visually identical look-alike names from different scripts (e.g. Cyrillic 'a' vs Latin 'a'). ENSIP-15 normalisation mitigates most cases but requires correct implementation in every client. MetaMask raised this as a security concern. Source: [SEAL ENS name handling](https://frameworks.securityalliance.org/ens/name-handling-normalization/)
- **Grace period attack surface**: expired names during their 90-day grace period can have ownership transferred if the previous owner neglects renewal, enabling takeover of previously established names. This is documented in academic literature. Source: [arXiv ENS good bad ugly](https://arxiv.org/pdf/2104.05185)
- **Governance token concentration**: ENS DAO uses token-weighted voting. Large holders and delegates wield disproportionate influence. Academic research on DAO voting power distributions notes this as a structural concern. Source: [ScienceDirect DAO voting power](https://www.sciencedirect.com/article/pii/S2096720924000216)
- **Ethereum-only trust anchor**: despite CCIP-Read enabling multi-chain records, ENS name ownership and security guarantees are rooted in Ethereum L1. Chains that do not trust Ethereum cannot use ENS as a trust anchor.

## Sources

- [ENS Documentation](https://docs.ens.domains) — accessed 2026-04-08
- [ENS Blog: ENSv2](https://blog.ens.domains/post/ensv2) — accessed 2026-04-08
- [ENS Blog: ENSv2 Update](https://ens.domains/blog/post/ensv2-update) — accessed 2026-04-08
- [ENS Blog: Gasless DNSSEC](https://ens.domains/blog/post/gasless-dnssec) — accessed 2026-04-08
- [The Block: ENS scraps Namechain](https://www.theblock.co/post/388932/ens-labs-scraps-namechain-l2-shifts-ensv2-fully-ethereum-mainnet) — accessed 2026-04-08
- [CoinTelegraph: ENS abandons Namechain](https://cointelegraph.com/news/ens-abandons-plan-namechain-will-stay-on-ethereum) — accessed 2026-04-08
- [CoinDesk: 2.8M registrations 2022](https://www.coindesk.com/markets/2023/01/04/ethereum-name-service-recorded-over-28m-domain-registrations-in-2022) — accessed 2026-04-08
- [ENS DAO Revenue Reports](https://discuss.ens.domains/t/ens-revenue-reports/20577/2) — accessed 2026-04-08
- [ENS DAO Basics: Protocol Revenue](https://basics.ensdao.org/protocol-revenue) — accessed 2026-04-08
- [ENS Support: Fees](https://support.ens.domains/en/articles/7900605-fees) — accessed 2026-04-08
- [ENS Support: Name Lifecycle](https://support.ens.domains/en/articles/8046877-eth-name-lifecycle) — accessed 2026-04-08
- [ENS Support: Grace Period](https://support.ens.domains/en/articles/8046905-grace-period) — accessed 2026-04-08
- [EIP-137: ENS Specification](https://eips.ethereum.org/EIPS/eip-137) — accessed 2026-04-08
- [EIP-3668: CCIP-Read](https://eips.ethereum.org/EIPS/eip-3668) — accessed 2026-04-08
- [ENSIP-10: Wildcard Resolution](https://docs.ens.domains/ensip/10/) — accessed 2026-04-08
- [ENSIP-15: Name Normalization](https://docs.ens.domains/ensip/15/) — accessed 2026-04-08
- [ENSIP-17: Gasless DNS Resolution](https://docs.ens.domains/ensip/17/) — accessed 2026-04-08
- [arXiv: ENS Good, Bad and Ugly](https://arxiv.org/pdf/2104.05185) — accessed 2026-04-08
- [SEAL: ENS Name Handling and Normalization](https://frameworks.securityalliance.org/ens/name-handling-normalization/) — accessed 2026-04-08
- [CryptoNewsNavigator: ENS Governance](https://www.cryptonewsnavigator.com/academy/article/why-ens-governance-decides-ethereums-identity-layer-future) — accessed 2026-04-08
- [ENS DAO Forum: Privacy Grant Proposal](https://discuss.ens.domains/t/grant-proposal-verifiable-and-privacy-preserving-ens-resolution/21108) — accessed 2026-04-08
- [ENS DAO Endowment FAQ](https://basics.ensdao.org/endowment) — accessed 2026-04-08
- [CoinDesk: ENS scraps rollup (Vitalik warning)](https://www.coindesk.com/tech/2026/02/06/ethereum-s-ens-identity-system-scraps-planned-rollup-amid-vitalik-s-warning-about-layer-2-networks) — accessed 2026-04-08
