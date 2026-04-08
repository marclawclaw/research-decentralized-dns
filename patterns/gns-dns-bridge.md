---
tags: [pattern, dns, compatibility, bridge, interop, gns, gnunet, legacy]
pattern: DNS bridge and legacy compatibility for alternative naming systems
seen-in: [gns]
related: [gns-dht-name-resolution, gns-zone-based-naming]
---

# Pattern: DNS Bridge / Legacy Compatibility (GNS)

Any alternative naming system must address how existing applications, browsers, and infrastructure that expect DNS interact with the new system. GNS provides multiple integration paths with varying levels of trust impact and deployment complexity.

---

## The compatibility problem

Legacy applications resolve names via the system stub resolver (usually `/etc/resolv.conf` → a DNS recursive resolver). Changing this path without breaking existing DNS resolution requires careful integration. GNS names are not valid DNS names (they may use `.gns.alt` or zTLD suffixes not recognised by DNS), so a translation layer is always necessary.

---

## GNS integration mechanisms

### 1. NSS plugin (`nss-gns`)

- Modifies `/etc/nsswitch.conf` to add `gns` as a resolution backend before or after `dns`.
- The GNS resolution step intercepts names matching GNS TLDs (e.g., `.gns.alt`, `.gnunet.org`) and resolves them via the local GNS daemon.
- DNS fallback continues for non-GNS names.
- **Trust impact**: none; full cryptographic verification retained.
- **Deployment**: system-level configuration change; requires root; works for all applications without per-app changes.

### 2. dns2gns proxy (`gnunet-dns2gns`)

- Runs a local DNS server (code default port 2853; GNUnet docs recommend port 5353 for non-privileged use; must be port 53 if editing `/etc/resolv.conf` directly).
- Receives DNS queries from any application; routes GNS-matching queries to the local GNS daemon; forwards all others to a configured upstream DNS resolver.
- Integrates with `systemd-resolved` via `resolvectl`.
- **Trust impact**: none for GNS names; trust depends on upstream DNS for non-GNS names.
- **Deployment**: daemon must be running; firewall may need opening for port 53; works for all applications.

### 3. SOCKS5 proxy (`gnunet-gns-proxy`)

- Runs a SOCKS5 proxy on port 7777 by default.
- Applications (browsers) route their HTTPS connections through this proxy.
- The proxy intercepts TLS connections for GNS names, resolves them via GNS, and provides TLS/X.509 revalidation using zone-embedded certificates.
- **Trust impact**: none for GNS names; full TLS verification.
- **Deployment**: per-application proxy configuration (browser-specific steps for Chrome, Firefox); does not affect system DNS.

### 4. Public DNS2GNS gateways

- Third parties run public DNS resolvers that accept queries for GNS names and resolve them via GNS.
- Search engine crawlers can use these to index GNS-hosted content.
- **Trust impact**: **breaks cryptographic chain of trust**; the gateway is a trusted third party for name resolution. The censorship-resistance property is lost.
- **Use case**: read-only access for legacy systems that cannot be reconfigured; search engine indexing.

---

## GNS2DNS delegation record

The GNS2DNS record type allows a GNS zone owner to delegate a sub-name back to legacy DNS:

```
example.myzone → GNS2DNS → example.com @ns1.example.com
```

Resolution proceeds via GNS until the GNS2DNS record is encountered, then switches to DNS. The resolver queries the specified DNS server for the remainder of the name.

Multiple GNS2DNS records may exist under the same label (tried in parallel or sequence). The DNS server can be specified as an IP address, a hostname, or a GNS-relative name ending in `.+`.

DNSSEC DS records may accompany a GNS2DNS record to secure the DNS delegation.

**Trust impact**: the GNS-to-DNS handoff introduces DNS trust assumptions from that point in the name. This is documented and expected; the boundary is explicit in the record.

---

## The `.alt` TLD (RFC 9476)

RFC 9476 (Sep 2023) defines `.alt` as a reserved pseudo-TLD for alternative naming systems that do not use DNS for resolution. GNS uses `.gns.alt` as its namespace under `.alt`. The FCFS registrar offers `.pin.gns.alt`.

Properties of `.alt`:

- `.alt` is not delegated in the DNS root zone; DNS resolvers return NXDOMAIN for any `.alt` query.
- Alternative naming systems are encouraged (not required) to use `.alt` as a suffix to signal to resolvers that the name is not a DNS name.
- No IANA registry or governance model for the `.alt` namespace; collision resolution is out of scope.
- Reduces but does not eliminate the risk of a future ICANN TLD conflicting with an alternative namespace TLD.

GNS pre-dates `.alt` and previously used `.gnu` as an informal TLD. RFC 9498 recommends `.gns.alt` as the canonical GNS TLD.

---

## DNS migration tools

GNS provides tooling for operators migrating from legacy DNS zones:

| Tool | Method | Suitable for |
|------|--------|-------------|
| `gnunet-namestore-zonefile` | Imports BIND-format zone files; converts TTL seconds to GNS microsecond expiration | One-time migration of static zones |
| `gnunet-zoneimport` | Reads domain names from stdin; queries authoritative DNS servers; imports results | Zones without AXFR access; live mirroring via polling |
| Ascension | Python daemon using AXFR/IXFR transfers; monitors SOA serial for changes | Ongoing synchronisation of large live zones |

Ascension was released in GNUnet 0.25.0 (Sep 2025). GNUnet used it to mirror .ee and .nu TLDs onto the main peer as of Aug 2025 (proof-of-concept, not production).

---

## BOX record: eliminating extra lookups for TLSA/SRV

DNS requires TLSA and SRV records to be stored under `_service._protocol.name` labels (e.g., `_443._tcp.example.com`). In a DHT system, resolving such names requires a separate DHT lookup for the underscore label.

The BOX record type avoids this by storing SRV and TLSA records in the record set of the parent label (`example.myzone`) as BOX records:

```
BOX(service=443, protocol=tcp, type=TLSA, data=<cert data>)
```

On resolution, the GNS resolver assembles the TLSA record from the BOX without an additional DHT round-trip. This:

- Reduces resolution latency for HTTPS/TLS.
- Makes TLS certificate data inseparable from the address record, strengthening DANE-style certificate pinning.

SBOX extends this to arbitrary underscore-label variants (SMIMEA, URI, etc.) by storing the string representation of the prefix instead of numeric service/protocol values.

---

## Behaviours relevant to RFP requirements

| Behaviour | Implication |
|-----------|------------|
| Multiple DNS integration paths with varying trust | A naming system MUST document all DNS integration mechanisms and their trust implications clearly |
| GNS2DNS for namespace bridging | A naming system SHOULD provide a delegation record type for handing off resolution to legacy DNS where needed |
| `.alt` TLD for namespace disambiguation | Alternative naming systems SHOULD use the `.alt` pseudo-TLD to signal non-DNS namespaces to resolvers |
| BOX/SBOX records avoid extra DHT round-trips | Service record types (TLSA, SRV) SHOULD be co-located with address records to avoid extra DHT lookups |
| Public gateways break cryptographic trust | Operators MUST be warned that DNS2GNS gateways break censorship resistance and SHOULD only be used for indexing, not for security-critical applications |
| DNS zone import tooling | A production naming system SHOULD provide migration tooling (zonefile import, AXFR mirroring) for operators transitioning from DNS |

---

## Sources

- [GNUnet user docs: DNS compatibility](https://docs.gnunet.org/latest/users/gns.html)
- [gnunet-dns2gns man page](https://manpages.org/gnunet-dns2gns)
- [gnunet-namestore-fcfsd man page](https://manpages.org/gnunet-namestore-fcfsd)
- [RFC 9498: The GNU Name System §8 (GNS2DNS record)](https://www.rfc-editor.org/rfc/rfc9498.html)
- [RFC 9476: The .alt Special-Use Top-Level Domain](https://datatracker.ietf.org/doc/rfc9476/)
- [LSD0008: The GNS SBOX Record Type](https://lsd.gnunet.org/lsd0008/)
- [GNUnet NGI Entrust GNS TLDs update (Aug 2025)](https://www.gnunet.org/en/news/2025-08-NGI-Entrust-GNS-TLDs-Update.html)
- [GNUnet 0.25.0 release notes (Ascension)](https://www.gnunet.org/en/news/2025-09-0.25.0.html)
- [[gns]] — main project note
