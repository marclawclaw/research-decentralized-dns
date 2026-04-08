---
tags: [pattern, naming, zooko, cross-cutting]
related_projects: [ens, gns, ipns, handshake, namecoin, unstoppable-domains, space-id]
---

# Pattern: Zooko's Triangle Approaches Across Systems

## Summary

Zooko's triangle states that a naming system cannot simultaneously achieve all three of: global uniqueness, security, and human-readability. Each system studied makes a different architectural trade-off. This note catalogues those choices and their consequences.

## Approaches by system

| System | Global | Secure | Human-readable | Trade-off chosen |
|--------|--------|--------|----------------|-----------------|
| DNS | Yes | No (without DNSSEC/CA) | Yes | Sacrifices security; relies on trusted registrars and CAs |
| [[projects/ens]] | Yes | Yes (on-chain) | Yes | Requires Ethereum as trust anchor; token-based economics to manage namespace |
| [[projects/handshake]] | Yes (TLDs) | Yes (PoW) | Yes | Requires dedicated PoW blockchain; 15-day auction latency |
| [[projects/namecoin]] | Yes (.bit) | Yes (merged mining with Bitcoin) | Yes | Squatting dominated 99.985% of namespace due to insufficient economic friction |
| [[projects/unstoppable-domains]] | Yes | Yes (on-chain) | Yes | Centralised minting chokepoint; company controls issuance |
| [[projects/space-id]] | Yes (per-chain) | Yes (on-chain per chain) | Yes | Yoda oracle is undocumented centralisation risk for cross-chain uniqueness |
| [[projects/gns]] (petnames) | **No** | Yes | Yes | Sacrifices global uniqueness; names are locally meaningful only |
| [[projects/gns]] (zTLDs) | Yes | Yes | **No** | Sacrifices human-readability; base32 public key encoding |
| [[projects/ipns]] | Yes | Yes | **No** | Sacrifices human-readability; CIDv1 key hash; composes with ENS/DNSLink for human names |

## Key insight

The blockchain-based systems (ENS, Handshake, Namecoin, SPACE ID, Unstoppable Domains) all attempt to achieve all three corners by introducing an economic trust anchor (token, auction, fee). The cost of this approach is chain dependency: the security guarantee is only as strong as the underlying blockchain's consensus.

The DHT-based systems (GNS, IPNS) make an honest architectural choice: they sacrifice one corner explicitly (GNS: global uniqueness; IPNS: human-readability) rather than obscuring the trade-off behind economics.

For a Logos naming system, the recommended approach is to follow GNS's model at the base layer (self-certifying, privacy-preserving, chain-agnostic) and layer human-readable discovery as an optional, pluggable registrar zone. This keeps the architectural honesty of the DHT approach while providing the usability that adoption requires.

## Sources

- [Zooko's Triangle Wikipedia](https://en.wikipedia.org/wiki/Zooko%27s_triangle)
- [RFC 9498 Section 3 (GNS and Zooko's Triangle)](https://www.rfc-editor.org/rfc/rfc9498.html)
- [[patterns/gns-petname-system]]
- [[patterns/self-certifying-names]]
