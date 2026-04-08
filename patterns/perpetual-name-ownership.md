# Pattern: Perpetual Name Ownership

## Summary

A blockchain naming system issues domain names as NFTs with no expiry date and no renewal requirement. Ownership is perpetual once acquired: the name is held until the owner chooses to transfer or burn it. This contrasts with both traditional DNS (annual renewal required) and ENS (annual rent). The one-time payment model is the primary user-facing differentiator for systems that adopt it.

## Observed In

- [[projects/unstoppable-domains]] (primary example; Ethereum/Polygon; all Web3 TLDs)

## How It Works

### Mechanism

1. The user pays a one-time fee at purchase time. The fee is set by the issuing company (Unstoppable Domains) and varies by name desirability and TLD. Prices typically range from a few dollars for standard names to thousands for premium names. [Source: https://unstoppabledomains.com/blog/categories/education/article/dns-fees]
2. The name is minted as an ERC-721 NFT and transferred to the buyer's wallet.
3. The smart contract Registry has no expiry logic and no admin key. Neither the issuing company nor any third party can reclaim or expire the name.
4. The owner may transfer the NFT freely on any compatible marketplace (OpenSea, etc.) without involving the issuing company.
5. No on-chain action is required to maintain ownership; the token simply persists.

### Economic Logic

From the protocol's perspective, perpetual ownership converts a recurring subscription (traditional DNS) into a one-time sale. This:
- Eliminates churn from users forgetting to renew (a major source of domain loss in traditional DNS).
- Creates a secondary market where the issuer has no direct stake (no transfer royalties in the base UNS contract, though marketplaces may apply them).
- Front-loads all revenue into the initial sale; there is no recurring revenue stream from existing names.

### Contrast with ENS

ENS charges an annual rent (approximately $5/year for five-character names; $160/year for three-character names as of 2024). This creates:
- A recurring revenue model for the ENS DAO.
- A natural name recycling mechanism: abandoned names expire and become available.
- A financial barrier to hoarding large numbers of names indefinitely.

Unstoppable Domains' model has no recycling mechanism. Names that are purchased and never used remain permanently off the market unless the owner sells them.

## Behaviours That Could Become Requirements

| Behaviour | Implication for RFP |
|-----------|---------------------|
| No admin key for revocation | The registry smart contract must be deployed without an owner/admin that can modify or transfer records |
| No expiry block logic | The ERC-721 token must have no time-based validity or expiry function |
| One-time issuance fee | The protocol must support a minting/purchase event that does not require subsequent fee payments |
| Secondary market tradability | The NFT must conform to ERC-721 so it is tradeable on existing NFT marketplaces |
| No name recycling | Names are permanently removed from the available pool once minted; the protocol must handle namespace exhaustion over the long term |

## Privacy Implications

- Perpetual ownership means a wallet address remains permanently associated with a name on-chain, with no natural anonymity refresh cycle (unlike ENS where names expire and identities can be cycled).
- A user who links their real identity to a perpetual name cannot "let it expire" to sever that link; they must actively transfer or burn the name.

## Censorship Resistance Implications

- The absence of an admin key means the issuing company cannot comply with legal orders to revoke or transfer a Web3 name. This is a genuine censorship-resistance property for existing names.
- However, the whitelisted minting system used by Unstoppable Domains means the issuer can block new registrations (e.g., trademarked names, sanctioned parties). Perpetual ownership applies only after minting; the issuance process remains censorship-capable.

## Risks and Limitations

### Namespace Squatting

Without renewal costs, the economic disincentive to hold unused names is eliminated. Large-scale acquisition of speculative names (cryptosquatting) is low-cost once the initial purchase is made.

### No Dispute Resolution Mechanism

Traditional DNS has UDRP for trademark disputes. A perpetual-ownership system with no admin key has no equivalent enforcement mechanism; a squatted name cannot be transferred via legal order.

### Issuer Revenue Model

Front-loaded revenue means the issuer depends on new name sales. If demand for new names plateaus, the issuer has no recurring revenue from the existing name base. This may create pressure to expand TLDs or pivot to other business models (as observed with Unstoppable Domains' shift to traditional DNS).

### Irrecoverability

If a user loses their private key, the name is permanently inaccessible. There is no recovery mechanism because there is no admin key. This is particularly consequential for perpetual names, which cannot simply be let expire.

## Sources

- https://docs.unstoppabledomains.com/smart-contracts/overview/uns-architecture-overview (UNS Architecture Overview, Unstoppable Domains Developer Portal)
- https://medium.com/unstoppabledomains/renewal-fee-announcement-940617d0a592 (Renewal Fee Announcement, Unstoppable Domains Medium, 2021)
- https://unstoppabledomains.com/blog/categories/education/article/dns-fees (How Much Does a Domain Name Cost?, Unstoppable Domains Blog)
- https://rulemobile.com/news/web3-domains-unstoppable-own-your-name-foreverno-renewals-no-renting (Web3 Domains: Own Your Name Forever, Rule Mobile)
- https://coinbureau.com/review/unstoppable-domains (Unstoppable Domains Review, Coin Bureau)

## Tags

#naming #perpetual-ownership #no-renewal #nft #erc-721 #censorship-resistance #namespace #pattern
