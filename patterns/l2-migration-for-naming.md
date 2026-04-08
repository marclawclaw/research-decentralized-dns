# Pattern: L2 Migration for Naming

## Summary

A blockchain naming protocol initially deployed on Ethereum L1 migrates its domain minting and management operations to a Layer 2 network to eliminate gas fees for end users. The migration is phased and preserves backward compatibility: existing L1 names continue to function while new names are issued on L2. The central outcome is that gas costs, previously a significant barrier to adoption (up to $100 per mint on L1), drop to zero on L2.

## Observed In

- [[projects/unstoppable-domains]] (Ethereum L1 CNS to Polygon UNS, November 2021)

## How It Works

### Trigger

High and volatile Ethereum L1 gas fees make minting and managing blockchain domain names expensive for average users. For Unstoppable Domains, gas fees of up to $100 per mint in 2021 were a direct barrier to adoption. [Source: https://decrypt.co/86157/unstoppable-domains-taps-polygon-to-cut-gas-fees-for-ethereum-nft-domains]

### Mechanism (Unstoppable Domains Polygon Migration)

The migration was broken into four phases:

1. **Phase 1:** New domain minting moves to Polygon. New purchases are minted on Polygon for $0 in gas.
2. **Phase 2:** Domain management (updating records, setting crypto addresses) moves to Polygon.
3. **Phase 3:** Bridging of existing Zilliqa and Ethereum L1 assets: existing domain holders can migrate their L1 domains to L2.
4. **Phase 4:** Domains can also be bridged back from Polygon to Ethereum L1 if desired.

[Source: https://unstoppabledomains.com/blog/polygon-l2-solution; https://mobileapp.unstoppabledomains.com/learn-about/polygon-layer2/]

The new UNS (Unstoppable Name Service) smart contract architecture was deployed on Polygon, replacing the legacy CNS (Crypto Name Service) architecture on Ethereum L1. The UNS consolidates the Registry and Resolver into a single contract for efficiency. [Source: https://docs.unstoppabledomains.com/smart-contracts/overview/uns-architecture-overview]

### Why Polygon Specifically

Polygon PoS was chosen because:
- It is EVM-compatible, so Solidity contracts required minimal adaptation.
- Near-zero gas fees (fractions of a cent per transaction vs. up to $100 on L1 in 2021).
- Fast finality for user-facing operations.
- Polygon (the company) was also an investor in Unstoppable Domains' Series A round. [Source: https://fortune.com/2022/07/27/unstoppable-domains-pantera-led-65-million-round-1-billion-valuation-web3-nft-domain/]

### Outcome

Unstoppable Domains users collectively saved $100 million in gas fees on Polygon relative to what those operations would have cost on Ethereum L1. [Source: https://polygon.technology/blog/unstoppable-domains-users-save-100-million-in-fees-on-polygon]

## Behaviours That Could Become Requirements

| Behaviour | Implication for RFP |
|-----------|---------------------|
| Phased migration | A protocol migrating from L1 to L2 must maintain backward compatibility so existing name holders are not disrupted |
| L2 bridge for existing names | Users who hold names on L1 must have a migration path to L2; this requires a bridge contract and a migration UX |
| Zero-gas minting | Minting and record updates on L2 should incur negligible gas cost for the end user |
| EVM compatibility | Choosing an EVM-compatible L2 minimises smart contract adaptation effort |
| Retained L1 resolution | Names that were not migrated from L1 must still resolve via L1 lookups; resolution libraries must query both L1 and L2 |
| Investor alignment | L2 choice may be influenced by investor relationships, not purely technical merit |

## Privacy Implications

- L2 transactions are cheaper, so users make more of them. On Polygon (public PoS chain), all L2 transactions are still publicly visible. The lower cost does not improve on-chain privacy.
- Users who bridge domains from L1 to L2 create a public on-chain trail linking their L1 and L2 wallet activity.
- Some L2 sequencers have elevated centralisation risk (single sequencer can censor transactions), which is a concern for censorship-resistant naming applications. Polygon PoS uses a validator set that is more decentralised than a single sequencer.

## Censorship Resistance Implications

- Polygon PoS is more centralised than Ethereum L1 in practice: a smaller validator set and Polygon Labs has outsized influence over the chain. A state-level actor targeting the naming protocol could apply pressure to Polygon Labs.
- Bridging from L2 to L1 provides an escape hatch if the L2 becomes compromised, but only if the user acts before the L2 censors the bridge transaction.
- The migration pattern does not intrinsically weaken or strengthen censorship resistance; the relevant factor is the security and decentralisation of the chosen L2.

## Risks and Limitations

### L2 Sequencer Centralisation

Many L2 networks use a single sequencer (or a small set) that can reorder, delay, or censor transactions. For a naming protocol, this means the L2 operator can theoretically prevent record updates or domain transfers.

### Bridge Risk

Bridging assets between L1 and L2 introduces smart contract risk. Bridge exploits have resulted in significant losses in other protocols. A naming protocol that relies on bridging for migration must audit bridge contracts thoroughly.

### Fragmented Liquidity and Resolution

After migration, some names exist on L1 and some on L2. Resolution libraries must know which chain to query for which name. If the library is not updated promptly, resolution failures occur. Unstoppable Domains addressed this with a unified Resolution library that queries both chains. [Source: https://github.com/unstoppabledomains/resolution]

### Long-Term L2 Viability

Choosing a specific L2 (e.g., Polygon PoS) creates a long-term dependency on that network's continued operation and community support. If the L2 becomes deprecated (as Polygon PoS is moving toward Polygon AggLayer architectures), the naming protocol must migrate again.

## Sources

- https://unstoppabledomains.com/blog/polygon-l2-solution (Unstoppable + Polygon: Hello L2, Goodbye Gas Fees!, Unstoppable Domains Blog)
- https://unstoppabledomains.com/blog/domain-minting-live-on-polygon (We're LIVE on Polygon! You Can Now Mint Your Domains for Free!, Unstoppable Domains Blog)
- https://mobileapp.unstoppabledomains.com/learn-about/polygon-layer2/ (What is Polygon Layer2 Scaling Solution?, Unstoppable Domains Mobile App)
- https://polygon.technology/blog/unstoppable-domains-users-save-100-million-in-fees-on-polygon (Unstoppable Domains Users Save $100 Million in Fees on Polygon, Polygon Technology Blog)
- https://docs.unstoppabledomains.com/manage-domains/polygon-overview/ (Polygon L2 Network Overview, Unstoppable Domains Developer Portal)
- https://decrypt.co/86157/unstoppable-domains-taps-polygon-to-cut-gas-fees-for-ethereum-nft-domains (Unstoppable Domains Taps Polygon to Cut Gas Fees, Decrypt)
- https://docs.unstoppabledomains.com/smart-contracts/overview/uns-architecture-overview (UNS Architecture Overview, Unstoppable Domains Developer Portal)
- https://github.com/unstoppabledomains/resolution (Unstoppable Domains Resolution Library, GitHub)

## Tags

#naming #l2 #polygon #gas-fees #migration #evm #bridge #scaling #pattern
