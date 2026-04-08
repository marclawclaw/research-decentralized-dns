---
tags: [pattern, naming, scalability, ethereum]
related_projects: [ens]
ensip: ENSIP-10
---

# Pattern: ENS Wildcard Resolution (ENSIP-10)

## Summary

Wildcard resolution (ENSIP-10) allows a single on-chain resolver registration to dynamically serve resolution for any subdomain of a parent name. Without wildcards, each subname (`alice.company.eth`, `bob.company.eth`, etc.) would require a separate on-chain transaction to register. With wildcards, a resolver set at `*.company.eth` handles all subnames via a single contract entry, using CCIP-Read to fetch actual records off-chain. This enables scalable subname issuance (enterprise SSO, NFT ownership, on-chain identity) without per-subname gas costs.

## How it works

Standard ENS resolution:
1. Compute node for `alice.company.eth`.
2. Query Registry for that node's resolver.
3. If resolver exists, query resolver for the record.

With ENSIP-10 (wildcard):
1. Compute node for `alice.company.eth`.
2. Query Registry. If **no resolver is set** for this exact node:
   a. Strip the leftmost label (`alice`), compute node for `company.eth`.
   b. Query Registry for `company.eth`'s resolver.
   c. If that resolver implements the `resolve(bytes name, bytes data)` function (defined in ENSIP-10), call it with the full original name and the record calldata.
3. The resolver's `resolve()` function handles the wildcard lookup (typically via CCIP-Read to a gateway).

The ENSIP-10 `resolve()` function receives the DNS-encoded name (not just the node), allowing the resolver to dynamically dispatch based on the label (e.g. look up `alice` in a database and return her address).

## Backwards compatibility

- Existing ENS Registry and resolvers are unmodified.
- Legacy clients that do not support ENSIP-10 fail gracefully (they just do not find a resolver for unregistered subnames).
- ENSIP-10 requires client-side support to strip labels and retry the lookup.

## Use cases in production

- **Enterprise subnames**: `user.coinbase.eth` for all Coinbase users; `you.gemini.eth` for Gemini wallet customers. Single on-chain resolver; millions of subnames stored in a database, served via CCIP-Read gateway.
- **NFT-linked names**: `<tokenId>.collection.eth` resolves to the current holder's address. The resolver queries an NFT contract via CCIP-Read.
- **Protocol-assigned names**: DeFi protocols assign `user.protocol.eth` names to all depositors, stored off-chain.

## Combination with CCIP-Read

Wildcard resolution alone determines *which* resolver to call. CCIP-Read ([[patterns/ccip-read-offchain-resolution]]) determines *how* the resolver fetches data. The two patterns are designed to work together:

```
Client resolution of alice.company.eth:
  1. ENSIP-10: no exact resolver found -> try parent company.eth
  2. company.eth resolver found; implements resolve()
  3. resolve() -> reverts with OffchainLookup (EIP-3668)
  4. Client hits gateway: GET /resolve?name=alice.company.eth&calldata=...
  5. Gateway returns alice's ETH address + signature
  6. Callback verifies signature -> returns address
```

## Limitations

- **Centralised database dependency**: in most production deployments, the wildcard resolver ultimately reads from a centralised server. The decentralisation guarantee is only as strong as the gateway backend.
- **Client requirement**: clients must implement ENSIP-10 label-stripping logic. Older clients silently fail on wildcard names.
- **No on-chain record for subnames**: wildcard subnames do not appear in on-chain indexes; tooling that scans the Registry for names cannot discover wildcard-issued subnames.

## RFP relevance

Wildcard resolution is a scalability pattern that any naming protocol must address when supporting sub-namespaces:

1. **Scalable subname issuance**: without wildcard equivalents, each subname requires an on-chain transaction and fee. This is prohibitive at scale.
2. **Centralisation trade-off**: wildcard + CCIP-Read offloads cost but re-introduces centralisation. A DHT-based system could serve wildcard lookups in a decentralised way.
3. **Discovery gap**: wildcard names are not discoverable via on-chain enumeration. Any naming system should consider whether name discoverability is a requirement.

## Sources

- [ENSIP-10: Wildcard Resolution](https://docs.ens.domains/ensip/10/) — accessed 2026-04-08
- [ENS Docs: Subdomains](https://docs.ens.domains/web/subdomains/) — accessed 2026-04-08
- [MyCrypto Blog: ENS and Layer 2](https://blog.mycrypto.com/ens-and-layer-2/) — accessed 2026-04-08
