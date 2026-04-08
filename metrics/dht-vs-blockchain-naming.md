---
tags: [metrics, comparison, dht, blockchain, naming, gns, handshake, ens]
metric: DHT-based vs blockchain-based naming: architectural comparison
researched: 2026-04-08
---

# Metric: DHT-based vs Blockchain Naming

A cross-project comparison of architectural properties relevant to a decentralised naming RFP. The primary reference is [[gns]] (DHT/P2P, chain-agnostic); comparators are [[handshake]] (PoW blockchain) and ENS (smart contract on Ethereum).

---

## Adoption and network size

| Metric | GNS | Handshake | ENS |
|--------|-----|-----------|-----|
| Active node count | [NOT FOUND]; self-described "tiny" (Mar 2025) | [NOT FOUND] (hsd full node count unpublished) | [NOT FOUND] (Ethereum nodes ≠ ENS-specific) |
| Registered names | [NOT FOUND] (FCFS registrar count unpublished) | 11.3 million TLDs registered | ~3.8 million .eth names registered (2024 per ENS stats page) |
| Market cap (token) | None (no token) | ~$3.3M USD (Apr 2026) | ENS token: ~$200M+ (varies; not directly correlated to name count) |
| Standardisation | RFC 9498 (IETF Informational, Nov 2023) | None | ERC-137 (Ethereum community standard) |
| Academic citations | [NOT FOUND] (CANS 2014 paper on Semantic Scholar) | N/A | N/A |

---

## Technical properties

| Property | GNS | Handshake | ENS |
|----------|-----|-----------|-----|
| Consensus mechanism | None (DHT) | PoW (BLAKE2b+SHA3) | Ethereum PoS (consensus inherited) |
| Token required | No | Yes (HNS for auctions) | Yes (ETH for gas; ENS token for governance) |
| Name squatting prevention | By design (no global human-readable namespace) | Vickrey auction + 2-year renewal | Annual fee (~$5/yr for 5-char names) |
| Zone enumeration | Requires brute force on (zone key, label) | Trivial (all names on-chain) | Trivial (all names on-chain) |
| Query privacy | DHT-level (key blinding) | None (full node has all data) | None (blockchain is public) |
| Name granularity | Second-level (arbitrary depth via delegation) | TLD only (SLDs off-chain) | Second-level (.eth) |
| Record encryption in storage | Yes (AES-256-CTR / ChaCha20) | No (DNS data in plaintext UPDATE covenant) | No (resolver records on-chain in plaintext) |
| Revocation | PoW-based revocation certificate (days to compute) | REVOKE covenant (immediate, irreversible) | Transfer/burn token |
| Key loss recovery | None (zone permanently lost without pre-computed revocation) | None (UTXO lost permanently) | None (NFT permanently lost) |
| DNS compatibility | dns2gns proxy, NSS plugin, SOCKS5 proxy, GNS2DNS delegation record | hnsd SPV resolver translates HNS → DNS; hns.to gateways | No native DNS compat; third-party gateways (eth.limo, etc.) |
| DNS record types supported | All standard + GNS-specific (BOX, SBOX, VPN, PKEY, EDKEY, GNS2DNS, DID_DOCUMENT) | Up to 512 bytes per UPDATE covenant (raw DNS data; any type) | A, AAAA, TXT, CNAME, MX, etc. (via PublicResolver) |
| Large zone support | Yes (PostgreSQL backend, EdDSA keys, bulk import API; .ee and .nu mirrored Aug 2025) | Limited (512 bytes per UPDATE covenant; unsuitable for large zones) | Limited (gas costs scale with record count) |
| Post-quantum readiness | In progress (GNUnet 0.26.x PQ layer, Dec 2025) | None noted | None noted |

---

## Decentralisation properties

| Property | GNS | Handshake | ENS |
|----------|-----|-----------|-----|
| Root authority | None (each user is their own root) | All full nodes equally; no admin key | ENS DAO (multi-sig governance) |
| Can a third party censor a name? | No (zone owner controls records; DHT has no admin) | Only by 51% attack on PoW | ENS DAO has admin powers over .eth |
| Can a third party transfer a name? | No (private key required) | Only by 51% attack | ENS DAO could in theory |
| Infrastructure dependencies | GNUnet P2P nodes (altruistic participation) | Miners + full nodes | Ethereum validators + ENS contracts |
| Chain-agnostic | **Yes** (no blockchain dependency) | No (Handshake chain only) | No (Ethereum only; ENSv2 adds L2s) |

---

## Usability and maturity

| Property | GNS | Handshake | ENS |
|----------|-----|-----------|-----|
| End-user setup complexity | High (daemon, key management, resolver config) | Medium (Bob Wallet GUI available) | Low (MetaMask + browser plugin) |
| Browser native support | None | None | None (MetaMask extension required) |
| One-click registration | No | Namebase marketplace | ENS app (app.ens.domains) |
| Production maturity | Alpha ("early adopters only") | Production (but declining ecosystem activity) | Production |
| Active development (as of Apr 2026) | Yes (v0.27.0 released Mar 2026) | Yes (hsd actively maintained) | Yes |

---

## Sources

- [GNUnet 0.24.0 release notes (network size warning)](https://www.gnunet.org/en/news/2025-03-0.24.0.html)
- [Handshake registrations: HIP-68 discussion](https://github.com/handshake-org/HIPs/discussions/68)
- [Handshake market cap: CoinMarketCap](https://coinmarketcap.com/currencies/handshake/)
- [RFC 9498: The GNU Name System](https://www.rfc-editor.org/rfc/rfc9498.html)
- [GNUnet NGI Entrust GNS TLDs update (Aug 2025)](https://www.gnunet.org/en/news/2025-08-NGI-Entrust-GNS-TLDs-Update.html)
- [GNUnet 0.26.2 post-quantum layer](https://www.sambent.com/gnunet-0-26-2-post-quantum-layer-and-utf-8-api-fixes/)
- [[gns]] — main GNS project note
- [[handshake]] — main Handshake project note
