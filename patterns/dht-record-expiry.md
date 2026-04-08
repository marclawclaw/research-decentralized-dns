# Pattern: DHT Record Expiry and Republishing

## Summary

Distributed Hash Table (DHT) implementations typically store records ephemerally rather than persistently. Records have a maximum time-to-live (TTL) enforced by storing peers, after which they are garbage collected. Publishers must therefore re-announce their records periodically to keep them alive in the network. This pattern creates a trade-off between network resource consumption (storage and bandwidth for republishing) and record availability.

This pattern is central to [[projects/ipns]], which relies on the Amino DHT for global record distribution.

## Observed In

- [[projects/ipns]] (IPNS over Amino DHT): 48-hour maximum record lifetime; Kubo republishes every 4 hours.
- IPFS content routing (provider records): records expire after ~24-48 hours; providers must republish.
- Kademlia DHT generally: most Kademlia implementations expire records to prevent unbounded storage growth on storing nodes.

## How It Works (IPNS on Amino DHT)

```
Publisher                DHT Network              Resolver
   |                         |                       |
   |-- publish(record) ----->|                       |
   |   (stored by k=20       |                       |
   |    closest peers)       |                       |
   |                         |                       |
   |   [4 hours pass]        |                       |
   |-- republish(record) --->|  (refreshes expiry)   |
   |                         |                       |
   |   [48 hours without     |                       |
   |    republish]           |-- garbage collect --  |
   |                         |   (record removed)    |
   |                         |                       |
   |                         |<----- resolve() ------|
   |                         |-- (no record found) ->|
   |                         |   (resolution fails)  |
```

## Configuration (Kubo Defaults, current as of v0.34.0)

| Parameter | Value | Notes |
|-----------|-------|-------|
| `--lifetime` (publish flag) | 48 hours | Validity timestamp embedded in the record |
| `--ttl` (cache hint flag) | 5 minutes | How long resolvers and gateways cache the result; lowered from 1 hour in Kubo v0.34.0 |
| Republish interval | Every 4 hours | Background daemon republishes all owned IPNS keys |
| DHT enforced max expiry | 48 hours | DHT storing peers discard records older than this regardless of `validity` field |

Sources: https://github.com/ipfs/kubo/releases/tag/v0.34.0, https://github.com/ipfs/kubo/blob/master/docs/config.md

## Key Behaviours

| Behaviour | Implication |
|-----------|-------------|
| Records expire after 48 hours | A publisher who goes offline causes their IPNS name to silently stop resolving |
| Republish every ~4 hours | Operational burden: the publisher node must remain online or delegate republishing |
| No persistent storage guarantee | DHT nodes are ephemeral; they can leave the network and take stored records with them |
| Record propagation on update | After republishing, updated records propagate as resolvers query and cache-invalidate |
| `validity` field vs DHT enforcement | The `validity` timestamp in the record can specify longer periods, but DHT nodes enforce their own 48-hour cap |
| Write-back on stale records | A resolver that finds an outdated record sends the latest version to the stale node, helping propagation |

## Resolution Semantics Under Expiry

Because multiple DHT nodes may store copies with different sequence numbers or ages, the resolver uses quorum-based resolution:
1. Collect responses from at least 16 of the 20 closest peers (quorum = 16).
2. Select the record with the highest valid sequence number.
3. Write the selected record back to any peer that returned an older version.

If all closest peers have discarded the record (post-expiry), resolution returns no result. This is distinct from a "name does not exist" response; there is no standardised error differentiating "record expired" from "name never existed."

## Contrast With Persistent Storage Models

| Dimension | DHT Expiry Model (IPNS) | Persistent Storage (DNS, ENS) |
|-----------|------------------------|-------------------------------|
| Storage duration | Ephemeral (max 48h without refresh) | Indefinite (until explicitly deleted) |
| Availability when publisher offline | Degrades after 48h | Maintained by authoritative server / chain |
| Operational burden | Continuous republishing required | Set-and-forget (except for renewals) |
| Network storage cost | Amortised across DHT peers | Borne by authoritative server or on-chain storage |
| Censorship resistance | High (no single storage point) | Lower (DNS: registrar can delete; ENS: higher resistance) |

## Relevance for Decentralised DNS RFP

A decentralised DNS system must decide whether name records are stored ephemerally or persistently:
- **Ephemeral (DHT-style):** lower storage burden per node, but publishers must stay online or use pinning services. Suitable for dynamic pointers that are frequently updated.
- **Persistent (chain-anchored or pinned):** name records survive publisher downtime. More appropriate for DNS-equivalent use cases where availability expectations are high (similar to traditional DNS uptime SLAs of 99.99%).
- A hybrid approach (IPNS over DHT with optional persistent pinning via a pinning service or on-chain anchor) is used in practice by services like Fleek and Filebase.

## Sources

- https://specs.ipfs.tech/ipns/ipns-record/ (IPFS Standards: IPNS Record spec, 2025)
- https://specs.ipfs.tech/routing/kad-dht/ (IPFS Standards: Kademlia DHT spec, 2025)
- https://docs.ipfs.tech/concepts/dht/ (IPFS Docs: Distributed Hash Tables, 2025)
- https://github.com/ipfs/kubo/releases/tag/v0.24.0 (Kubo v0.24.0 release notes, 2023)
- https://github.com/ipfs/kubo/releases/tag/v0.34.0 (Kubo v0.34.0 release notes, March 2025)
- https://github.com/ipfs/kubo/blob/master/docs/config.md (Kubo config documentation, 2025)
- https://www.probelab.network/blog/ipns-performance-amino-dht (ProbeLab: IPNS Performance, August 2025)

## Tags

#dht #naming #expiry #republishing #ipns #kademlia #availability #pattern
