---
tags: [pattern, root-zone, dns, icann, decentralisation, handshake]
pattern: Root Zone Replacement
seen-in: [[handshake]]
---

# Pattern: Root Zone Replacement

An architectural approach where a decentralised protocol replaces the ICANN-managed DNS root zone entirely, rather than operating as a parallel namespace requiring a custom resolver overlay. Every participant in the network independently validates and replicates the root zone state.

---

## Where It Appears

- [[handshake]] (only production example)

---

## What Is the DNS Root Zone?

The DNS root zone is the authoritative source of truth for all top-level domains. It maps each TLD (.com, .org, .uk, etc.) to the authoritative name servers responsible for that TLD. All DNS resolution ultimately traces back to the root zone.

Under the traditional system, ICANN controls who may register a TLD, at what price, and under what conditions. ICANN can revoke TLDs, refuse to add new ones, or be subject to legal/political pressure. Root servers are operated by 12 organisations under agreements with ICANN.

---

## Handshake's Replacement Model

Handshake replaces the root zone file with a blockchain state. The TLD-to-NS mapping that would normally be in the root zone file is instead stored as covenant UTXO data on the Handshake blockchain.

Every Handshake full node replicates the complete root zone state and serves as an authoritative root server. The consensus protocol ensures all honest nodes agree on the same state. No single operator can modify the root zone unilaterally.

```
Traditional:
  Browser -> root servers (ICANN-operated) -> TLD NS -> authoritative NS -> record

Handshake:
  Browser -> hnsd/hsd (validates blockchain PoW) -> TLD NS (from blockchain) -> authoritative NS -> record
```

- Source: [Handshake FAQ](https://handshake.org/faq/)
- Source: [CoinTelegraph HNS explanation](https://cointelegraph.com/news/what-are-handshake-hns-domains-and-how-do-they-work)

---

## Reserved TLDs (ICANN Compatibility)

To avoid conflicts with the existing DNS ecosystem, Handshake at launch reserved all existing ICANN TLDs (.com, .org, .net, etc.) from auction. These TLDs are blacklisted on the Handshake network; the Handshake resolver falls through to traditional DNS for ICANN TLDs.

Existing TLD operators (ICANN registries) were given a 3-year window to claim their corresponding HNS TLD via the CLAIM covenant.

Known gap: some pending ICANN TLDs (.MUSIC, .WEB, and others) were not included in the initial reservation list due to incomplete data sources at launch.

When a Handshake resolver receives a query for an ICANN TLD, it defers to traditional DNS. When it receives a query for an HNS-only TLD, it resolves via the blockchain.

- Source: [hs-names conflict issue](https://github.com/handshake-org/hs-names/issues/6)
- Source: [Porkbun: What is a Handshake TLD](https://kb.porkbun.com/article/211-what-is-a-handshake-tld-and-how-to-resolve-them)

---

## Resolver Fallthrough Logic

A Handshake-aware resolver (hsd, hnsd, or compatible gateway) uses this logic:

1. Is the TLD in the Handshake blockchain? Resolve via blockchain.
2. Is the TLD a reserved/ICANN TLD? Resolve via traditional DNS (DNSSEC-validated where possible).
3. Unknown/not found: NXDOMAIN.

This means a Handshake resolver is backward compatible with the entire existing DNS namespace, while adding the new HNS namespace on top.

---

## Implications for Naming System Design

| Consideration | Detail |
|---------------|--------|
| Coexistence vs. replacement | Handshake coexists with ICANN DNS; it does not break existing names |
| TLD granularity | A naming system that replaces the root zone allocates at TLD granularity; the TLD owner controls the entire SLD namespace below their TLD |
| SLD management is off-chain | The blockchain stores only TLD records; SLD management is delegated to the TLD owner's DNS infrastructure |
| No browser support | No major browser implements HNS root zone resolution natively; users need software changes |
| Censorship model difference | Traditional DNS can be censored at the registrar, registry, or root level; HNS can only be censored at the consensus level (51% attack) |
| ICANN future TLD conflicts | Any new ICANN TLD that conflicts with an existing HNS TLD creates a resolution ambiguity |

---

## Contrast: Namespace Overlay vs. Root Zone Replacement

| Approach | Examples | Key difference |
|----------|----------|---------------|
| Namespace overlay | [[ens]], [[unstoppable-domains]], [[space-id]] | Adds a new namespace (.eth, .crypto, etc.) alongside traditional DNS; traditional DNS remains the root of trust for non-overlay names |
| Root zone replacement | [[handshake]] | The blockchain IS the root of trust for all names; the system must implement fallthrough for compatibility with existing TLDs |

Root zone replacement is more ambitious and more disruptive. It requires changes at the resolver level (not just the application level) and creates potential conflicts with the existing DNS ecosystem. Namespace overlays are easier to deploy incrementally but perpetuate dependence on the traditional DNS infrastructure for non-overlay names.

---

## RFP Implications

- A Logos naming system can choose between the namespace overlay and root zone replacement approaches
- Root zone replacement requires resolver-level deployment (not just a smart contract) and entails compatibility management with ICANN
- Namespace overlay is lower friction but creates a two-tier name system
- The Handshake experience shows root zone replacement is technically feasible but faces user-experience barriers from lack of browser support
- For a system intended to work within Logos/Waku applications (not general web browsing), the root zone replacement model is more attractive because browser compatibility is less critical

---

## Sources

- [Handshake FAQ](https://handshake.org/faq/)
- [CoinTelegraph HNS explanation](https://cointelegraph.com/news/what-are-handshake-hns-domains-and-how-do-they-work)
- [Porkbun: Handshake TLD resolution](https://kb.porkbun.com/article/211-what-is-a-handshake-tld-and-how-to-resolve-them)
- [hs-names conflict issue](https://github.com/handshake-org/hs-names/issues/6)
- [Namebase tutorial: Difference from traditional domains](https://www.namebase.io/blog/tutorial-5-difference-between-handshake-domains-and-traditional-domains/)
