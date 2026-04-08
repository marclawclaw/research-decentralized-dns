# Pattern: Self-Certifying Names

## Summary

A naming system is self-certifying when the name itself encodes a public key fingerprint, allowing anyone who receives a record to verify its authenticity cryptographically without contacting a trusted authority. No certificate authority, DNS root, or blockchain is required to validate the binding between a name and its current value.

The concept originates from the Self-Certifying File System (SFS) by Mazieres et al. (1999) and is the foundational design of [[projects/ipns]].

## Observed In

- [[projects/ipns]] (IPNS): name = `SHA2-256(public_key)`, encoded as a CIDv1 with the `libp2p-key` multicodec. Records are signed with the corresponding private key (Ed25519 by default).
- GNU Name System (GNS): zones are identified by public keys; records are signed by zone owners.
- Secure Scuttlebutt (SSB): feed identifiers are Ed25519 public keys; all messages are signed.
- DIDs (W3C Decentralised Identifiers): the `did:key` method encodes a public key directly in the identifier.

## How It Works (IPNS)

```
1. Key generation
   Private key (Ed25519) → Public key → SHA2-256 hash → CIDv1 (libp2p-key codec)
   Result: an IPNS name, e.g. k51qz...

2. Record publication
   Publisher signs {value, sequence, validity, ttl} with the private key
   → Signed record distributed to DHT / PubSub / HTTP routing

3. Record verification (by anyone)
   a. Extract public key from record (or derive from name if Ed25519)
   b. Verify signatureV2 over DAG-CBOR data field
   c. Check validity timestamp (record not expired)
   d. Check sequence number (record is the latest seen)
   → If all pass: record is authentic; no external trust anchor needed
```

## Key Behaviours

| Behaviour | Implication |
|-----------|-------------|
| Name = public key fingerprint | Anyone can verify authenticity without a trusted authority |
| Permissionless name creation | Generate a keypair offline; the name is immediately valid globally |
| Control = key possession | Whoever holds the private key controls the name; no password reset or admin override possible |
| No namespace registry | Names cannot conflict (each is a unique key hash); no first-come-first-served registration |
| Key loss is permanent | Losing the private key means permanently losing the ability to update the name |
| No revocation (in IPNS) | There is currently no standardised way to invalidate a compromised key; the attacker retains control |

## Contrast With Trust-Anchored Naming

| Dimension | Self-Certifying (IPNS) | DNS / ENS / Handshake |
|-----------|----------------------|----------------------|
| Who validates the name-key binding? | Anyone (cryptographic proof) | Root authority (DNS ICANN, ENS DAO, blockchain consensus) |
| Registration required? | No; any keypair is valid | Yes; registration with a registry |
| Cost | Zero | Gas fee / registrar fee |
| Namespace | Cryptographic (unguessable identifiers) | Human-readable strings |
| Key rotation | Loses the name | Managed separately from the name |
| Revocation | Not standardised in IPNS | Supported (DNS: delete record; ENS: transfer or set resolver to zero) |

## Composability with Human-Readable Names

Self-certifying names solve authentication but not discoverability. Human-readable names are layered on top:
- **DNSLink:** DNS TXT record maps `example.com` → `/ipns/k51qz...` (DNS provides human-readability; IPNS provides authenticity)
- **ENS:** `.eth` name stores IPNS name as `contenthash` (ENS provides human-readability and on-chain ownership; IPNS provides mutable content routing)

The self-certifying layer and the human-readable layer are decoupled and independently upgradeable.

## Relevance for Decentralised DNS RFP

A decentralised DNS system for Logos could adopt self-certifying names at the cryptographic layer:
- Node identities and zone ownership could be expressed as key hashes, avoiding reliance on any external registry or blockchain for authentication.
- Human-readable names (`.logos`, `.eth`, etc.) would be layered on top via a separate namespace registry.
- Key rotation and revocation must be explicitly designed; the IPNS experience shows these are hard unsolved problems in pure self-certifying systems.

## Sources

- https://docs.ipfs.tech/concepts/ipns/ (IPFS Docs: IPNS, 2025)
- https://specs.ipfs.tech/ipns/ipns-record/ (IPFS Standards: IPNS Record spec, 2025)
- https://arxiv.org/pdf/2105.08395 (Academic: Self-verifiable mutable content in IPFS, 2021)
- https://github.com/ipfs/specs/issues/219 (GitHub: IPNS Key Revocation issue, ongoing)
- https://github.com/ipfs/camp/blob/master/DEEP_DIVES/16-revocation-rotating-of-ipns-keys.md (IPFS Camp: Key revocation deep dive)
- https://blog.ceramic.network/key-revocation-in-self-certifying-protocols/ (Ceramic: Key revocation in self-certifying protocols)

## Tags

#naming #cryptography #self-certifying #ipns #public-key #pattern
