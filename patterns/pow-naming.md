---
tags: [pattern, pow, blockchain, naming, utxo, handshake, namecoin]
pattern: Proof-of-Work Blockchain as Naming Root of Trust
seen-in: [[handshake]], [[namecoin]]
---

# Pattern: Proof-of-Work Blockchain as Naming Root of Trust

A naming system where a dedicated PoW blockchain serves as the canonical source of name ownership records. The blockchain's cumulative proof-of-work provides the security guarantee; no trusted authority is required to validate names.

---

## Where It Appears

- [[handshake]] (primary production example; TLD-level; custom BLAKE2b+SHA3 PoW)
- [[namecoin]] (original example; SLD .bit; merged-mined with Bitcoin)

---

## Core Properties

| Property | Detail |
|----------|--------|
| Permissionless registration | Any participant with HNS coins can register a name; no approval required |
| Censorship resistance | Revoking a name requires sustained majority hash-rate attack; no administrative path |
| Immutable history | Name state transitions are permanently recorded; auditable |
| Self-certifying ownership | Name ownership is proven by possession of the corresponding private key |
| Sybil resistance | PoW and auction cost prevent low-cost spam registration |
| No central operator | No single entity can unilaterally modify the name registry |

---

## Handshake Consensus Parameters

| Parameter | Value | Source |
|-----------|-------|--------|
| PoW algorithm | BLAKE2b + SHA3 | [hsd-dev protocol summary](https://hsd-dev.org/protocol/summary.html) |
| Block time | 10 minutes | [hsd-dev protocol summary](https://hsd-dev.org/protocol/summary.html) |
| Difficulty adjustment | Every block (targets 144 blocks/day) | [hsd-dev protocol summary](https://hsd-dev.org/protocol/summary.html) |
| Block size | 1,000,000 bytes base / 4,000,000 weight | [hsd-dev protocol summary](https://hsd-dev.org/protocol/summary.html) |
| Initial block reward | 2,000 HNS | [hsd-dev protocol summary](https://hsd-dev.org/protocol/summary.html) |
| Halving interval | 170,000 blocks (~3.25 years) | [ViaBTC](https://www.viabtc.com/en/blog/Beginner-what-is-handshake-hns-how-to-mine-handshake-hns-458) |
| Block withholding defence | maskHash rerandomisation in block header | [hsd-dev protocol summary](https://hsd-dev.org/protocol/summary.html) |
| Merge mining | No (dedicated chain; no merged mining) | [GeeksforGeeks HNS](https://www.geeksforgeeks.org/blogs/handshake-an-peer-to-peer-naming-system/) |

---

## Comparison: Handshake vs. Namecoin PoW

| Aspect | Handshake | Namecoin |
|--------|-----------|---------|
| Launch | 2020 | 2011 |
| Chain | Dedicated | Bitcoin fork |
| Merge mining | No | Yes (with Bitcoin) |
| PoW algorithm | BLAKE2b + SHA3 | SHA256d (Bitcoin) |
| Namespace scope | TLDs only | SLDs (.bit) |
| Name expiry | 2-year renewal (miner fee) | ~200 days renewal |
| State storage | Covenant UTXO | Key-value NVS |
| Adoption | 11.3M domain registrations (Jul 2024) | [NOT FOUND: current domain count] |

- Source: [Namecoin.org](https://www.namecoin.org/)
- Source: [hsd-dev protocol summary](https://hsd-dev.org/protocol/summary.html)

---

## Security Model

The security of name records depends on the cumulative PoW of the chain. An attacker wishing to reorg the chain and alter name records must accumulate more than 50% of the network's hash rate and sustain that majority for the duration of the attack. For Handshake (relatively small network, ~$3.3M market cap as of Apr 2026), this 51% attack cost is significantly lower than Bitcoin. This is the primary security weakness of PoW naming systems with small hash rates.

Unlike DNSSEC, PoW naming does not have a single administrative key that, if compromised, undermines all names. The attack vector is economic rather than cryptographic.

An additional security consideration unique to PoW naming: key loss is permanent. Traditional DNSSEC allows a registrar to re-issue credentials. In Handshake, the private key IS the credential; if lost, the name is permanently unrecoverable.

- Source: [Blockchain-names.com: Handshake security](http://www.blockchain-names.com/handshake.html)

---

## Light Client (SPV) Feasibility

PoW blockchains naturally support Simple Payment Verification (SPV) light clients that:
- Download only block headers (not full chain state)
- Request Merkle proofs for specific name lookups
- Verify proofs against the PoW header chain

For Handshake, hnsd and hnsquery implement this pattern. SPV enables lightweight resolution without running a full node, enabling browser extensions, mobile apps, and embedded resolvers.

Privacy note: SPV clients leak queried TLD names to the full nodes they connect to. The hnsquery library partially mitigates this by sending a hash of the name rather than the plaintext, but the full node can still observe resolution patterns.

- Source: [hnsd GitHub](https://github.com/handshake-org/hnsd)
- Source: [hnsquery GitHub](https://github.com/imperviousinc/hnsquery)

---

## RFP Implications

- PoW naming provides the strongest censorship resistance of any naming approach (requires majority hash-rate attack, not just a legal order)
- Trade-off: PoW chains are energy-intensive, have name acquisition latency (auction cycles), and require HNS for participation
- For a Logos/Waku naming system, the PoW model is architecturally analogous to using an existing L1 (Bitcoin or Ethereum) as the root of trust, but without the overhead of a dedicated chain
- SPV light client compatibility is a direct requirement for practical deployment in constrained environments
- The 51% attack risk at low market cap is a design concern for any new PoW naming chain; piggybacking on an existing high-hash-rate chain (merged mining or inheriting security) would mitigate this

---

## Sources

- [hsd-dev protocol summary](https://hsd-dev.org/protocol/summary.html)
- [Handshake white paper](https://handshake.org/files/handshake.txt)
- [ViaBTC: HNS mining guide](https://www.viabtc.com/en/blog/Beginner-what-is-handshake-hns-how-to-mine-handshake-hns-458)
- [hnsd GitHub](https://github.com/handshake-org/hnsd)
- [hnsquery GitHub](https://github.com/imperviousinc/hnsquery)
- [Namecoin.org](https://www.namecoin.org/)
- [GeeksforGeeks HNS](https://www.geeksforgeeks.org/blogs/handshake-an-peer-to-peer-naming-system/)
