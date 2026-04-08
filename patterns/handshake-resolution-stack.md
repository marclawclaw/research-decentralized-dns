---
tags: [pattern, resolution, spv, light-client, dns, handshake]
pattern: Handshake Resolution Stack
seen-in: [[handshake]]
---

# Pattern: Handshake Resolution Stack

The layered architecture by which a Handshake name query is resolved, from user application to blockchain state. Multiple resolution paths exist with different trade-offs between trust, privacy, and setup complexity.

---

## Resolution Paths

### Path 1: Full Node (hsd) — Most Trustless

The user runs a complete Handshake full node (hsd). The node:
1. Replicates the entire Handshake blockchain
2. Maintains the complete root zone state (all TLD records)
3. Acts as both an authoritative root server and a recursive resolver
4. Resolves HNS TLDs from local state; falls through to traditional DNS for ICANN TLDs

Privacy: no external party sees queries (all resolution is local).
Security: full node validates every block; no trust assumption beyond PoW.
Cost: ~tens of GB of chain data; not suitable for mobile or embedded use.

- Source: [hsd GitHub](https://github.com/handshake-org/hsd)
- Source: [Bob Wallet](https://bobwallet.io/)

### Path 2: SPV Light Client (hnsd) — Trustless, Lightweight

hnsd is a Handshake SPV resolver written in C. It:
1. Downloads only block headers (not full block data)
2. Connects to full nodes to request Merkle proofs for specific TLD lookups
3. Verifies proofs against the downloaded header chain
4. Translates HNS name data into standard DNS responses (UDP)
5. Acts as an authoritative root server for other DNS software (e.g., libunbound)

Architecture:
```
Stub resolver (OS/browser)
  -> libunbound (recursive resolver)
    -> hnsd (SPV authoritative root server)
      -> full node peers (P2P proof requests)
        -> Handshake blockchain state
```

Privacy limitation: the full nodes hnsd connects to learn which TLDs are being queried.
Mitigation: hnsquery (a library version) sends a hash of the name rather than plaintext.

- Source: [hnsd GitHub](https://github.com/handshake-org/hnsd)
- Source: [BlockChannel: HNS bridge resolver guide](https://medium.com/blockchannel/hns-bridge-dns-resolver-setup-guide-bb3b5f3c5d44)

### Path 3: hnsquery Library — Embeddable SPV

hnsquery is a cross-platform Go/C library wrapping the SPV resolution logic. Intended for embedding in apps (browsers, mobile apps, desktop software). Key difference from hnsd: queries to full nodes use the hash of the name, partially preserving privacy.

- Source: [hnsquery GitHub](https://github.com/imperviousinc/hnsquery)

### Path 4: Third-Party DNS Gateway — Convenient, Trust-Dependent

Services such as hns.to, hns.is, and rsvr.xyz act as DNS proxies. Users configure their OS or browser to use these resolvers. The gateway performs all HNS resolution on behalf of the user.

Privacy: the gateway operator sees all queries.
Security: the user trusts the gateway operator to return honest results.
Convenience: no software to install; works with any existing DNS configuration.

Examples:
- hns.to: proxy gateway; converts HNS URLs to traditional HTTPS accessible via a browser extension
- hns.is: DNS resolver (configurable as a system DNS server)
- rsvr.xyz: DNS resolver

- Source: [Namebase: Access Handshake names](https://learn.namebase.io/starting-from-zero/how-to-access-handshake-sites)
- Source: [Handypedia: Resolving Handshake Domains](https://en.handypedia.org/wiki/Resolving_Handshake_Domains)

### Path 5: PoWDoH (PoW over DNS-over-HTTPS) — Hybrid

PoWDoH is a technique that fetches the DNSSEC chain from a DoH server and verifies it against a PoW SPV proof. Combines the convenience of DoH with the security of SPV verification. Currently experimental.

- Source: [hnsquery README](https://github.com/imperviousinc/hnsquery/blob/main/README.md)

---

## SLD Resolution (Second-Level Domains)

The Handshake blockchain stores only TLD records (NS and/or DS records). Resolution of SLDs beneath a Handshake TLD proceeds through normal DNS, using the authoritative name server identified by the TLD's NS record.

```
Query: example.mynewTLD
  1. Resolver queries HNS blockchain -> finds NS record for mynewTLD -> ns1.mynewTLD
  2. Resolver queries ns1.mynewTLD -> finds A record for example.mynewTLD
```

This means the TLD owner must operate their own DNS infrastructure to support SLDs. The TLD owner can use any standard DNS hosting service that supports DNSSEC.

---

## Privacy Summary

| Path | Who sees queries | Verification |
|------|-----------------|-------------|
| Full node (hsd) | Nobody (local) | Full blockchain validation |
| SPV (hnsd) | Connected full nodes (TLD name in plaintext) | SPV Merkle proofs against header chain |
| hnsquery | Connected full nodes (TLD name as hash) | SPV Merkle proofs against header chain |
| DNS gateway | Gateway operator (all queries) | None (trust-based) |

---

## RFP Implications

- Any decentralised naming system for Logos/Waku should consider whether an SPV light client path exists
- The hsd/hnsd architecture demonstrates how a blockchain naming system can be split into full-node and light-client tiers
- Privacy at resolution time is structurally weak in all PoW naming systems (SPV must contact full nodes); a DHT-based system (e.g., GNS) has better query privacy
- For embedded use in Logos applications, the hnsquery library pattern (embeddable SPV with hashed queries) is the closest prior art for a lightweight trustless resolver component
- Third-party gateway equivalents in a Logos naming system should be clearly labelled as trust-dependent to avoid misleading users about the system's decentralisation properties

---

## Sources

- [hnsd GitHub](https://github.com/handshake-org/hnsd)
- [hsd GitHub](https://github.com/handshake-org/hsd)
- [hnsquery GitHub](https://github.com/imperviousinc/hnsquery)
- [Bob Wallet](https://bobwallet.io/)
- [Namebase: Access Handshake names](https://learn.namebase.io/starting-from-zero/how-to-access-handshake-sites)
- [Handypedia: Resolving Handshake Domains](https://en.handypedia.org/wiki/Resolving_Handshake_Domains)
- [BlockChannel: HNS Bridge DNS Resolver guide](https://medium.com/blockchannel/hns-bridge-dns-resolver-setup-guide-bb3b5f3c5d44)
