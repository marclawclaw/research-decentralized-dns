---
tags: [pattern, zone, naming, delegation, cryptography, gns, gnunet]
pattern: Zone-based naming with cryptographic delegation
seen-in: [gns]
---

# Pattern: Zone-Based Naming (GNS)

GNS organises names into **zones**: cryptographically-identified namespaces each owned by a single entity. Zones are connected via delegation records into an arbitrarily deep graph. The zone model is the key architectural departure from DNS's globally-agreed hierarchy.

---

## Zone concept

A zone is a namespace identified by a public key. The owner of the corresponding private key can publish, update, and revoke any record within the zone. There is no central registry of zones; any key pair is a valid zone.

Zones have no inherent human-readable name. Human-readable names are petnames: user-local assignments of a chosen label to a zone public key (see [[gns-petname-system]]).

---

## Zone key types

| Type identifier | Curve | Signature | Use case |
|----------------|-------|-----------|---------|
| PKEY | Curve25519 (Montgomery) | ECDSA (RFC 6979 deterministic) | Default; required for key-blinding linearity |
| EDKEY | edwards25519 | Modified Ed25519 (custom nonce derivation) | Faster verification; recommended for zones with millions of records |

**Why not standard EdDSA for PKEY?** The key-blinding step requires that a blinded public key can be derived from the original public key by applying a scalar multiplication: `Z_l = h * Z` (where `h = H(Z, l)`). This requires the private key scalar to be directly usable as a multiplier. EdDSA hashes the private key before use, which destroys this linearity. GNS therefore uses raw ECDSA on Curve25519 for PKEY zones. For EDKEY zones, a custom nonce derivation replaces the RFC 8032 standard nonce to work around the same constraint.

Both zone identifiers are 36 bytes (32 bytes key + 4 bytes type tag), which fits within a single DNS label, enabling use as a zone TLD (zTLD) in DNS tooling without modification.

---

## Zone categories (from resolver perspective)

| Category | Description | Example |
|----------|-------------|---------|
| Local zone | User-owned zone; private key present locally | `myzone` (created with `gnunet-identity`) |
| Remote zone | Zone owned by another user; accessed via petname or zTLD | `bob` (mapped to Bob's PKEY) |
| zTLD | Remote zone accessed via its base32-encoded public key; no prior configuration needed | `5HT2...ABCD.gns.alt` |

---

## Delegation records

PKEY and EDKEY records are the mechanism for zone delegation. Adding a PKEY record under label `bob` in `myzone` delegates the `bob.myzone` namespace to Bob's zone:

```
gnunet-namestore -a -n bob --type PKEY -V $BOBKEY -e 1d -z myzone
```

Resolution of `www.bob.myzone`:

1. Resolve `myzone` from the Start Zone configuration.
2. Look up label `bob` in `myzone`; find a PKEY record pointing to Bob's zone key.
3. Look up label `www` in Bob's zone.
4. Return the A/AAAA records for `www`.

Delegation chains can be arbitrarily deep. Each hop requires one DHT lookup.

---

## Zone relative names

The special suffix `.+` means "relative to the current authoritative zone." This allows records within a zone to reference other labels in the same zone without hard-coding the full name, supporting portability when a zone is delegated under different petnames by different users.

---

## Record lifecycle

| Event | Mechanism |
|-------|-----------|
| Publish a record | Mark as public in NAMESTORE; ZONEMASTER signs and publishes to DHT |
| Update a record | Overwrite in NAMESTORE; ZONEMASTER publishes new block; old block expires by timestamp |
| Delete a record | Remove from NAMESTORE; old DHT block expires naturally (no explicit DHT DELETE needed) |
| Revoke zone | Pre-computed proof-of-work certificate broadcast to network; all resolution involving the key fails |

---

## Large zone support

For zones with millions of records (e.g., mirrored DNS TLDs):

- Use EDKEY zones (faster EdDSA operations).
- Use PostgreSQL NAMESTORE backend (SQLite is single-writer and not designed for bulk operations).
- Configure ZONEMASTER worker thread count: `gnunet-config -s zonemaster -o WORKER_COUNT -V N`.
- Use the NAMESTORE bulk import C API or REST endpoint.

GNUnet successfully mirrored the .ee and .nu TLDs onto the main GNUnet peer using Ascension in 2025 as a proof-of-concept for large-zone operation.

---

## Key backup and recovery

Critical files:

| File | Contents | Consequence of loss |
|------|----------|---------------------|
| `~/.local/share/gnunet/private_key.ecc` | Peer identity | Cannot reconnect as the same peer |
| `~/.local/share/gnunet/identity/egos/` | All zone private keys (unencrypted) | **All zones permanently lost**; cannot revoke; no recovery |

Revocation certificates must be pre-computed before any risk of key loss. Computation takes 4-5 days on typical hardware (2024); pre-computation and offline storage is the only recovery path.

---

## Behaviours relevant to RFP requirements

| Behaviour | Implication |
|-----------|------------|
| Zone = public key | A naming system MUST identify namespaces by cryptographic keys, not by registered strings |
| Any key pair is a valid zone | Zone creation MUST be permissionless; no registration fee or approval required |
| Delegation via special record type | A zone owner MUST be able to delegate a sub-namespace to another zone with a single record |
| No zone TTL; records have per-record expiry | Zone ownership is indefinite; individual records expire and are republished |
| Private key loss is unrecoverable | System designers MUST consider key custody and revocation pre-computation as critical UX requirements |
| EDKEY for large zones | Record storage and signing architecture MUST scale to zones with millions of entries via appropriate key type and database backend |

---

## Sources

- [RFC 9498: The GNU Name System](https://www.rfc-editor.org/rfc/rfc9498.html)
- [GNUnet user docs: GNS zones](https://docs.gnunet.org/latest/users/gns.html)
- [draft-schanzen-gns-28](https://datatracker.ietf.org/doc/html/draft-schanzen-gns-28)
- [GNUnet NGI Entrust GNS TLDs update (Aug 2025)](https://www.gnunet.org/en/news/2025-08-NGI-Entrust-GNS-TLDs-Update.html)
- [[gns]] — main project note
