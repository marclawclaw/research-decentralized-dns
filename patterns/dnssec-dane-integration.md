---
tags: [pattern, dnssec, dane, tlsa, certificate-authority, security, handshake]
pattern: DNSSEC and DANE Integration for CA Replacement
seen-in: [[handshake]]
---

# Pattern: DNSSEC and DANE Integration (CA Replacement)

A security architecture where DNS-Based Authentication of Named Entities (DANE, RFC 6698) is used to bind TLS certificates to domain names via DNS records, eliminating the need for a trusted third-party Certificate Authority (CA). When the DNS root of trust is secured by a PoW blockchain (as in Handshake), the entire chain of trust becomes permissionless and censorship-resistant.

---

## Where It Appears

- [[handshake]] (primary production example combining PoW root trust + DANE)
- Traditional DNSSEC + DANE (standards context; not decentralised)

---

## Background: The CA Problem

In the traditional web PKI, browsers trust a fixed set of root CAs. Any CA can issue a certificate for any domain, creating a single-point-of-failure trust model. A rogue or compromised CA can issue fraudulent certificates that most browsers will accept. DNSSEC + DANE addresses this by anchoring certificate trust to domain name ownership rather than to a CA list.

---

## How DANE Works

DANE (RFC 6698) uses TLSA DNS resource records to specify certificate constraints for a domain at a specific TCP/UDP port. A TLSA record can:

1. Specify that a particular CA must be the issuer (CA constraint)
2. Specify that a particular certificate must be used (service certificate constraint)
3. Specify a trust anchor certificate (trust anchor assertion)
4. Specify a certificate fingerprint directly (domain-issued certificate)

When the browser checks a TLSA record and the record matches the presented TLS certificate, trust is established without consulting any CA.

The weakness of DANE on traditional DNS: DNS itself is vulnerable to DNSSEC key compromise at higher levels of the hierarchy (root, TLD). If ICANN's root signing key were compromised, all DNSSEC trust would be undermined.

- Source: [RFC 6698](https://www.rfc-editor.org/rfc/rfc6698)
- Source: [DANE Wikipedia](https://en.wikipedia.org/wiki/DNS-based_Authentication_of_Named_Entities)

---

## Handshake's Enhancement: PoW Root of Trust

Handshake replaces the ICANN root zone (and therefore the DNSSEC root signing key) with a PoW blockchain. The chain of trust is:

```
User device
  -> hnsd/hsd (verifies blockchain PoW)
    -> HNS root zone (TLD NS/DS records on-chain)
      -> TLD owner's authoritative DNS (served by TLD owner's infrastructure)
        -> TLSA record for the service
          -> Self-signed TLS certificate (matches TLSA)
```

Because the TLSA record is cryptographically committed to by the PoW chain (via the HNS root zone), no CA is needed. The certificate may be self-signed; trust is derived from blockchain PoW, not a trusted third party.

- Source: [Matthew Zipkin: Using HNS websites securely](https://matthewzipkin.medium.com/using-hns-websites-securely-69959ae02052)
- Source: [Matthew Zipkin: Building a secure HNS site](https://matthewzipkin.medium.com/building-a-secure-website-on-your-handshake-tld-a8922a950a4f)

---

## Implementation Steps (Handshake)

1. Register a TLD on Handshake via auction
2. Configure an authoritative name server for the TLD (off-chain)
3. Publish an NS record and/or DS record (DNSSEC delegation) in the HNS blockchain via UPDATE covenant
4. On the authoritative server, create a TLSA record for the target service
5. Generate a self-signed TLS certificate matching the TLSA record's fingerprint
6. Configure the web server to serve the self-signed cert

Visitors using a Handshake-aware resolver (hsd, hnsd, gateway) can verify the entire chain without relying on a CA.

---

## Current Adoption Barriers

| Barrier | Detail |
|---------|--------|
| Chrome does not support DANE | Google Chrome has deliberately not implemented DANE; most web users cannot benefit | Source: [Matthew Zipkin](https://matthewzipkin.medium.com/using-hns-websites-securely-69959ae02052) |
| Firefox partial support | Firefox has limited DANE support and not by default | |
| Requires HNS-aware resolver | Users must be running hsd, hnsd, or a trusted gateway to get HNS root zone data | |
| Complex setup | TLD owners must manage their own DNSSEC-signed authoritative zone | |
| Irreversible key loss | Loss of HNS private key means permanent loss of TLD and therefore the entire DANE chain | |

---

## RFP Implications

- A decentralised naming system can eliminate CA dependency by combining a decentralised root of trust (blockchain or DHT) with DANE/TLSA
- This is a technically sound but browser-adoption-limited approach as of 2026
- The RFP should consider whether CA elimination is a requirement or an optional property
- If the naming system is intended for use within a specific application stack (e.g., Logos/Waku), DANE can be enforced at the application layer without requiring browser support
- Privacy consideration: TLSA records reveal the certificate fingerprint; this is a minor metadata leak

---

## Sources

- [RFC 6698: DANE TLSA](https://www.rfc-editor.org/rfc/rfc6698)
- [DANE Wikipedia](https://en.wikipedia.org/wiki/DNS-based_Authentication_of_Named_Entities)
- [Matthew Zipkin: Using HNS websites securely](https://matthewzipkin.medium.com/using-hns-websites-securely-69959ae02052)
- [Matthew Zipkin: Building a secure HNS TLD site](https://matthewzipkin.medium.com/building-a-secure-website-on-your-handshake-tld-a8922a950a4f)
- [Handshake FAQ (CA replacement)](https://handshake.org/faq/)
- [HNS + PowerDNS + Nginx + DANE guide](https://blog.htools.work/posts/hns-pdns-nginx-part-3/)
