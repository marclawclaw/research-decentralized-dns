# Unstoppable Domains

## Summary

Unstoppable Domains (founded 2018, San Francisco) is a Web3 naming company that issues blockchain-based domain names as non-fungible tokens (ERC-721) on Ethereum and Polygon. Its primary differentiator is a one-time purchase model with no renewal fees, making names permanently owned assets rather than annually licensed handles. After gaining ICANN accreditation in August 2024, the company pivoted aggressively into traditional DNS registration; by early 2026, conventional DNS domains composed over 90% of its business by volume.

## Key Links

- Website: https://unstoppabledomains.com
- Docs: https://docs.unstoppabledomains.com
- Resolution library: https://github.com/unstoppabledomains/resolution
- UNS architecture: https://docs.unstoppabledomains.com/smart-contracts/overview/uns-architecture-overview
- CNS architecture: https://docs.unstoppabledomains.com/smart-contracts/overview/cns-architecture-overview/

## Classification

- **Category:** Web3 naming / digital identity / domain registrar
- **Ecosystem:** Ethereum (L1) + Polygon (L2); also ICANN-accredited DNS registrar
- **Stage:** Production; live since 2019 for Web3 names; DNS registrar since October 2024
- **Chain type:** Ethereum mainnet (legacy CNS) + Polygon PoS (current UNS)

## Team and Funding

- **CEO / Co-founder:** Matthew Gould
- **Co-founders:** Braden Pezeshki, Bradley Kam
- **Founded:** 2018

| Round | Date | Amount | Lead Investor | Source |
|-------|------|--------|---------------|--------|
| Seed | December 2018 | $4M | Draper Associates, Boost VC, Digital Currency Group | https://www.theblock.co/linked/24653/unstoppable-domains-raises-4m-to-create-a-decentralized-domain-registry |
| Series A | July 2022 | $65M | Pantera Capital | https://fortune.com/2022/07/27/unstoppable-domains-pantera-led-65-million-round-1-billion-valuation-web3-nft-domain/ |

**Total raised:** $69M (as of July 2022). Valuation at Series A: $1 billion (unicorn). [Source: https://fortune.com/2022/07/27/unstoppable-domains-pantera-led-65-million-round-1-billion-valuation-web3-nft-domain/]

Series A co-investors: Mayfield, OKG Investments, Polygon, Boost VC, Draper Associates. [Source: https://fortune.com/2022/07/27/unstoppable-domains-pantera-led-65-million-round-1-billion-valuation-web3-nft-domain/]

## Adoption Metrics

| Metric | Value | Date | Source |
|--------|-------|------|--------|
| Total Web3 domains registered (blockchain) | 4.6M+ | November 2025 | https://tracxn.com/d/companies/unstoppable-domains/__5yK6TVAPTXnNb2zNMdcTpmVHN7NvV6ZGoWi9vQdwYz8 |
| Total Web3 domains registered (blockchain) | 4.2M+ | April 2025 | https://tracxn.com/d/companies/unstoppable-domains/__5yK6TVAPTXnNb2zNMdcTpmVHN7NvV6ZGoWi9vQdwYz8 |
| DNS domains under management (traditional) | 750,000+ | March 2026 (13 months after starting) | https://domainnamewire.com/2026/03/12/unstoppable-domains-growth-the-past-and-what-happens-in-the-future/ |
| New .com DNS registrations generated in 2025 | ~500,000 | 2025 full year | https://domainnamewire.com/2026/03/12/unstoppable-domains-growth-the-past-and-what-happens-in-the-future/ |
| DNS TLDs onboarded | ~200 | Early 2026 | https://domainnamewire.com/2026/01/26/top-domain-registrars-namesilo-hits-milestone-unstoppable-domains-surges/ |
| Gas savings for users on Polygon | $100M | Cumulative to 2022 | https://polygon.technology/blog/unstoppable-domains-users-save-100-million-in-fees-on-polygon |
| Supported cryptocurrencies for resolution | 310+ | 2024 | https://docs.unstoppabledomains.com/resolution/overview |
| Registrar accreditations held (ICANN shell entities) | 10 (UnstoppableUS1–10 LLC) | 2025 | https://domainincite.com/31629-unstoppable-buys-10-new-registrars |
| Web3 TLD extensions offered | 10 | 2024 | https://cryptoninjas.net/2021/09/22/trust-wallet-adds-support-for-all-10-unstoppable-domains-crypto-name-extensions/ |

**Top Web3 TLDs by registration count (approximate, 2024):**

| TLD | Registrations | Source |
|-----|--------------|--------|
| .crypto | 108,000 | https://beincrypto.com/learn/unstoppable-domains/ |
| .wallet | 85,000 | https://beincrypto.com/learn/unstoppable-domains/ |
| .nft | 77,000 | https://beincrypto.com/learn/unstoppable-domains/ |

Note: these per-TLD counts appear to be from 2024 and may not reflect the full 4.6M total, which likely spans all TLDs and includes older registrations. [NOT CONFIRMED — discrepancy between per-TLD totals and overall figure needs reconciliation]

## How It Works

### User Perspective

1. A user purchases a domain (e.g., `alice.wallet`) from unstoppabledomains.com via a one-time payment. Prices range from approximately $5 for standard names to thousands for premium names. [Source: https://unstoppabledomains.com/blog/categories/education/article/dns-fees]
2. The domain is minted as an ERC-721 NFT on Polygon (current default) and sent to the user's wallet. Minting, gas, and renewal fees are all zero on Polygon. [Source: https://unstoppabledomains.com/blog/polygon-l2-solution]
3. The user sets records on the domain: crypto payment addresses (for 310+ currencies), IPFS content hashes for decentralised websites, and profile metadata.
4. The domain is permanently owned. No annual renewal is required; the NFT cannot be clawed back even by Unstoppable Domains. [Source: https://docs.unstoppabledomains.com/smart-contracts/overview/uns-architecture-overview]
5. The user can log into Web3 applications using "Login with Unstoppable," a single sign-on (SSO) feature built on OpenID Connect (OIDC) extended with a wallet signature. [Source: https://support.unstoppabledomains.com/support/solutions/articles/48001215516-what-is-login-with-unstoppable-]

### Protocol Perspective

#### Smart Contract Systems

Unstoppable Domains has operated two smart contract architectures:

**CNS (Crypto Name Service) — legacy, Ethereum L1:**
- Two contracts: a `Registry` (maps domain names to owner address and resolver address) and a `Resolver` (maps domain names to records).
- A `ProxyReader` contract consolidates multiple on-chain lookups into a single call for resolution libraries.
- Domains are ERC-721 tokens identified by a namehash of the domain string.
- [Source: https://docs.unstoppabledomains.com/smart-contracts/overview/cns-architecture-overview/]

**UNS (Unstoppable Name Service) — current, Polygon:**
- Single `Registry` contract handles both ownership and record storage.
- Every domain is an ERC-721 token; the Registry is the canonical authority for all UNS domains.
- No separate Resolver contract; records are stored directly in the Registry.
- The Registry smart contract has no admin key: no entity, including Unstoppable Domains, can transfer or manage a domain without the owner's private key.
- [Source: https://docs.unstoppabledomains.com/smart-contracts/overview/uns-architecture-overview]

#### Resolution Flow

Resolution can be performed via three methods:

1. **Direct blockchain call:** Query the `ProxyReader` (CNS) or `Registry` (UNS) contract with the domain's namehash and desired record keys. Requires an Ethereum or Polygon RPC node. Fully trustless and on-chain.
2. **Resolution library:** Unstoppable's open-source `resolution` library (JavaScript, Swift, Java) wraps the blockchain calls. [Source: https://github.com/unstoppabledomains/resolution]
3. **Resolution Service API:** A centralised HTTP API for developers who do not want to run their own node. Simpler to integrate but introduces a trusted intermediary. [Source: https://docs.unstoppabledomains.com/apis/resolution/openapi/domains]

#### Domain Identification

Each domain is identified by its **namehash**: a recursive Keccak-256 hash of the domain's labels (e.g., `keccak256(keccak256('') || keccak256('wallet')) || keccak256('alice')`). This uint256 value is the ERC-721 `tokenId`. [Source: https://docs.unstoppabledomains.com/smart-contracts/overview/uns-architecture-overview]

#### Minting Control

New second-level domains (e.g., `alice.x`) are minted only by **whitelisted minters**, which are Unstoppable Domains-controlled accounts. This means the company controls domain issuance even though it cannot control existing domains after minting. [Source: https://docs.unstoppabledomains.com/smart-contracts/overview/uns-architecture-overview]

## Supported TLDs (Web3)

The 10 Web3 TLD extensions offered (all on Ethereum/Polygon): `.crypto`, `.nft`, `.wallet`, `.coin`, `.bitcoin`, `.dao`, `.blockchain`, `.888`, `.x`, `.zil`.

[Source: https://cryptoninjas.net/2021/09/22/trust-wallet-adds-support-for-all-10-unstoppable-domains-crypto-name-extensions/]

## Key Behaviours

### Perpetual Ownership (No Renewal)

The single most-cited differentiator. Once minted, a UNS domain NFT is owned permanently by the holder's wallet. No renewal transaction or fee is required. Neither Unstoppable Domains nor any third party can reclaim the domain. See [[patterns/perpetual-name-ownership]].

### Polygon Migration for Zero-Gas Minting

Unstoppable Domains migrated from Ethereum L1 to Polygon in November 2021, eliminating gas fees for minting and management. Users who had paid up to $100 in gas on L1 could migrate to L2 for free. The migration was phased: first minting, then domain management, then bridging existing assets. See [[patterns/l2-migration-for-naming]].

### Login with Unstoppable (Web3 SSO)

A domain owner can use their NFT domain as a portable Web3 identity across applications. Login with Unstoppable extends OIDC with a wallet signature instead of a password. Users can selectively share off-chain profile metadata (email, avatar, social handles) with each app. Data is stored off-chain and shared only via a signed wallet message. This is free for both users and integrating applications. [Source: https://support.unstoppabledomains.com/support/solutions/articles/48001215516-what-is-login-with-unstoppable-]

### Reverse Resolution

A wallet address can be resolved to a domain name (reverse resolution), enabling applications to display human-readable names rather than hex addresses. [Source: https://docs.unstoppabledomains.com/resolution/guides/reverse-resolution/overview/]

### ICANN Dual-Track Strategy

After ICANN accreditation in August 2024, Unstoppable Domains began selling traditional DNS domains alongside its Web3 TLDs. By early 2026, CEO Matthew Gould stated that DNS domains made up more than 90% of the business by volume, and that within two to three years, DNS could account for 99%+. The company is also supporting 19+ Web3 companies to apply for ICANN gTLDs in the 2026 application round, seeking to legitimise its own Web3 TLDs (`.x`, `.wallet`, `.crypto`, `.nft`, `.dao`) in the traditional DNS root. [Source: https://domainnamewire.com/2026/03/12/unstoppable-domains-growth-the-past-and-what-happens-in-the-future/; https://circleid.com/posts/20240814-unstoppable-domains-receives-icann-accreditation]

## Architecture Decisions

### Single Registry, No Admin

The choice to deploy a Registry contract without an admin key means Unstoppable Domains deliberately gave up the ability to revoke or transfer domains. This was a trust-building decision but also means the company cannot comply with UDRP dispute rulings for Web3 TLDs (unlike the Doma Protocol's [[patterns/icann-compliant-decentralised-naming]] approach). [Source: https://docs.unstoppabledomains.com/smart-contracts/overview/uns-architecture-overview]

### Whitelisted Minting

Although domain records are immutable post-mint, new domain issuance is permissioned: only Unstoppable Domains-controlled addresses can mint new domains. This means:
- The supply of any given name is controlled by the company.
- A hypothetical `alice.crypto` can only be registered if Unstoppable Domains permits it (e.g., they block trademarked names via an API check).
- This is a centralised chokepoint for new registrations.

### Resolution Layer Choice

The existence of three resolution methods (direct blockchain, resolution library, HTTP API) gives developers options across the trustlessness spectrum. However, most integrations default to the Resolution Service API for simplicity, which reintroduces a centralised dependency. Browser native support (Brave since v1.40; Opera for `.crypto` only) uses direct blockchain resolution. [Source: https://support.brave.app/hc/en-us/articles/39196019032973-Resolve-Methods-for-Unstoppable-Domains; https://support.unstoppabledomains.com/support/solutions/articles/48001252805-resolving-domains-in-brave]

### Off-Chain Identity Data

Profile metadata (email, avatar, social handles) for Login with Unstoppable is stored off-chain, shared only with user consent via signed wallet message. This separates the public on-chain record (wallet address owns token) from private identity data. [Source: https://unstoppabledomains.com/blog/categories/announcements/article/data-privacy-and-ownership-at-unstoppable]

## Differentiators

| Differentiator | Detail |
|---------------|--------|
| No renewal fees | One-time purchase; permanent ownership; no annual cost |
| Zero gas on Polygon | Minting, transferring, and updating records are free on Polygon |
| ICANN accreditation | Legitimate registrar status; sells traditional DNS domains alongside Web3 names |
| Web3 SSO | "Login with Unstoppable" provides portable identity via OIDC + wallet signature |
| 310+ currency resolution | Single domain resolves to payment addresses for 310+ cryptocurrencies |
| Browser-native resolution | Brave (all 10 TLDs) and Opera (.crypto only) resolve natively without extensions |
| Patent protection | Holds US Patent No. 11,558,344 ("Resolving Blockchain Domains"); USPTO upheld it against ENS challenge in November 2025 |

## Limitations and Criticisms

### Centralised Minting Chokepoint

New domain issuance requires whitelisted minter accounts controlled by Unstoppable Domains. The company can and does block registration of domains it deems infringing (trademark holders, known persons/businesses identified via API check). This directly contradicts claims of censorship-resistance. [Source: https://coinrivet.com/unstoppable-domains-is-censoring-domains/]

### Centralised Resolution API

The Resolution Service API is a centralised HTTP endpoint. Most dApps use it rather than direct blockchain calls. If the API goes down or Unstoppable Domains decides to block a domain, resolution fails at the application layer even though the on-chain record exists. [Source: https://docs.unstoppabledomains.com/apis/resolution/openapi/domains]

### No Decentralised Dispute Resolution

Web3 TLDs have no UDRP equivalent. Unstoppable Domains does not support forced domain transfer via legal ruling for its Web3 names (the registry has no admin key). This creates a trademark squatting and cryptosquatting risk. [Source: https://www.ccn.com/education/crypto/unstoppable-domains-explained/]

### Walled Garden for DNS TLDs

The Web3 TLDs (`.crypto`, `.wallet`, etc.) are not in the ICANN DNS root. They only resolve in browsers that natively support them (Brave, Opera) or via the Unstoppable browser extension. Standard DNS resolvers, ISP resolvers, and most browsers do not resolve them. This significantly limits practical utility.

### Privacy: On-Chain Ownership is Public

All domain ownership (wallet address to token) is publicly visible on-chain. On-chain records (crypto addresses, IPFS hashes) are also public. Linking a domain name to a real identity exposes all associated wallet addresses and IPFS content. [Source: https://unstoppabledomains.com/blog/categories/announcements/article/data-privacy-and-ownership-at-unstoppable]

### Business Pivot Risk

The company's rapid pivot to traditional DNS (90%+ of revenue by early 2026) raises questions about long-term investment in Web3 naming infrastructure. If traditional DNS becomes the core business, Web3 TLD support could be deprioritised.

### Patent Controversy

Unstoppable Domains holds a patent on blockchain domain resolution (US 11,558,344). ENS petitioned the USPTO to invalidate it in 2023, arguing the patent covered ENS's own prior art. The USPTO denied ENS's petition in November 2025. Unstoppable Domains has stated it will not enforce the patent aggressively, but the patent creates IP risk for any open-source blockchain naming project. [Source: https://unstoppabledomains.com/blog/categories/announcements/article/patent-update; https://blockworks.co/news/ens-unstoppable-domains-paten]

## Sources

- https://docs.unstoppabledomains.com/smart-contracts/overview/uns-architecture-overview (UNS Architecture Overview, Unstoppable Domains Developer Portal)
- https://docs.unstoppabledomains.com/smart-contracts/overview/cns-architecture-overview/ (CNS Architecture Overview, Unstoppable Domains Developer Portal)
- https://docs.unstoppabledomains.com/resolution/overview (Resolution Overview, Unstoppable Domains Developer Portal)
- https://github.com/unstoppabledomains/resolution (Resolution library, GitHub)
- https://unstoppabledomains.com/blog/polygon-l2-solution (Unstoppable + Polygon: Hello L2, Goodbye Gas Fees!)
- https://polygon.technology/blog/unstoppable-domains-users-save-100-million-in-fees-on-polygon (Unstoppable Domains Users Save $100 Million in Fees on Polygon, Polygon Technology)
- https://fortune.com/2022/07/27/unstoppable-domains-pantera-led-65-million-round-1-billion-valuation-web3-nft-domain/ (Unstoppable Domains notches $1 billion valuation, Fortune, July 2022)
- https://www.prnewswire.com/news-releases/unstoppable-domains-gains-icann-accreditation-becomes-largest-onchain-registrar-302222578.html (Unstoppable Domains Gains ICANN Accreditation, PR Newswire, August 2024)
- https://circleid.com/posts/20240814-unstoppable-domains-receives-icann-accreditation (Unstoppable Domains Receives ICANN Accreditation, CircleID, August 2024)
- https://domainnamewire.com/2026/03/12/unstoppable-domains-growth-the-past-and-what-happens-in-the-future/ (Unstoppable Domains growth: the past, and what happens in the future, Domain Name Wire, March 2026)
- https://domainnamewire.com/2026/01/26/top-domain-registrars-namesilo-hits-milestone-unstoppable-domains-surges/ (Top domain registrars: NameSilo hits milestone, Unstoppable Domains surges, Domain Name Wire, January 2026)
- https://domainincite.com/31629-unstoppable-buys-10-new-registrars (Unstoppable buys 10 new registrars, Domain Incite, 2025)
- https://support.unstoppabledomains.com/support/solutions/articles/48001215516-what-is-login-with-unstoppable- (What is "Login with Unstoppable"?, Unstoppable Domains Support)
- https://unstoppabledomains.com/blog/categories/announcements/article/data-privacy-and-ownership-at-unstoppable (Data Privacy and Ownership at Unstoppable, Unstoppable Domains Blog)
- https://coinrivet.com/unstoppable-domains-is-censoring-domains/ (Is Unstoppable Domains blocking users from buying certain domains?, Coin Rivet)
- https://blockworks.co/news/ens-unstoppable-domains-paten (ENS lead developer calls out Unstoppable Domain patents, Blockworks)
- https://unstoppabledomains.com/blog/categories/announcements/article/patent-update (Unstoppable Domains Wins US Patent Trademark Office Patent Challenge Against ENS, Unstoppable Domains Blog, November 2025)
- https://support.brave.app/hc/en-us/articles/39196019032973-Resolve-Methods-for-Unstoppable-Domains (Resolve Methods for Unstoppable Domains, Brave Help Center)
- https://tracxn.com/d/companies/unstoppable-domains/__5yK6TVAPTXnNb2zNMdcTpmVHN7NvV6ZGoWi9vQdwYz8 (Unstoppable Domains Company Profile, Tracxn, 2025)

## Tags

#naming #web3 #ethereum #polygon #nft #dns #icann #identity #sso #perpetual-ownership #l2
