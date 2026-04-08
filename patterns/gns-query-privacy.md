---
tags: [pattern, privacy, query-privacy, key-blinding, zone-enumeration, gns, gnunet]
pattern: Query privacy in naming systems via key blinding
seen-in: [gns]
related: [gns-dht-name-resolution]
---

# Pattern: Query Privacy in Naming (GNS Key Blinding)

Conventional DHT-based naming systems store records under predictable keys (e.g., `hash(domain_name)`) that allow any DHT participant to trivially enumerate stored names or correlate a query to a zone. GNS solves this through **key blinding**: records are stored under a derived key that is unlinkable to the zone identity without knowledge of both the zone key and the specific label being queried.

---

## Problem: query privacy in naming DHTs

In a naive DHT naming system:

- **Zone enumeration**: a DHT peer receiving a PUT can determine the zone from the key (if keys are `hash(zone_public_key)`), and by watching many PUTs can infer all labels in a zone.
- **Query correlation**: a DHT peer receiving a GET can determine which zone is being queried, and potentially which user is interested in that zone.
- **Traffic analysis**: by correlating PUT and GET traffic, an adversary can build a social graph of who resolves whom.

DNSSEC with NSEC records is subject to zone walking. DNSSEC with NSEC3 uses hashed labels but is still subject to offline dictionary attacks. GNS's approach is stronger because the DHT key is derived from both the zone key and the label jointly, making partial knowledge insufficient.

---

## GNS blinding scheme

### Key derivation

For a zone with public key `Z` and a label `l`:

```
h = H(Z, l)           # H is a hash-to-scalar function over the curve
Z_l = h * Z           # scalar multiplication on the elliptic curve (blinded key)
z_l = h * z           # corresponding blinded private key (zone owner only)
```

Properties:

- `Z_l` is computationally unlinkable to `Z` without knowledge of `l`.
- `Z_l` for label `l1` is computationally unlinkable to `Z_l2` for label `l2` without knowledge of both labels.
- The zone owner can compute `z_l` to sign records.
- A resolver who knows `Z` and `l` can derive `Z_l` to verify signatures and fetch records.

### Record encryption

The record body is symmetrically encrypted:

```
K = KDF(h, Z)         # symmetric key derived from blinding factor and zone key
IV = (NONCE || expiration_timestamp || block_counter)
BDATA = Encrypt(K, IV, RDATA)
```

For PKEY zones: AES-256-CTR. For EDKEY zones: ChaCha20.

The expiration timestamp in the IV means a record block cannot be decrypted after expiry, even if a resolver has the key, unless the expiry is known and matched. This prevents stale data exploitation.

---

## What key blinding protects against

| Adversary capability | Protected? | Notes |
|---------------------|-----------|-------|
| DHT peer observes a GET request | Yes | Sees only blinded key `Z_l`; cannot identify zone `Z` or label `l` |
| DHT peer receives a PUT | Yes | Encrypted RRBLOCK; blinded storage key; cannot read record contents |
| DHT peer enumerates all labels of a zone | Yes (probabilistic) | Would require brute-forcing `H(Z, l)` over all candidate labels |
| Network observer correlates GETs from same resolver | Partial | Multiple GETs for different labels of the same zone produce different, unlinkable keys at the DHT level |
| Strong adversary identifies zone owner peer | **No** | A strong adversary watching P2P traffic can determine which peer publishes (PUTs) records for a zone; RFC 9498 §10 acknowledges this |
| Offline dictionary attack on label space | Partial | Cost scales with dictionary size and zone key space; large dictionaries are feasible for determined adversaries |

---

## Comparison to other systems

| System | Query privacy | Zone enumeration resistance |
|--------|--------------|---------------------------|
| DNS | None (cleartext queries to authoritative servers) | Trivial (AXFR, NSEC walking) |
| DNSSEC + NSEC3 | None (queries still to authoritative) | Brute-forceable hash |
| ENS | None (on-chain state is public) | Trivial (all records on-chain) |
| Handshake | None (full node state is public) | Trivial (all TLDs on-chain) |
| **GNS** | **DHT-level query privacy via key blinding** | **Requires brute force over (zone key, label) space** |

---

## Limitations

1. **Not anonymous**: GNS is explicit that it provides query privacy, not anonymity. A resolver's identity is visible to the DHT peers it contacts.
2. **Zone owner peer identifiable**: the peer that repeatedly PUTs records for a zone can be identified with sufficient network monitoring. This is documented in RFC 9498 §10 as a known limitation.
3. **Local resolver knows everything**: the user's own GNS resolver obviously knows both `Z` and `l` for every query it makes; privacy is only against external DHT peers.
4. **DNS2GNS gateways break privacy**: when a query is routed through a public gateway (for legacy DNS compatibility), the gateway learns the full name being resolved.

---

## Behaviours relevant to RFP requirements

| Behaviour | Implication |
|-----------|------------|
| DHT storage key must not reveal zone identity | A naming system MUST use key blinding or equivalent to prevent DHT peers from linking queries to zone identities |
| Record contents must be encrypted in DHT | Raw record data MUST NOT be stored in plaintext in a public DHT |
| Expiry baked into encryption prevents stale exploitation | Record expiry SHOULD be enforced cryptographically, not only at the application layer |
| Query privacy is distinct from anonymity | A naming system MUST clearly distinguish between query privacy (DHT peers cannot identify the query subject) and anonymity (requester identity hidden); GNS provides the former, not the latter |
| On-chain naming systems cannot provide query privacy | Any system storing names as on-chain state has inherently public zone enumeration; privacy requires an off-chain DHT layer |

---

## Sources

- [RFC 9498: The GNU Name System §5 (key blinding), §10 (security)](https://www.rfc-editor.org/rfc/rfc9498.html)
- [LSD0001: The GNU Name System](https://lsd.gnunet.org/lsd0001/)
- [GNUnet user docs: GNS privacy](https://docs.gnunet.org/latest/users/gns.html)
- [GNUnet FAQ: GNS privacy limitations](https://www.gnunet.org/en/faq.html)
- [draft-schanzen-gns-28](https://datatracker.ietf.org/doc/html/draft-schanzen-gns-28)
- [[gns]] — main project note
- [[gns-dht-name-resolution]] — DHT storage and retrieval details
