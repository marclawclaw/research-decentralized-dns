# Pattern: Dual-Token Name Ownership

## Summary

A name system separates two distinct rights using two distinct token types: (1) ownership/title rights (transferable, collaterable, fractionalisable) and (2) operational/service rights (DNS settings, routing, active service configuration). This allows the ownership asset to participate in financial markets while the underlying service continues to function normally.

## Observed In

- [[projects/d3-doma]] (Doma Protocol) — DOT (Domain Ownership Token) and DST (Domain Service Token)

## Motivation

In traditional DNS, ownership and DNS control are conflated. The registrant who owns the domain also controls the DNS records. To trade or collateralise the domain as a financial asset, you must also hand over operational control of the DNS, which creates service disruption risk.

Separating these into two tokens solves the problem: a domain can be sold or used as DeFi collateral without affecting email delivery, website routing, or other live DNS services.

## Token Definitions (Doma)

| Token | Type | Rights | Transferable | DeFi Use |
|-------|------|--------|-------------|----------|
| DOT (Domain Ownership Token) | NFT (ERC-721 equivalent) | Title, transfer rights, ultimate control | Yes | Trade, collateral, fractionalization |
| DST (Domain Service Token) | Fungible / operational | DNS record management, email routing, name server config | Separate from DOT | Utility staking |

[Source: https://docs.doma.xyz/readme/protocol-overview; https://messari.io/report/doma-domains-galore]

## How Ownership Transfer Works

1. Seller lists or transfers the DOT on a marketplace or DeFi protocol.
2. The DOT transfer triggers the Custodian Module: registrar notifies the new owner to submit contact information.
3. During the handover period, the DST (and thus DNS operations) remains with the original configuration unless explicitly transferred.
4. Once the new owner completes the claim process, they gain control over both DOT and DST.
5. DNS records remain live throughout; there is no service interruption.

[Source: https://docs.doma.xyz/readme/protocol-overview]

## Fractionalization Extension

The Domain Partitioning Module extends the DOT by locking it into a fractionalization contract and minting fungible ERC-20 tokens representing partial ownership shares. Payments for fractional trading are handled in bridged USDC. The underlying DNS domain continues to operate via the DST, independent of how many fractional tokens are outstanding.

[Source: https://messari.io/report/doma-domains-galore]

## Behaviours That Could Become Requirements

| Behaviour | Implication for RFP |
|-----------|---------------------|
| Ownership token is separate from service token | A name asset can be traded without disrupting the active service record |
| DNS continues to function during ownership transfer | No downtime for websites or email during domain trading |
| Fractional ownership is possible without DNS fragmentation | Multiple parties can hold economic exposure to a name without creating DNS record conflicts |
| Collateral seizure (e.g., liquidation) affects the DOT, not the DST directly | A liquidated domain keeps its DNS records intact until the new DOT owner claims service control |

## Analogies

- Real estate: title deed (DOT) vs. lease agreement / tenancy (DST). You can sell the building while the tenant's lease continues.
- Company shares vs. operating rights: shareholders own equity; board/management runs operations.

## Limitations

- Complexity: two tokens per domain multiplies the number of on-chain objects and transactions.
- Potential for misalignment: if DOT and DST ownership diverge (e.g., DOT sold but DST not reclaimed), the new DOT owner may not immediately have DNS control.
- The pattern is not applicable to purely on-chain names (e.g., .eth) where there is no off-chain DNS record to protect.

## Sources

- https://docs.doma.xyz/readme/protocol-overview (Doma Protocol Documentation, 2025)
- https://messari.io/report/doma-domains-galore (Messari: Doma Domains Galore, 2025)
- https://blog.doma.xyz/doma-mainnet-is-live/ (Doma Mainnet Launch Post, 25 November 2025)

## Tags

#naming #tokens #nft #defi #dns #ownership #pattern #dual-token
