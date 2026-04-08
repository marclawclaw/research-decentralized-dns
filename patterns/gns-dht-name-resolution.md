---
tags: [pattern, dht, name-resolution, key-blinding, r5n, gnunet, gns]
pattern: DHT name resolution with key blinding
seen-in: [gns]
---

# Pattern: DHT Name Resolution (GNS / R5N)

The GNU Name System stores name records in a distributed hash table using a key-blinding scheme that decouples the observable DHT lookup key from the zone's public identity. This enables privacy-preserving record retrieval without any anonymising overlay network.

---

## Core mechanism

### Stored key

Records are not stored under the zone public key `Z`. Instead, for each label `l` in a zone, GNS derives a blinded key:

```
Z_l = ZKDF(Z, l)   # Zone Key Derivation Function
```

The blinded key `Z_l` is used as the DHT storage key. A resolver computes `Z_l` from `(Z, l)` and queries the DHT directly. A DHT peer observing the query sees only `Z_l` and cannot determine `Z` or `l` without a brute-force attack against the known space of zone keys and labels.

### Record encryption

The record set is encrypted before DHT storage. For PKEY zones (ECDSA): AES-256-CTR with key derived from the blinding process and an IV formed from `(NONCE || expiration_timestamp || block_counter)`. For EDKEY zones (EdDSA variant): ChaCha20 with the same key derivation. The expiration timestamp is baked into the IV, so a record block cannot be decrypted with an incorrect IV, making expired blocks cryptographically inert without requiring explicit DHT deletion.

### Signature verification

Each record block (RRBLOCK) is signed by `Z_l` (the blinded private key). A resolver verifies the signature using `Z_l`, which it derives independently. This provides:

- **Integrity**: only the zone owner (who knows the private key `z`) can produce valid signatures for any label.
- **Unlinkability**: signatures for different labels under the same zone are unlinkable by external observers.

---

## R5N DHT routing

GNS's underlying DHT is R5N ("Randomized Recursive Routing for Restricted-route Networks"). Key properties relevant to naming:

| Property | Kademlia | R5N |
|----------|---------|-----|
| Routing metric | XOR distance | XOR distance (greedy phase) |
| Bootstrap for restricted routes (NAT) | Problematic | Handled via random walk phase |
| Loop prevention | Not guaranteed | Bloom filter per path |
| Data validation on-path | Not standard | Required; malformed data dropped |
| Random walk before greedy routing | No | Yes (log n hops) |

The random walk phase prevents routing into local minima in partially-connected topologies, which is critical for a consumer-oriented P2P system where many nodes sit behind NAT.

---

## Privacy properties

| Threat | Protected? | Mechanism |
|--------|----------|-----------|
| DHT peer enumerates zone labels | Yes (probabilistic) | Queries arrive as blinded keys; without knowing `Z`, enumeration requires brute force |
| DHT peer correlates queries to a zone | Yes (probabilistic) | Different labels produce different, unlinkable blinded keys |
| Network observer identifies zone owner | **Partial** | A strong adversary can identify which peer publishes a zone's records; see GNS RFC §10 |
| Resolver learns queried label | No | The local resolver knows `(Z, l)` by definition |
| DNS-level observer leaks | Yes (for GNS path) | Only when using DNS2GNS gateway does the gateway learn the query |

---

## Resolution latency factors

- **DHT lookup**: O(log n) hops in the greedy phase, plus O(log n) random-walk hops. Empirical GNUnet performance has not been formally published for the current network.
- **Delegation chain depth**: each PKEY delegation requires an additional DHT lookup. A name with 3 labels plus 2 delegation steps requires 5 round-trips.
- **NAMECACHE**: recently resolved record blocks are cached locally by the NAMECACHE service, reducing repeat-query latency to near zero.
- **Records are republished proactively**: the ZONEMASTER service re-publishes records before expiry, keeping them available without a fresh DHT PUT on every resolution.

---

## Zone enumeration resistance

GNS is resistant to offline zone enumeration (analogous to the NSEC3 walking attack in DNSSEC):

- Without knowing both `Z` (the zone public key) and the set of candidate labels, a DHT peer cannot determine the contents of a zone.
- Online brute force (trying common label strings against a known `Z`) is possible but expensive; the cost scales with the size of the label dictionary.
- Compare to: DNSSEC with NSEC (fully enumeratable), Handshake (all names public on-chain, trivially enumeratable).

---

## Behaviours relevant to RFP requirements

| Behaviour | Implication |
|-----------|------------|
| Records stored under blinded keys | A naming system SHOULD store records such that DHT peers cannot correlate queries to zones without prior knowledge of both zone key and label |
| Expiry baked into encryption IV | A record SHOULD become cryptographically inaccessible after its expiry without requiring explicit deletion from the DHT |
| Signature tied to blinded key | Record integrity verification SHOULD not require knowledge of the zone's master public key; derived keys MUST suffice |
| NAMECACHE for repeat queries | A resolver SHOULD maintain a local cache of recently resolved records to avoid repeated DHT round-trips |
| No separate anonymity layer needed | Query privacy SHOULD be achievable through cryptographic blinding alone, without requiring onion routing or mix networks |

---

## Sources

- [RFC 9498: The GNU Name System §5-6 (key blinding, record format)](https://www.rfc-editor.org/rfc/rfc9498.html)
- [LSD0001: The GNU Name System](https://lsd.gnunet.org/lsd0001/)
- [R5N DHT spec (LSD0004)](https://lsd.gnunet.org/lsd0004/)
- [R5N paper: Randomized Recursive Routing (IEEE 2011)](https://ieeexplore.ieee.org/document/6060022/)
- [GNUnet developer docs: GNS resolution](https://docs.gnunet.org/v0.20.x/developers/gns/gns.html)
- [draft-schanzen-gns-28 (late IETF draft)](https://datatracker.ietf.org/doc/html/draft-schanzen-gns-28)
