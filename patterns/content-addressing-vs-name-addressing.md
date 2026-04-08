# Pattern: Content Addressing vs Name Addressing

## Summary

Content addressing and name addressing are two complementary approaches to locating data in a network. Content addressing identifies data by its hash (what it is); name addressing identifies data by a mutable pointer (where it is, currently). The distinction is fundamental to understanding how [[projects/ipns]] fits into the IPFS ecosystem and how decentralised naming systems bridge the gap between immutable cryptographic identifiers and the mutable, human-meaningful identifiers that users expect.

## Core Concepts

### Content Addressing (IPFS CIDs)

In IPFS, every piece of data is identified by a Content Identifier (CID): a cryptographic hash of the data itself. The CID is immutable: changing the content changes the hash and therefore the address.

```
Data → SHA2-256 (or SHA2-512, BLAKE3, etc.) → Multihash → CIDv1
e.g. bafybeig6xv5nwphfmvcnektpnojts33jqcuam7bmye2pb54adnrtccjlsu
```

Properties:
- **Verifiable:** anyone with the CID can confirm the data has not been tampered with.
- **Deduplication:** identical data has the same CID everywhere in the network.
- **Immutable:** you cannot update content at a CID; you must publish a new CID.
- **Location-independent:** data can be fetched from any node that has it.

### Name Addressing (IPNS)

IPNS introduces mutable pointers layered on top of CIDs. An IPNS name is a stable cryptographic identifier (a public key hash) that can be redirected to any CID by publishing a new signed record.

```
IPNS Name (stable) → [signed record] → CIDv1 (current content)
k51qz... → /ipfs/bafybeig6... (published today)
k51qz... → /ipfs/Qmab12... (published tomorrow after content update)
```

The mental model: IPFS CID = immutable pointer to content; IPNS name = mutable pointer to a mutable CID target. [Source: https://docs.ipfs.tech/concepts/ipns/]

## Why Both Are Needed

| Scenario | CID Alone | IPNS Required |
|----------|-----------|---------------|
| Archiving a document snapshot | Sufficient (the hash is the proof) | Not needed |
| Publishing a website that updates weekly | Sharing a new CID each update is impractical | IPNS provides a stable address |
| NFT metadata pointing to unchanging art | Sufficient | Not needed |
| Software package registry (latest version) | Not sufficient | IPNS or equivalent needed |
| DNS-equivalent: name resolves to current server | Not sufficient | IPNS or blockchain equivalent needed |

## How Name Addressing Layers on Content Addressing

```
Human-readable layer       (DNS / ENS / DNSLink)
         ↓
Name addressing layer      (IPNS: mutable signed pointer)
         ↓
Content addressing layer   (IPFS CID: immutable content hash)
         ↓
Transport layer            (Bitswap / HTTP / libp2p streams)
```

Each layer is independently verifiable and independently cacheable:
- The DNS-to-IPNS mapping is verified by DNS DNSSEC (or bypassed with trust in the resolver).
- The IPNS record is verified by Ed25519 signature over the public key embedded in the name.
- The CID content is verified by hashing the retrieved bytes.

## Trade-offs

| Property | Content Addressing | Name Addressing (IPNS) |
|----------|--------------------|------------------------|
| Verifiability | Perfect (hash of data) | Good (signature by key holder) |
| Mutability | None | Full (publisher can redirect at will) |
| Caching | Indefinite (content never changes) | Bounded by TTL (5 minutes default since Kubo v0.34.0, 48 hours max) |
| Namespace | Cryptographic (unguessable) | Cryptographic (human names need a bridge) |
| Resolution latency | Fast (content can be cached anywhere) | Slow if via DHT (~11s median) |
| Availability after publisher offline | Permanent (if pinned) | Degrades after 48h |
| Privacy | CID reveals content (hash leakage) | Name is a key fingerprint (still linkable) |

## Implications for Decentralised DNS

A decentralised DNS replacement must address both layers:
1. **Name addressing:** a stable, updateable identifier that maps to the current record set (analogous to a DNS zone).
2. **Content addressing (optional but desirable):** zone records can themselves be content-addressed for verifiability and deduplication, with the name pointing to the latest record set CID.

This is analogous to how IPNS is used in practice: the IPNS name is the stable "domain", and the IPFS CID it points to is the versioned snapshot of the content. For DNS records, the CID could be a DAG-CBOR encoded zone file, enabling zone history, efficient diffing, and cryptographic auditability.

## Sources

- https://docs.ipfs.tech/concepts/ipns/ (IPFS Docs: IPNS, 2025)
- https://docs.ipfs.tech/concepts/content-addressing/ (IPFS Docs: Content Identifiers, 2025)
- https://docs.ipfs.tech/concepts/immutability/ (IPFS Docs: Immutability, 2025)
- https://eth-limo.gitbook.io/documentation/beginner/configuring-your-ens-name/content-hash-overview/understanding-content-hashes-ipns-and-ipfs-for-ens (eth.limo: IPNS and ENS content hashes, 2025)
- https://filebase.com/blog/ipfs-content-addressing-explained/ (Filebase: Content Addressing Explained, 2024)
- https://github.com/ipfs/kubo/releases/tag/v0.34.0 (Kubo v0.34.0 release notes, March 2025)

## Tags

#content-addressing #name-addressing #cid #ipns #ipfs #mutability #naming #pattern
