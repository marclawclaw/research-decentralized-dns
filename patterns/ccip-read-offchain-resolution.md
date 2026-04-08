---
tags: [pattern, naming, offchain, ethereum, l2]
related_projects: [ens, d3-doma, space-id]
eip: EIP-3668
ensip: ENSIP-10 (wildcard prerequisite)
---

# Pattern: CCIP-Read Offchain Resolution (EIP-3668)

## Summary

CCIP-Read (Cross-Chain Interoperability Protocol Read, EIP-3668) is an Ethereum standard that allows smart contracts to defer data lookups to off-chain HTTP gateways. For ENS, it is the mechanism enabling L2-backed and database-backed name records to be resolved without on-chain storage of every record. The resolution is transparent to end-users: applications resolve names the same way regardless of whether data is stored on-chain or off-chain.

## How it works

```
Client                   Resolver (on-chain)          Gateway (off-chain)
  |                              |                            |
  |-- resolve(name, calldata) -->|                            |
  |                              |-- revert OffchainLookup --|
  |<-- OffchainLookup error -----|  (urls[], callData,        |
  |    contains:                 |   callbackFunction,        |
  |    - gateway URL(s)          |   extraData)               |
  |    - callData for gateway    |                            |
  |    - callback selector       |                            |
  |    - extraData               |                            |
  |                              |                            |
  |-- HTTP POST/GET to gateway -------------------------------->|
  |<-- gateway returns response --------------------------------|
  |                              |                            |
  |-- callbackFunction(response, extraData) -->|              |
  |    (on-chain verification step)            |              |
  |<-- verified result ------------------------|              |
```

Steps:
1. Client calls the resolver with the name and query data.
2. Resolver reverts with `OffchainLookup(address sender, string[] urls, bytes callData, bytes4 callbackFunction, bytes extraData)`.
3. Client sends the callData to each gateway URL in sequence until one responds.
4. Client calls the callback function on the resolver with the gateway response and extraData.
5. The resolver verifies the response (e.g. checking a signature or Merkle proof) and returns the final value.

## Centralisation trade-offs

CCIP-Read introduces a **gateway trust dependency**:

- The gateway URL is stored in the resolver contract and is changeable by the resolver owner.
- If the gateway is offline, the name cannot be resolved.
- The gateway can selectively refuse to answer certain queries (censorship).
- The gateway operator sees every lookup request, compromising query privacy.

Mitigations:
- Multiple gateway URLs in the `urls` array provide fallback redundancy.
- Cryptographic verification in the callback can prove the response is authentic (signed by a known key or proven against an on-chain Merkle root), but cannot force liveness.
- The ENS community has discussed requiring multiple independent gateways for critical names.

## ENS-specific applications

- **L2 names**: resolvers on Optimism, Base, Arbitrum, etc. publish records; the CCIP-Read gateway bridges resolution to the target chain.
- **Subname services**: companies (e.g. Coinbase, Gemini) issue thousands of subnames (`user.coinbase.eth`, `you.gemini.eth`) stored in a centralised database, served via a CCIP-Read gateway. No per-subname on-chain transaction needed.
- **Gasless DNS import (ENSIP-17)**: DNS domain owners set a TXT record with a DNSSEC proof; the OffchainDNSResolver fetches and verifies the proof at resolution time, requiring zero on-chain gas.

## Wildcard resolution interplay (ENSIP-10)

CCIP-Read is most powerful when combined with wildcard resolution (ENSIP-10). A single on-chain resolver entry for `*.company.eth` can serve any subname dynamically by delegating to the gateway. Together they enable:

- Dynamic NFT-based subnames: `<tokenId>.nifty.eth` resolves to the current NFT owner.
- User-generated subnames at scale without individual on-chain registrations.

## Privacy implications

The gateway sees every query. An entity monitoring the gateway can build a graph of:
- Which wallet addresses look up which names
- Lookup frequency and timing

This is the same privacy problem as traditional DNS resolvers, plus the gateway URL is publicly visible on-chain (trivially monitored). No production ENS-compatible gateway offers query privacy as of April 2026; a grant proposal for verifiable privacy-preserving resolution was submitted in 2024 and remains in discussion. See [[projects/ens#Limitations and criticisms]].

## RFP relevance

CCIP-Read is the current industry standard for off-chain naming data that needs on-chain verification. Key implications for a decentralised naming RFP:

1. **Liveness vs trustlessness**: the pattern makes liveness dependent on gateway availability. A DHT-based alternative could improve liveness guarantees.
2. **Privacy gap**: any CCIP-Read gateway is a surveillance point. A DHT-based resolution removes the single resolver oracle.
3. **Verifiability vs censorship resistance**: the callback verification step is strong but does not prevent gateway censorship (refusing to serve responses).
4. **Standardisation value**: EIP-3668 is widely adopted; any new naming system should consider interoperability with existing CCIP-Read resolvers.

## Sources

- [EIP-3668: CCIP Read](https://eips.ethereum.org/EIPS/eip-3668) — accessed 2026-04-08
- [ENS Docs: Offchain/L2 Resolvers](https://docs.ens.domains/resolvers/ccip-read/) — accessed 2026-04-08
- [ENS Docs: Layer 2 and Offchain Resolution](https://docs.ens.domains/learn/ccip-read/) — accessed 2026-04-08
- [ENS DAO Forum: Privacy Grant Proposal](https://discuss.ens.domains/t/grant-proposal-verifiable-and-privacy-preserving-ens-resolution/21108) — accessed 2026-04-08
- [ENSIP-10: Wildcard Resolution](https://docs.ens.domains/ensip/10/) — accessed 2026-04-08
- [ENSIP-17: Gasless DNS Resolution](https://docs.ens.domains/ensip/17/) — accessed 2026-04-08
