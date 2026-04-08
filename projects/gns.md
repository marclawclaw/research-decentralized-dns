---
tags: [project, dht, p2p, naming, dns, gns, gnunet, chain-agnostic, rfc9498]
project: GNU Name System (GNS)
ecosystem: GNUnet (chain-independent, pure DHT/P2P)
status: active, research-grade
researched: 2026-04-08
---

# GNU Name System (GNS)

GNS is a decentralised, censorship-resistant domain name resolution protocol developed as part of the GNUnet peer-to-peer framework. It provides a privacy-enhancing alternative to DNS without relying on any blockchain or central authority. Each user is the root of their own namespace; delegation to other users' zones creates a graph of human-readable names anchored in cryptographic identities. GNS was standardised as RFC 9498 in November 2023.

- Official page: https://www.gnunet.org/en/gns.html
- RFC 9498: https://www.rfc-editor.org/rfc/rfc9498.html
- Developer docs: https://docs.gnunet.org/latest/users/gns.html
- LSD specification: https://lsd.gnunet.org/lsd0001/
- GANA record type registry: https://gana.gnunet.org/gnu-name-system-record-types/gnu_name_system_record_types.html
- Source code (mirror): https://github.com/yonasBSD/gnunet
- GNS DID method spec: https://lsd.gnunet.org/lsd0005/
- Academic paper (CANS 2014): https://link.springer.com/chapter/10.1007/978-3-319-12280-9_9

---

## Adoption Metrics

| Metric | Value | Date | Source |
|--------|-------|------|--------|
| FCFS registered names (.pin.gns.alt) | [NOT FOUND] | — | No public count available at fcfs.gnunet.org |
| Active GNUnet peer count | [NOT FOUND] (self-described "tiny" network) | Mar 2025 | [GNUnet 0.24.0 release notes](https://www.gnunet.org/en/news/2025-03-0.24.0.html) |
| Open bug count | ~190 | Mar 2025 | [GNUnet 0.24.0 release notes](https://www.gnunet.org/en/news/2025-03-0.24.0.html) |
| RFC 9498 publication date | November 2023 | Nov 2023 | [RFC Editor](https://www.rfc-editor.org/rfc/rfc9498.html) |
| GNUnet latest stable version | 0.27.0 | Mar 2026 | [Tux Machines announcement](https://news.tuxmachines.org/n/2026/03/20/GNU_Projects_GNUnet_0_27_0_and_libredwg_0_13_4_released.shtml) |
| Academic citations (CANS 2014 paper) | [NOT FOUND] (listed on Semantic Scholar) | — | [Semantic Scholar](https://www.semanticscholar.org/paper/A-Censorship-Resistant%2C-Privacy-Enhancing-and-Fully-Wachs-Schanzenbach/f447f115cecee88ab72a61e79b342cce8bc86954) |
| TLDs mirrored on GNUnet main peer | .ee and .nu (proof-of-concept) | Aug 2025 | [GNUnet NGI Entrust update](https://www.gnunet.org/en/news/2025-08-NGI-Entrust-GNS-TLDs-Update.html) |
| Funders | NLnet (NGI Zero Entrust, NGI Search and Discovery), European Commission, Deutsche Forschungsgemeinschaft | 2023-2026 | [GNUnet project pages](https://www.gnunet.org/en/gns.html) |

> **Adoption note:** GNUnet development releases explicitly warn that the network is "tiny and thus unlikely to provide good anonymity." No third-party stats on active nodes or GNS name registrations are publicly available. GNS remains primarily a research and standards artefact with limited production deployment.

---

## How It Works

### Zone model

Each user owns one or more **zones**. A zone is identified by a public key (not a human-readable string). The zone owner controls all names within that zone by publishing signed records. Users interact with zones through human-readable aliases called **petnames**: locally assigned names that map a chosen label to a zone's public key.

There is no globally shared human-readable root. Instead, resolution always starts from a configured **Start Zone**, which is a locally known (public key, label) pair. Transitivity via PKEY/EDKEY delegation records then allows resolving names across multiple zones.

### Zone key types (see [[gns-zone-based-naming]])

| Type | Curve | Signature scheme | Notes |
|------|-------|-----------------|-------|
| PKEY | Curve25519 (Montgomery) | ECDSA with RFC 6979 deterministic signatures | Default; ECDSA (not EdDSA) required because key blinding needs linearity that EdDSA's private key hash destroys |
| EDKEY | edwards25519 (Ed25519) | Modified EdDSA with nonce derived from key expansion + blinding factor | Faster; recommended for large zones; non-standard EdDSA variant |

Both key types fit into a single DNS label (32 bytes + 4-byte type tag), which supports usability with existing DNS tooling.

### Key blinding and DHT storage (see [[gns-dht-name-resolution]])

This is the core privacy mechanism. Records are never stored under the raw zone public key; instead:

1. A Zone Key Derivation Function (ZKDF) takes the zone key `Z` and label `l` as input and produces a derived (blinded) key `Z_l`.
2. The blinded key `Z_l` is used as the DHT lookup key.
3. A resolver who knows both `Z` and `l` can independently derive `Z_l` and fetch the record.
4. An adversary who observes the DHT query learns only `Z_l`. Without knowing both `Z` and `l`, they cannot link the query to the zone or enumerate the zone's contents.
5. Record data is symmetrically encrypted (AES-256-CTR for PKEY zones; ChaCha20 for EDKEY zones) using a key derived from the same blinding process. The expiration time is incorporated into the initialisation vector.

This scheme prevents zone enumeration and query correlation without requiring any anonymising overlay beyond the DHT itself.

### Resolution algorithm

Name resolution in GNS is recursive:

1. Parse the rightmost label of the query as the Start Zone (either a configured petname or a raw zTLD public key string).
2. Derive the blinded DHT key for `(zone_key, label)`.
3. Fetch the encrypted record block from the DHT (via GNUnet's R5N DHT; see [[gns-dht-name-resolution]]).
4. Decrypt using the derived symmetric key.
5. Verify the block signature against the blinded public key.
6. If the result contains a PKEY or EDKEY record: follow the delegation to the referenced zone and continue from step 2 with the next label.
7. If the result contains a GNS2DNS record: hand off to DNS resolution for the remainder.
8. Return the resolved record set (A, AAAA, TXT, MX, etc.) to the caller.

The recursion depth is bounded by the number of labels in the name.

### R5N DHT

GNS uses GNUnet's R5N ("Randomized Recursive Routing for Restricted-route Networks") DHT. R5N differs from Kademlia by:

- Starting each lookup with a random walk of `log n` hops before switching to greedy XOR routing. This avoids local minima in overlay topologies that are not fully connected.
- Using a per-path Bloom filter to prevent routing loops.
- Validating data on-path (malformed records are dropped by relaying peers).
- Supporting topologies with restricted routes (NAT, firewalls), making it more suitable for consumer deployments.

R5N is being standardised separately as an IETF Internet-Draft (draft-schanzen-r5n).

### Name publishing

1. Users manage records in the local **NAMESTORE** (SQLite by default; PostgreSQL for large zones).
2. Records are marked public or private. Private records are never published to the DHT.
3. The **ZONEMASTER** service periodically signs public record sets (using the blinded key for each label), encrypts them, and publishes the resulting **RRBLOCK** to the DHT.
4. The **NAMECACHE** caches blocks retrieved from the DHT for local reuse.

Record TTLs are encoded as absolute expiration timestamps (in microseconds) and are incorporated into the RRBLOCK encryption. Expired blocks cannot be decrypted with the current IV, limiting stale data propagation.

---

## Record Types

GNS supports all standard DNS record types (A, AAAA, MX, TXT, NS, CNAME, etc.) plus GNS-specific types:

| Type | Purpose |
|------|---------|
| PKEY | Delegation to another ECDSA zone; enables petname chaining |
| EDKEY | Delegation to another EdDSA zone |
| GNS2DNS | Delegates resolution to a legacy DNS server; enables GNS-to-DNS namespace bridging |
| BOX | Wraps SRV and TLSA records to avoid separate DHT lookups for `_service._protocol` labels; stores them in the parent label's record set |
| SBOX | Extension of BOX handling arbitrary underscore-label prefixes (SMIMEA, URI, etc.) |
| VPN | Points to a GNUnet Virtual Public Network peer/service; resolver allocates a local IPv4/IPv6 address |
| LEHO | Legacy hostname hint; tells the resolver what HTTP Host header to use when connecting via a GNUnet overlay service |
| DID_DOCUMENT | W3C DID Document stored directly in a GNS zone (see [[gns-did-method]]) |

See the full GANA registry at https://gana.gnunet.org/gnu-name-system-record-types/gnu_name_system_record_types.html.

---

## DNS Compatibility and Bridge Mechanisms (see [[gns-dns-bridge]])

GNS provides several integration paths for legacy DNS clients:

| Mechanism | How it works | Trust implications |
|-----------|-------------|-------------------|
| **NSS plugin** | Adds `gns` entry to `/etc/nsswitch.conf`; system resolver calls GNS for matching names | Full cryptographic verification; no trust compromise |
| **dns2gns proxy** | Runs a local DNS server on port 5253; forwards matching queries to GNS, all others to upstream DNS | Full verification for GNS names; trust depends on upstream for DNS |
| **gnunet-gns-proxy** | SOCKS5 proxy (port 7777); intercepts HTTPS and performs GNS resolution with TLS/X.509 revalidation | Full verification; requires browser SOCKS5 configuration |
| **systemd-resolved integration** | `resolvectl` routes GNS TLD queries to the local dns2gns port | Full verification |
| **GNS2DNS gateways** | Public servers that resolve GNS names via DNS; used by search engine crawlers | **Breaks cryptographic chain of trust**; gateway is trusted third party |

The `.alt` pseudo-TLD (RFC 9476, Sep 2023) provides the standardised namespace slot for alternative naming systems. GNS uses `.gns.alt` as its namespace under `.alt`, with `.pin.gns.alt` offered by the FCFS registrar as a shared discovery zone.

---

## Trust Model and Name Squatting (see [[gns-petname-system]])

GNS explicitly rejects a global human-readable namespace. This is a deliberate design choice rooted in the impossibility of simultaneously achieving the three properties of Zooko's triangle (global, secure, human-readable) without a trusted authority.

**How GNS resolves the triangle:**

- **Global and not human-readable**: zTLDs (the base32 encoding of a zone public key) are globally unique and secure, but not memorable.
- **Human-readable and not globally unique**: petnames are memorable but only meaningful within a user's local trust graph.
- **Secure by construction**: every record is signed by the zone's private key; forgery is computationally infeasible.

**Consequence for name squatting:** There is no globally valuable human-readable namespace to squat. Squatting a petname in one user's configuration has no effect on any other user. The incentive that drives squatting in DNS (owning the globally canonical "google" label) does not exist in GNS.

**Trade-off:** Bootstrapping discovery of a new zone's public key out-of-band (e.g., exchanging a QR code, trusting a FCFS registrar's zone key, or following a friend's PKEY delegation) remains a usability challenge.

**FCFS registrar model:** The `gnunet-namestore-fcfsd` tool implements a first-come, first-served web service where users can register a subdomain label under the registrar's zone. The official instance at `fcfs.gnunet.org` offers names under `.pin.gns.alt`. Registrations do not expire. GNU Taler payment integration (using demo KUDOS currency) is included. This is a voluntary convenience layer; it does not constitute a global namespace.

---

## Key Revocation

If a zone private key is compromised, GNS provides a revocation mechanism:

1. A revocation certificate is computed by solving an expensive proof-of-work puzzle (designed to take 4-5 days on typical hardware as of 2024).
2. The certificate is signed by the zone private key (so only the legitimate owner can revoke) and broadcast to the GNUnet overlay.
3. All subsequent resolution attempts involving the revoked key fail.

**Critical limitation:** If the private key is lost before a revocation certificate is precomputed, the zone cannot be revoked and names under it may resolve to stale or attacker-controlled records indefinitely. The recommended mitigation is to precompute and securely store the revocation certificate immediately after zone creation.

---

## Architecture Decisions

| Decision | Rationale | Trade-off |
|----------|-----------|-----------|
| No global root zone | Eliminates the need for any trusted authority; every user is their own root | Discovery of zone public keys requires out-of-band exchange or trust in a FCFS registrar |
| Key blinding for DHT storage | Prevents zone enumeration and query correlation from DHT peers | Slightly more complex cryptographic implementation; ECDSA linearity requirement constrains PKEY key type |
| Petname system (not global names) | Eliminates name squatting incentive; each user maintains their own namespace graph | Names are not universally resolvable; requires social coordination to build the delegation graph |
| PKEY uses ECDSA over Curve25519 (not EdDSA) | EdDSA's private key hashing destroys the algebraic linearity needed for key blinding | Unusual combination; ECDSA is generally considered less modern than EdDSA; EDKEY zones introduced EdDSA variant with custom nonce derivation |
| R5N DHT (not Kademlia) | Handles restricted-route topologies (NAT, firewalls); avoids routing local minima | R5N has smaller adoption/analysis base than Kademlia; random walk adds latency |
| Record expiry via IV not TTL | Expired records cannot be decrypted; stale DHT entries become inert automatically | Resolvers must republish records before expiry; ZONEMASTER does this automatically but requires a running GNUnet node |
| Chain-agnostic by design | No tokenomics, no blockchain dependency; can run on any IP-connected network | No economic incentive layer; DHT participation is altruistic; network remains small |

---

## Differentiators

1. **Only RFC-standardised, production-implemented DHT naming system**: RFC 9498 (Nov 2023) makes GNS the only chain-agnostic naming system with a published IETF-track specification defining wire formats, cryptographic routines, and resolution procedures.
2. **Query privacy without anonymity overlay**: the key-blinding scheme prevents DHT peers from linking a query to a zone or enumerating zone contents, without requiring Tor-like onion routing.
3. **No blockchain, no token**: participation requires only a GNUnet node; no economic barrier to zone creation or record publishing.
4. **Zooko's triangle resolved by design**: petnames + zTLDs cover both the human-readable and global-secure poles without compromise; the system is architecturally honest about the trade-off.
5. **DID method (LSD0005)**: zones double as W3C DID Document registries (`did:gns:`), enabling GNS as a self-sovereign identity infrastructure without additional components.
6. **DNS migration tooling**: Ascension (AXFR/IXFR daemon), gnunet-namestore-zonefile (BIND import), and gnunet-zoneimport (live DNS mirroring) allow incremental migration from legacy DNS zones.
7. **Post-quantum awareness**: GNUnet 0.26.x (Nov-Dec 2025) introduced post-quantum cryptographic layer work; future GNS key types may use post-quantum schemes.

---

## Limitations and Criticisms

| Limitation | Detail |
|-----------|--------|
| Tiny network | GNUnet itself acknowledges the network is "tiny"; sparse peers degrade DHT availability, routing quality, and privacy |
| Alpha-stage maturity | Still described as "only suitable for early adopters with some reasonable pain tolerance" after 20+ years of development (GNUnet 0.24.0, Mar 2025) |
| Steep usability curve | Manual peer management; no GUI for common tasks; configuration requires command-line proficiency |
| No global human-readable namespace | Deliberate design choice, but means GNS names are not interoperable without prior key exchange or FCFS registrar trust |
| Zone key loss is unrecoverable | Without a pre-computed revocation certificate, a lost zone key cannot be revoked; stale records persist |
| No strong anonymity at zone owner level | The RFC notes it is currently possible for a strong adversary to determine which peer is responsible for a zone; GNS provides query privacy, not full zone-owner anonymity |
| FCFS registrar as centralisation vector | The shared `.pin.gns.alt` discovery zone relies on a single registrar; trusting that zone key reintroduces a central point of trust for discovery (though not for record integrity) |
| Protocol incompatibility between major versions | 0.21.0, 0.24.0, and 0.27.0 each broke compatibility with prior network versions; fragmented peer connectivity during upgrades |
| Browser integration requires configuration | No browser natively supports GNS; SOCKS5 proxy, NSS plugin, or dns2gns required |
| No incentive for DHT participation | Pure altruism; no token reward for hosting records; network relies on ideological or institutional motivation |

---

## Ecosystem and Tooling

| Component | Description | Status |
|-----------|-------------|--------|
| gnunet (core daemon) | P2P framework; includes DHT, NAMESTORE, NAMECACHE, ZONEMASTER, IDENTITY subsystems | Active, v0.27.0 (Mar 2026) |
| gnunet-gns | CLI resolver | Active |
| gnunet-namestore | CLI record management | Active |
| gnunet-identity | Ego/zone key management | Active |
| gnunet-namestore-fcfsd | FCFS registrar web server; integrates GNU Taler for payments | Active (v1.0.7) |
| gnunet-dns2gns | DNS-to-GNS proxy for legacy DNS clients | Active |
| gnunet-gns-proxy | SOCKS5 proxy for browser GNS resolution with TLS revalidation | Active |
| Ascension | Python daemon for AXFR/IXFR DNS zone mirroring into GNS | Active (released 0.25.0, Sep 2025) |
| gnunet-namestore-zonefile | BIND zonefile importer for GNS | Active |
| gnunet-zoneimport | Live DNS zone scraper; converts DNS records to GNS | Active |
| gnunet-go | Alternative GNUnet implementation in Go | Experimental |
| GNS DID method (LSD0005) | Spec for using GNS zones as W3C DID registries | Draft spec, not widely implemented |

---

## Related Patterns

- [[gns-dht-name-resolution]] — how records are stored and looked up in the DHT via key blinding
- [[gns-zone-based-naming]] — zone model, PKEY/EDKEY types, delegation chains
- [[gns-petname-system]] — petname design, Zooko's triangle, no-global-namespace trade-off
- [[gns-query-privacy]] — key blinding as a privacy mechanism without anonymising overlay
- [[gns-dns-bridge]] — DNS2GNS, NSS plugin, SOCKS5 proxy, .alt TLD integration

---

## Related Projects

- [[handshake]] — PoW blockchain replacing the DNS root zone; contrasts with GNS's chain-agnostic DHT model
- [[ens]] — smart-contract naming on Ethereum; global human-readable names via token-backed auction
- [[namecoin]] — original blockchain naming (2011); .bit TLD; illustrates squatting problem GNS avoids

---

## Sources

- [RFC 9498: The GNU Name System](https://www.rfc-editor.org/rfc/rfc9498.html)
- [GNS official page](https://www.gnunet.org/en/gns.html)
- [GNUnet user docs: GNS](https://docs.gnunet.org/latest/users/gns.html)
- [GNUnet developer docs: GNS](https://docs.gnunet.org/v0.20.x/developers/gns/gns.html)
- [LSD0001: The GNU Name System specification](https://lsd.gnunet.org/lsd0001/)
- [LSD0005: The GNU Name System DID Method](https://lsd.gnunet.org/lsd0005/)
- [LSD0008: The GNS SBOX Record Type](https://lsd.gnunet.org/lsd0008/)
- [GANA: GNS record type registry](https://gana.gnunet.org/gnu-name-system-record-types/gnu_name_system_record_types.html)
- [Wachs, Schanzenbach, Grothoff: A Censorship-Resistant, Privacy-Enhancing and Fully Decentralized Name System (CANS 2014)](https://link.springer.com/chapter/10.1007/978-3-319-12280-9_9)
- [Grothoff: The GNU Name System (30C3 slides)](https://grothoff.org/christian/30c3gns.pdf)
- [R5N DHT spec (draft-schanzen-r5n)](https://lsd.gnunet.org/lsd0004/)
- [GNUnet 0.24.0 release notes](https://www.gnunet.org/en/news/2025-03-0.24.0.html)
- [GNUnet 0.25.0 release notes](https://www.gnunet.org/en/news/2025-09-0.25.0.html)
- [GNUnet 0.27.0 announcement](https://news.tuxmachines.org/n/2026/03/20/GNU_Projects_GNUnet_0_27_0_and_libredwg_0_13_4_released.shtml)
- [GNUnet NGI Entrust GNS TLDs update (Aug 2025)](https://www.gnunet.org/en/news/2025-08-NGI-Entrust-GNS-TLDs-Update.html)
- [GNUnet NGI Webinar: GNS and road to RFC (Feb 2024)](https://www.gnunet.org/en/news/2024-02-NGI-Webinar-GNS.html)
- [FCFS registrar (fcfs.gnunet.org)](https://fcfs.gnunet.org/)
- [gnunet-namestore-fcfsd man page](https://manpages.org/gnunet-namestore-fcfsd)
- [RFC 9476: The .alt Special-Use Top-Level Domain](https://datatracker.ietf.org/doc/rfc9476/)
- [GNUnet FAQ](https://www.gnunet.org/en/faq.html)
- [GNUnet Wikipedia article](https://en.wikipedia.org/wiki/GNUnet)
- [GNUnet bibliography](https://bib.gnunet.org/)
