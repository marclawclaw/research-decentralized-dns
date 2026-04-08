# Namecoin

**Type:** UTXO/PoW Blockchain (Bitcoin fork)
**Status:** Maintained; minimal adoption
**Website:** https://www.namecoin.org/
**Wikipedia:** https://en.wikipedia.org/wiki/Namecoin
**Primary TLD:** `.bit`
**Token:** NMC (Namecoin)

---

## Summary

Namecoin is the first blockchain-based naming system, launched on 18 April 2011 as a fork of Bitcoin. It introduced the `.bit` top-level domain, operated outside ICANN jurisdiction, and pioneered the concept of storing key-value data on a blockchain. Namecoin was the first published solution to [[zookos-triangle]] (the naming trilemma of human-meaningful, secure, and decentralised names).

Despite its historical significance, Namecoin failed to achieve mainstream adoption. The primary causes were [[name-squatting]] dominating registrations, the absence of native browser resolution support, and a near-absent path to recovering brand names from squatters. As of 2015, only 28 of approximately 120,000 registered `.bit` names had non-trivial content (Kalodner et al., WEIS 2015).

Namecoin is best studied as a **cautionary reference** for decentralised naming RFP design: it correctly identified the problem space but made economic and architectural choices that prevented real-world adoption.

---

## Key Metrics

| Metric | Value | Source / Date |
|--------|-------|---------------|
| Launch date | 18 April 2011 | [Wikipedia](https://en.wikipedia.org/wiki/Namecoin) (2024) |
| Total registered `.bit` names (2015 study) | ~120,000 | [Kalodner et al. 2015](https://econinfosec.org/archive/weis2015/papers/WEIS_2015_kalodner.pdf) |
| Active names with non-trivial content (2015) | 28 | [Kalodner et al. 2015](https://econinfosec.org/archive/weis2015/papers/WEIS_2015_kalodner.pdf) |
| Squatter-to-legitimate ratio (2015) | >99.98% squatted or inactive | [Kalodner et al. 2015](https://econinfosec.org/archive/weis2015/papers/WEIS_2015_kalodner.pdf) |
| Transfers from squatter to legitimate user (lifetime, 2015) | 14 (lower bound) to ~250 (upper bound) | [Kalodner et al. 2015](https://econinfosec.org/archive/weis2015/papers/WEIS_2015_kalodner.pdf) |
| Market cap (April 2026) | ~$13.2M USD | [CoinMarketCap](https://coinmarketcap.com/currencies/namecoin/) (Apr 2026) |
| Circulating supply | 14,736,400 NMC | [CoinMarketCap](https://coinmarketcap.com/currencies/namecoin/) (Apr 2026) |
| Max supply | 21,000,000 NMC | [Wikipedia](https://en.wikipedia.org/wiki/Namecoin) |
| Registration fee | 0.01 NMC | [Namecoin FAQ](https://www.namecoin.org/docs/faq/) |
| Name expiry | 36,000 blocks (~200 days) | [Namecoin FAQ](https://www.namecoin.org/docs/faq/) |
| Block time | ~10 minutes | [Wikipedia](https://en.wikipedia.org/wiki/Namecoin) |
| Consensus | Proof-of-Work, SHA-256, merged mining with Bitcoin | [Wikipedia](https://en.wikipedia.org/wiki/Namecoin) |
| Value size limit per record | 520 bytes (UTF-8 JSON) | [Namecoin Wiki: Domain Name Specification](https://wiki.namecoin.org/index.php?title=Domain_Name_Specification) |
| Latest stable release | 0.21.0 (January 2021) | [Wikipedia](https://en.wikipedia.org/wiki/Namecoin) (2024) |

---

## Architecture

### Blockchain Foundation

Namecoin is a fork of Bitcoin, adding a key-value namespace to the UTXO model. It uses SHA-256 proof-of-work and supports [[merged-mining-for-naming]], allowing miners to mine Bitcoin and Namecoin simultaneously at zero additional energy cost. Merged mining activated at block 19,200.

The blockchain maintains 21 million NMC as the maximum supply, with 10-minute target block times identical to Bitcoin.

### Namespace Structure

Names in Namecoin use prefixes to distinguish purpose:

| Prefix | Purpose | Example |
|--------|---------|---------|
| `d/` | DNS domain (maps to `.bit` TLD) | `d/example` → `example.bit` |
| `id/` | Decentralised identity | `id/alice` |

The `id/` namespace, launched May 2012, is considered the first Web3 decentralised identity service, predating ENS.

### Three-Operation Registration Model

Namecoin uses three distinct blockchain operations for name lifecycle management. See [[name-registration-two-phase-commit]] for the anti-frontrunning pattern.

**1. `name_new` (Pre-registration)**
- Broadcasts a salted hash commitment: `hash(d/example + random_salt)`
- Outputs a special unspendable UTXO with a 0.01 NMC fee
- Does not reveal the name publicly (prevents front-running)
- Implemented as `OP_NAME_NEW` script opcode

**2. `name_firstupdate` (Registration, minimum 12 blocks after `name_new`)**
- Spends the `name_new` UTXO
- Reveals the name and salt to validate the commitment
- Sets the initial value as a UTF-8 JSON string (≤520 bytes)
- Implemented as `OP_NAME_FIRSTUPDATE` opcode

**3. `name_update` (Update/Renewal)**
- Spends the current name UTXO and issues a replacement
- Resets the expiry timer
- Can modify the value at the same time
- Implemented as `OP_NAME_UPDATE` opcode
- No fee beyond the standard transaction fee

### DNS Record Format

The `.bit` domain value is a UTF-8 JSON object of ≤520 bytes. It supports:
- `ip` / `ip6`: A and AAAA records
- `ns`: delegate to a nameserver
- `alias`: CNAME-equivalent
- `ds`: DNSSEC DS records
- `tls`: dehydrated TLS certificates (compressed to fit within the 520-byte limit)
- `tor`: `.onion` address mapping (privacy use case)
- `i2p`: I2P address mapping

The 520-byte limit can be worked around using an `import` directive referencing up to 3 other names, allowing approximately 2 KB of total data.

### Name Expiry and Renewal

See [[name-expiry-renewal-mechanism]] for full pattern details.

- **Semi-expiry:** after 31,968 blocks (~222 days) without renewal, the name stops resolving but the owner retains exclusive right to renew.
- **Full expiry:** after an additional 4,032 blocks (~28 days) of non-renewal (total ~250 days), the name becomes available for re-registration by anyone.
- Renewal requires only a `name_update` transaction (standard transaction fee, no registration fee).

### Resolution Infrastructure

Namecoin has no built-in DNS resolution. Users require one of:

1. **Full node**: runs the Namecoin Core daemon locally and uses `ncdns` (a Namecoin-to-DNS bridge) to expose a local DNS resolver.
2. **Browser extension**: various extensions that intercept `.bit` lookups and query a trusted node.
3. **Third-party proxy** (deprecated): OpenNIC previously operated a centralised inproxy, which it discontinued in July 2019 following malware abuse concerns.

Major browsers (Chrome, Firefox, Safari) do not natively support `.bit` resolution. Microsoft and other major browser vendors have declined integration because they cannot enforce brand protection in the namespace (e.g., `google.bit` is held by a squatter).

### TLS Certificate Support

The `ncdns` tool supports TLSA/DANE records for TLS certificate validation anchored in the Namecoin blockchain rather than the CA hierarchy. On Windows, Chromium and Edge support `.bit` TLS validation natively via the Windows certificate store. Tor Browser has experimental support. As of 2025, Namecoin is working on AIA (Authority Information Access) mechanisms for broader mainstream browser compatibility.

---

## Adoption Analysis

### The Squatting Catastrophe

The 2015 Princeton/WEIS study by Kalodner, Carlsten, Ellenbogen, Bonneau, and Narayanan is the most rigorous empirical analysis of Namecoin to date:

- Of ~120,000 registered `.bit` names, only **28** had non-trivial, non-squatted content.
- The market for name transfers was essentially non-existent: 14 to ~250 total lifetime transfers from squatters to legitimate users.
- Most "legitimate" names either redirected to a traditional DNS domain or mirrored content from one.

The root cause: registration cost was fixed at **0.01 NMC**, a negligible sum even when NMC had real market value. This price disincentivised serious use while offering almost no friction to mass squatting. Pretty much every short word, common name, and brand acronym was registered in bulk.

### Browser Integration Failure

The `.bit` TLD is not recognised by any major browser by default. Users must either:
- Install additional software (`ncdns` + local resolver)
- Use a browser extension
- Add a trailing slash or explicit `http://` prefix to prevent the browser treating `.bit` as a search query

This friction is prohibitive for mainstream users. The bar is far higher than for a standard DNS domain.

### OpenNIC Discontinuation (July 2019)

OpenNIC had operated a centralised resolver for `.bit` names, providing the easiest access path. In July 2019, OpenNIC voted 13-2 to drop `.bit` support because `.bit` domains were being used as malware command-and-control infrastructure. The pseudonymous, censorship-resistant nature of `.bit` attracted criminal use. Malware families used Namecoin domains for bulletproof hosting, domain generation algorithms (DGAs), and fast-flux networks.

Namecoin developer Jeremy Rand acknowledged this as the "right decision" given the security issues with centralised inproxies. Source: [Namecoin.org, July 2019](https://www.namecoin.org/2019/07/30/opennic-does-right-thing-shuts-down-centralized-inproxy.html).

### ICANN Takeover Vulnerability

Namecoin's `.bit` TLD faces a structural attack vector: if ICANN decided to operate a `.bit` TLD within the conventional DNS namespace, the vast majority of internet users (whose resolvers follow ICANN) would resolve `.bit` differently from the Namecoin blockchain. This would create consumer confidence chaos and effectively delegitimise the decentralised system through network effects alone. ICANN could seize `.bit` naming authority without touching Namecoin's blockchain at all.

### Developer Community and Funding Challenges

Namecoin suffered from funding difficulties over its lifecycle. Some developers lost engagement; others explored partnerships with ICANN and Google, which contradicted the project's censorship-resistance goals. Development continues as of 2025/2026 (community presence at FOSDEM 2026, 39C3), but the core team is small.

---

## What Namecoin Got Right

1. **First solution to Zooko's Triangle**: Namecoin proved a human-meaningful, secure, decentralised naming system is technically constructible (2011). This directly inspired ENS, Handshake, and others.
2. **Two-phase commit anti-frontrunning**: The `name_new` / `name_firstupdate` pattern (hash commitment then reveal) prevents miners and observers from sniping names during registration. This pattern has been adopted (or independently rediscovered) by successors.
3. **Namespace separation**: The `d/` and `id/` prefix convention cleanly separates DNS from identity, enabling multiple services on one chain without conflict.
4. **Merged mining**: Allowing Bitcoin miners to simultaneously mine Namecoin at zero extra energy cost is elegant. It provides Namecoin with Bitcoin-grade hash rate security without requiring a separate mining ecosystem.
5. **Expiry mechanism**: Forcing name renewal prevents permanent squatting of expired names and ensures the namespace remains potentially claimable. The semi-expiry grace period (name stops resolving but owner retains rights) is a practical user-protection mechanism.
6. **Tor / I2P / onion address support**: Storing `.onion` and `.i2p` addresses in `.bit` records enables human-readable names for anonymous services. This remains a legitimate use case.
7. **Self-sovereign identity (id/ namespace)**: The `id/` namespace is historically significant as the first blockchain-based decentralised identity layer, predating ENS by years.

---

## What Namecoin Got Wrong

1. **Registration price too low and not adjustable**: 0.01 NMC was negligible, with no market-based price discovery. The fixed fee cannot be raised (that would be a soft fork) or lowered (hard fork) without network consensus. No mechanism existed to align registration cost with name value.
2. **No dispute resolution**: There is no mechanism to recover a legitimately valuable name (e.g., `microsoft.bit`) from a squatter. The blockchain is final; the squatter wins by default.
3. **No native browser integration path**: The system was designed with no clear path to mainstream browser support. ICANN non-recognition is a structural barrier, not a temporary one.
4. **Centralised inproxy dependency**: The main user-accessible path (OpenNIC) was centralised and ultimately failed due to abuse. A censorship-resistant naming system should not depend on centralised access infrastructure.
5. **520-byte value limit**: The constraint forces complex workarounds (multi-import) for even moderately complex DNS configurations.
6. **Privacy not built in**: All registrations are public and pseudonymous only. The link between a name and a blockchain address is permanently public. DNS lookups via a local resolver are not visible to ISPs, but the registration link is.
7. **Adopted a new TLD rather than retrofitting existing DNS**: By choosing `.bit` as a new TLD outside ICANN, Namecoin guaranteed that existing web infrastructure would not resolve its names. Handshake partially addresses this by operating at the root zone level rather than a new TLD.

---

## Differentiators vs Successors

| Feature | Namecoin | Handshake | ENS |
|---------|----------|-----------|-----|
| Chain type | Bitcoin PoW fork | Purpose-built PoW | Ethereum smart contracts |
| Merged mining | Yes (with Bitcoin) | No | N/A |
| TLD approach | New `.bit` TLD | Replaces root zone | `.eth` subdomain |
| Name auction | No (first-come first-served + fee) | Vickrey blind auction | Vickrey auction (for short names) |
| Dispute resolution | None | None | None (ICANN-level only) |
| Browser support | Requires extension/software | Requires resolver | Requires extension |
| Value size limit | 520 bytes | Larger (covenants) | Unlimited (smart contract storage) |
| Active use | Near-zero | Low but growing | High (2.5M+ names) |
| Launch year | 2011 | 2020 | 2017 |

---

## Privacy and Censorship Resistance

**Censorship resistance (strong):** Once a name is registered on the Namecoin blockchain, no central authority can revoke it. The blockchain record is permanent unless the owner fails to renew. Merged mining with Bitcoin means the chain is secured by Bitcoin-grade hash power.

**Privacy (weak):** All registrations are on a public blockchain. The name, its associated records (IP addresses, public keys), and the owner's Namecoin address are permanently public. Pseudonymity is offered (address ≠ legal identity) but is the same level as Bitcoin. The `id/` namespace by definition encourages users to attach real identifying information (email, GPG keys, social profiles) to their on-chain identity.

**DNS lookup privacy:** Using a local `ncdns` resolver avoids exposing lookups to a third-party DNS resolver. However, IP-level traffic analysis by an ISP is still possible.

**Malware abuse:** The censorship resistance that protects legitimate users equally protects criminal operators. Namecoin `.bit` domains have been used for malware C2, bulletproof hosting, and DGAs. This was a key factor in OpenNIC dropping `.bit` support. Source: [abuse.ch: .bit Next Generation Bulletproof Hosting](https://abuse.ch/blog/dot-bit-the-next-generation-of-bulletproof-hosting/).

---

## Current Status (2026)

- Development is active but the project is small. Latest stable release: 0.21.0 (January 2021).
- Community presence at FOSDEM 2026 and 39C3 (December 2025).
- Ongoing work on TLS certificate support via AIA for mainstream browser compatibility.
- Market cap: ~$13.2M USD (April 2026).
- Circulating supply: 14,736,400 NMC out of 21M maximum.
- No significant growth in `.bit` domain adoption has been documented since the 2015 study.

---

## Lessons for Decentralised Naming RFP Design

These are the critical design lessons derived from Namecoin's failure to achieve mainstream adoption:

1. **Name pricing must create real anti-squatting friction.** A fixed negligible fee guarantees squatting dominance. Consider auction-based pricing (as Handshake and ENS do for short names), market-based pricing, or burn mechanisms that scale with name value.
2. **Dispute resolution is necessary for brand-name conflicts.** Without a mechanism, any squatter can permanently hold a valuable name. This is a barrier to institutional adoption.
3. **Browser and resolver integration must be designed from day one.** A censorship-resistant naming system is useless if end users cannot access names without installing software. Consider designs that are compatible with existing DNS infrastructure.
4. **Centralised access paths undermine decentralisation claims.** If the primary usability path is a centralised proxy (like OpenNIC's inproxy), the system is only as decentralised as that proxy. The proxy will eventually fail, be censored, or be shut down due to abuse.
5. **Censorship resistance enables abuse.** Any truly censorship-resistant naming system will attract malicious use. Designers must acknowledge this trade-off and consider whether abuse mitigation is possible without compromising the censorship-resistance property.
6. **Privacy requires active design, not accident.** Namecoin's public blockchain provides pseudonymity, not privacy. A naming system that stores IP addresses and public keys on a public ledger exposes significant information.
7. **Choosing a new TLD guarantees a cold-start problem.** New TLDs require user-side software changes. Designs that augment existing DNS infrastructure (root-zone level, DNS bridge, or CCIP-Read patterns) have a lower adoption barrier.

---

## Related Notes

- [[zookos-triangle]]
- [[name-squatting-prevention]]
- [[merged-mining-for-naming]]
- [[name-expiry-renewal-mechanism]]
- [[name-registration-two-phase-commit]]
- [[handshake]]
- [[ens]]

---

## Sources

- [Namecoin.org](https://www.namecoin.org/)
- [Namecoin Wikipedia](https://en.wikipedia.org/wiki/Namecoin)
- [Kalodner et al., WEIS 2015: An Empirical Study of Namecoin](https://econinfosec.org/archive/weis2015/papers/WEIS_2015_kalodner.pdf)
- [Namecoin FAQ](https://www.namecoin.org/docs/faq/)
- [Namecoin Wiki: Domain Name Specification](https://wiki.namecoin.org/index.php?title=Domain_Name_Specification)
- [Namecoin Wiki: Register and Configure .bit Domains](https://wiki.namecoin.org/index.php?title=Register_and_Configure_.bit_Domains)
- [CITP Blog: Empirical Study of Namecoin](https://blog.citp.princeton.edu/2015/05/21/an-empirical-study-of-namecoin-and-lessons-for-decentralized-namespace-design/)
- [OpenNIC shuts down .bit inproxy, July 2019](https://www.namecoin.org/2019/07/30/opennic-does-right-thing-shuts-down-centralized-inproxy.html)
- [Namecoin.org: ncdns](https://www.namecoin.org/docs/ncdns/)
- [abuse.ch: .bit Bulletproof Hosting](https://abuse.ch/blog/dot-bit-the-next-generation-of-bulletproof-hosting/)
- [Zooko's Triangle Wikipedia](https://en.wikipedia.org/wiki/Zooko%27s_triangle)
- [Namecoin.org News (FOSDEM 2026)](https://www.namecoin.org/news/)
- [CoinMarketCap: Namecoin](https://coinmarketcap.com/currencies/namecoin/)
- [BitInfoCharts: Namecoin statistics](https://bitinfocharts.com/namecoin/)
- [Bluish Coder: Namecoin DNS alternative (2011)](https://bluishcoder.co.nz/2011/05/12/namecoin-a-dns-alternative-based-on-bitcoin.html)
- [Namecoin Tor Meeting 2024 summary](https://www.namecoin.org/2024/07/24/tor-2024-gpn-22-monerokon-4-summary.html)
