# Pattern: Cross-Chain Name Resolution

## Summary

A name registered on one blockchain (or in a DNS zone) resolves to addresses and records on multiple chains, without duplicating the authoritative record. The user sees a single name; the resolver handles chain-specific lookups.

## Observed In

- [[projects/d3-doma]] (Doma Protocol + ENS integration) — DNS domain tokenised on Doma Chain resolves as a native ENS name on Ethereum and resolves addresses on Base, Solana, Bitcoin, etc.
- ENS multichain (EIP-2304, CCIP-Read) — .eth name can store addresses for multiple chains

## How It Works (Doma + ENS)

### Integration Mechanism

1. D3 updated the Doma registry smart contract to emit ENS-compatible records (standardised on-chain name records).
2. ENS whitelisted the Doma registry contract, allowing the ENS protocol to treat tokenised DNS domains as native .eth-equivalent names.
3. The ENS record is programmatically linked to the DOT NFT: when the DOT is transferred, the ENS profile (linked wallet address, avatar, text records) updates automatically.
4. No DNSSEC setup is required; the Doma registry acts as the authoritative source for ENS resolution.

[Source: https://ens.domains/blog/post/d3-doma, October 2025]

### Resolution Path for a Tokenised Domain (e.g., software.ai)

```
User queries "software.ai" in an ENS-aware wallet
  → ENS Universal Resolver checks if "software.ai" has a registered resolver
  → Finds Doma registry (whitelisted)
  → Doma registry returns the address records stored against the DOT owner
  → Records can reference Ethereum, Base, Solana, Bitcoin addresses simultaneously
  → No DNS lookup required for on-chain resolution
```

For traditional DNS resolution (browser navigating to software.ai):
```
Browser → standard DNS → authoritative nameserver (managed by registrar)
  → A record returns web server IP
  → (DNS resolution is unchanged; the blockchain layer is additive)
```

### CCIP-Read (EIP-3668) for Offchain/L2 Resolution

ENS supports CCIP-Read: a smart contract can throw a standard error directing the resolver to fetch data from an L2 or off-chain endpoint, verify it, and return it to the caller. This allows an L2 like Doma Chain to serve as the authoritative record store for names that resolve in the L1 ENS system, without requiring the L1 contract to hold all records.

[Source: https://docs.ens.domains/resolvers/ccip-read/]

## Key Behaviours

| Behaviour | Implication for RFP |
|-----------|---------------------|
| Single name resolves on multiple chains | A name registered once can point to addresses on any supported chain without re-registration |
| NFT transfer triggers automatic resolver update | Name ownership transfer is atomic with record update; no manual re-configuration needed |
| ENS whitelisting grants cross-ecosystem compatibility | A new naming chain can gain ENS ecosystem reach (wallets, dApps, L2s) without rebuilding resolver infrastructure |
| Traditional DNS and on-chain resolution coexist | The same name works in browsers (DNS) and wallets (ENS), with no conflict or downtime |
| CCIP-Read enables L2-native record stores | Name records can live on a purpose-built L2 for cost/throughput while resolving through L1 infrastructure |

## Contrast With Single-Chain Resolution

| Dimension | Cross-Chain Model | Single-Chain Model (e.g., SNS) |
|-----------|------------------|-------------------------------|
| Address coverage | Multiple chains from one name | One chain only |
| Record update on transfer | Automatic (NFT-linked) | Manual or separate tx required |
| Infrastructure dependency | Reliance on L1 (ENS) whitelisting | Self-contained |
| Censorship at resolver layer | ENS DAO can revoke whitelist | No external dependency |

## Limitations

- Cross-chain resolution via ENS depends on ENS governance: ENS DAO could revoke the Doma registry whitelist, breaking resolution for all tokenised domains.
- LayerZero messaging (used for bridging DOTs cross-chain) is a separate security assumption from ENS resolution; a compromise of LayerZero does not break resolution but could affect ownership.
- Traditional DNS resolution remains on centralised infrastructure; the cross-chain naming layer does not improve DNS privacy or censorship resistance for browser navigation.

## Sources

- https://ens.domains/blog/post/d3-doma (ENS Blog, October 2025)
- https://docs.ens.domains/resolvers/ccip-read/ (ENS CCIP-Read Docs, 2025)
- https://docs.doma.xyz/readme/protocol-overview (Doma Protocol Documentation, 2025)
- https://messari.io/report/doma-domains-galore (Messari: Doma Domains Galore, 2025)

## Tags

#naming #cross-chain #ens #resolution #ccip-read #multi-chain #pattern
