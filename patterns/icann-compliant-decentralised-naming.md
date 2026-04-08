# Pattern: ICANN-Compliant Decentralised Naming

## Summary

A naming protocol designed to tokenise and trade ICANN-governed DNS domains on a blockchain, while preserving all ICANN compliance obligations (registrant data, dispute resolution, transfer rules). Rather than competing with the DNS root, the protocol integrates with it.

## Observed In

- [[projects/d3-doma]] (Doma Protocol) — primary example; purpose-built L2

## How It Works

### Core Tension

ICANN compliance requires:
1. Valid registrant contact information for every domain.
2. Support for UDRP dispute resolution and court-ordered transfers.
3. Domain lifecycle synchronisation (registration, expiry, deletion reflected on-chain).

Decentralised ownership requires:
1. Pseudonymous or anonymous ownership (wallet address, no identity).
2. Immutable, uncensorable records.
3. Self-custodied assets not subject to forced transfer.

These requirements directly conflict. ICANN-compliant decentralised naming resolves the conflict by accepting ICANN's legal authority and adding compliance layers on top of on-chain ownership.

### Mechanism: Custodian Module / Registrar Handshake

When a Domain Ownership Token (DOT) is transferred on-chain:
1. The Doma smart contract notifies the registrar.
2. The registrar places the domain in a temporary proxy/custodian account.
3. The new owner must complete a claim process, providing the contact information ICANN requires (name, email, address).
4. Only after claim completion does the registrar update the Whois record and grant full DNS control.

This means on-chain transfer is immediate, but DNS control transfer is gated behind off-chain KYC/contact submission.

### Mechanism: Compliance Module / UDRP Enforcement

- **Transfer Lock:** During a UDRP investigation, the Compliance Module freezes the domain's DOT, preventing further on-chain transfer.
- **Forced Detokenisation:** Following a legal ruling, the DOT is burned on-chain and domain control returns to the registrar. The on-chain record is deliberately mutable to reflect legal reality.

### Mechanism: Dual Token System

See [[patterns/dual-token-name-ownership]].

- **DOT (Domain Ownership Token):** NFT; represents title and transfer rights; tradeable, collaterable.
- **DST (Domain Service Token):** Governs DNS operations (A records, MX records, routing). Separating these allows the ownership asset to circulate in DeFi while the domain continues to function normally.

## Behaviours That Could Become Requirements

| Behaviour | Implication for RFP |
|-----------|---------------------|
| Custodian handshake enforces registrant data collection | Any ICANN-linked name system must gate DNS control transfer behind identity verification |
| UDRP Transfer Lock is enforceable on-chain | On-chain name assets can be frozen by off-chain legal order; "decentralised" does not mean censorship-resistant in this model |
| Forced Detokenisation burns the on-chain asset | Name assets in this model are not censorship-resistant; they mirror the legal status of the underlying DNS domain |
| DNS domain continues to resolve during on-chain ownership transfer | DOT/DST separation ensures no DNS downtime during trading |
| Registrar remains in the trust path | Protocol relies on ICANN-accredited registrars; registrar withdrawal breaks on-chain/off-chain sync |

## Privacy Implications

- Full pseudonymity is not possible. Contact information must be submitted to the registrar before DNS control transfers.
- The on-chain ownership record (wallet address) is public; the registrant contact data lives with the registrar (off-chain), preserving some separation.
- GDPR/privacy regulations governing Whois data apply to the registrar, not to the on-chain protocol directly.

## Censorship Resistance Implications

- Doma Protocol explicitly supports government and court-ordered domain seizure through the Compliance Module.
- This is by design: ICANN compliance requires it.
- Domains tokenised via this pattern have the same censorship vulnerability as traditional DNS domains; the blockchain layer adds financial programmability but does not improve censorship resistance.

## Contrast With Censorship-Resistant Naming

| Dimension | ICANN-Compliant Model (Doma) | Censorship-Resistant Model (e.g., Handshake, ENS) |
|-----------|------------------------------|---------------------------------------------------|
| Root authority | ICANN / DNS root | Blockchain consensus |
| Forced seizure | Possible via Compliance Module | Not possible without key holder cooperation |
| Identity requirement | Registrant contact data required | Pseudonymous wallet address only |
| DNS resolver compatibility | Native (domain IS a DNS domain) | Requires custom resolver or browser extension |
| TLD availability | Any ICANN-approved TLD | Protocol-defined or auction-based |

## Sources

- https://docs.doma.xyz/readme/protocol-overview (Doma Protocol Documentation, 2025)
- https://messari.io/report/doma-domains-galore (Messari: Doma Domains Galore, 2025)
- https://ens.domains/blog/post/d3-doma (ENS Blog: Tokenized DNS Domains with Doma, October 2025)
- https://www.prnewswire.com/news-releases/d3-raises-25m-series-a-led-by-paradigm-announces-the-first-blockchain-for-internets-362m-domain-names-302362780.html (D3 Series A Press Release, 29 January 2025)

## Tags

#naming #icann #compliance #dns #rwa #censorship-resistance #privacy #pattern
