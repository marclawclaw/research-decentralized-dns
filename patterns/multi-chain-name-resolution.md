# Pattern: Multi-Chain Name Resolution

## Definition

Multi-chain name resolution is the ability for a single human-readable name (e.g. `alice.bnb`) to resolve to a correct wallet address or record on any of multiple blockchains, without the user or application specifying the target chain explicitly.

---

## Why It Is a Design Problem

Each blockchain has its own address format and address space. A user holding assets on BNB Chain, Arbitrum, and Solana has three distinct addresses. Without a naming layer, senders must know which chain-specific address to use, and must correctly match the address to the chain. Errors cause permanent fund loss.

Multi-chain resolution adds a mapping layer: `name + chain` or `name + asset type` points to the correct address.

---

## Production Approaches (as at 2025)

Three architecturally distinct approaches are deployed in production:

### Approach 1: Single canonical chain, per-coin-type records (ENS)

**How it works:** All name records are stored on Ethereum mainnet. ENSIP-9 (EIP-2304) extended the ENS resolver to store addresses for multiple coin types (identified by SLIP-0044 coin type integers or EVM chain ID derivations). Resolution always starts on Ethereum L1. ENSv2 / Namechain adds CCIP-Read: a resolver gateway contract that can verify off-chain or L2 data via a signed response returned to the L1 verifier.

**Canonical chain:** Ethereum mainnet (L1)

**Uniqueness guarantee:** on-chain (Ethereum L1 registry is the single source of truth)

**Latency:** requires Ethereum L1 query; with CCIP-Read, also a gateway HTTP round-trip

**Cost:** Ethereum L1 gas for all registrations and updates, regardless of which chain the resolved address is on

**Censorship resistance:** high; Ethereum L1 is highly decentralised

**Example:** ENS `.eth` — a single `.eth` name stores `.eth` (Ethereum), `.btc` (Bitcoin), `.sol` (Solana) addresses in the same L1 resolver contract

Source: [[ens]], [ENSIP-9](https://docs.ens.domains/ensip/9/), [ENS multichain docs](https://docs.ens.domains/web/multichain/)

---

### Approach 2: Per-chain deployment with cross-chain uniqueness oracle (SPACE ID)

**How it works:** A separate registrar smart contract ("Jedi") is deployed on each supported chain. Name registration and resolution for a given TLD are performed on that chain's Jedi contract. Cross-chain uniqueness is enforced by an off-chain oracle ("Yoda") that aggregates registration events from all chains and issues authentication signatures required by each Jedi before finalising any new registration.

**Canonical chain:** none; each chain's Jedi is authoritative for names on that chain

**Uniqueness guarantee:** off-chain oracle (Yoda); centralisation risk documented as [NOT FULLY DECENTRALISED]

**Latency:** native to each chain (e.g. BNB Chain speed for `.bnb` queries)

**Cost:** native gas on each chain; no cross-chain gas

**Censorship resistance:** per-chain on-chain records are censorship-resistant; the Yoda oracle layer introduces a centralised dependency for new registrations

**TLD namespace:** per-chain TLDs (`.bnb` on BNB Chain, `.arb` on Arbitrum); same name string is reserved across all chains by Yoda

**Example:** [[space-id]] — `.bnb` on BNB Chain, `.arb` on Arbitrum, plus 20+ community TLDs

Source: [[space-id]], [SPACE ID docs](https://docs.space.id), [ChainTech SPACE ID](https://www.chaintech.network/blog/what-is-space-id-a-multi-chain-name-service/)

---

### Approach 3: Purpose-built canonical chain with multi-chain record references (D3 / Doma Protocol)

**How it works:** A purpose-built blockchain (Doma Protocol) serves as the canonical registry. Domain ownership is mirrored between a traditional DNS registrar and the Doma blockchain. Name records are chain-agnostic at the ENS resolver level: a single name can reference addresses on Ethereum, L2s, Bitcoin, Solana, and any other chain. When an NFT representing a domain is transferred on Doma, both the DNS registrar record and the on-chain record are updated atomically.

**Canonical chain:** Doma Protocol (purpose-built)

**Uniqueness guarantee:** on the Doma canonical chain

**Latency:** requires Doma chain query plus any cross-chain bridge for non-Doma address records

**Cost:** Doma chain gas

**Censorship resistance:** depends on the decentralisation of the Doma validator set (early stage; not fully characterised)

**ICANN compatibility:** yes, traditional DNS and on-chain ownership are synchronised

**Example:** [[d3-doma]] — bridges 362M+ existing DNS domains to Web3 identity

Source: [[d3-doma]], [D3 Series A](https://www.prnewswire.com/news-releases/d3-raises-25m-series-a-led-by-paradigm-announces-the-first-blockchain-for-internets-362m-domain-names-302362780.html)

---

## Comparison Table

| Property | ENS (single canonical chain) | SPACE ID (per-chain + oracle) | D3 / Doma (purpose-built chain) |
|----------|------------------------------|-------------------------------|----------------------------------|
| Canonical chain | Ethereum L1 | None (oracle-coordinated) | Doma Protocol |
| Uniqueness guarantee | On-chain (L1) | Off-chain oracle (Yoda) | On-chain (Doma) |
| Per-chain latency | High (L1 query required) | Low (native chain) | Medium (Doma query) |
| ICANN compatibility | No | No | Yes |
| Permissionless TLDs | No | Yes (SPACE ID 3.0) | TBD |
| Token required | No ($ETH for fees) | Yes ($ID for TLD launch) | TBD |
| Query privacy | None | None | None |
| Decentralisation of uniqueness layer | High (Ethereum L1) | Unclear (Yoda oracle) | Early stage |
| Production scale | 2.5M+ active names | 6.7M+ registered names | Testnet |
| Non-EVM support | Yes (SLIP-0044 coin types) | Yes (.sol, .sei, .inj) | Yes (ENS resolver records) |

---

## Key Design Tensions

### 1. Uniqueness vs decentralisation

Enforcing that the same name cannot be registered on two different chains requires a coordination mechanism. Options:

- **On a single canonical chain** (ENS approach): strong uniqueness guarantee; all resolution traffic goes through one chain; that chain's liveness and decentralisation level bound the system
- **Off-chain oracle** (SPACE ID approach): lower latency and cost per chain; introduces trust in the oracle operator; oracle downtime can block new registrations but does not break existing resolution
- **Purpose-built chain** (D3 approach): dedicated chain optimised for the naming use case; security depends on validator set; adds another chain to the ecosystem

### 2. Human-readable name vs chain-specific address

A name like `alice` must eventually resolve to an address on a specific chain. Approaches:

- **Explicit coin type**: the resolver stores separate records for each chain type; the caller specifies which chain type to resolve (ENS ENSIP-9 model)
- **Contextual resolution**: the SDK or wallet infers the target chain from the transaction context and queries the appropriate record (common in practice)
- **Universal address**: no current system achieves a single address across all chains; cross-chain bridges and smart contract wallets address this at the execution layer, not the naming layer

### 3. Per-chain TLDs vs universal TLDs

SPACE ID's model ties TLDs to chains (`.bnb` is BNB Chain; `.arb` is Arbitrum). This makes the chain origin explicit but fragments the namespace. ENS uses a single `.eth` TLD across all chains, storing chain-specific addresses as record fields. A universal TLD is conceptually simpler for users but requires a canonical chain.

---

## Implications for a DHT-Based Naming RFP

A DHT-based naming system without a canonical chain faces unique challenges for multi-chain resolution:

1. **No canonical chain** means no obvious location for a global registry or uniqueness check. The DHT itself must act as the coordinator.
2. **Record freshness**: DHT records expire and must be re-published; address records on multiple chains can become stale independently.
3. **Cross-chain uniqueness**: a DHT-native approach must either tolerate name conflicts across chains (accepting fragmentation) or implement a distributed consensus mechanism within the DHT for uniqueness — an unsolved problem in production systems.
4. **Opportunity**: a DHT-based system could store multi-chain address records natively (similar to ENS ENSIP-9 records) without requiring any blockchain, if record updates are authenticated by the name owner's keypair.

See also: [[gnu-name-system]] for a production DHT-based naming system (no multi-chain address records), [[ipns]] for DHT-based mutable naming over libp2p.

---

## Related Notes

- [[space-id]] — largest production deployment of per-chain approach
- [[ens]] — largest production deployment of single-canonical-chain approach
- [[d3-doma]] — purpose-built chain approach
- [[gnu-name-system]] — DHT-based; no multi-chain address record support
- [[ipns]] — DHT-based mutable naming; no human-readable layer

---

## Sources

- [ENSIP-9 multichain resolution](https://docs.ens.domains/ensip/9/)
- [ENS multichain docs](https://docs.ens.domains/web/multichain/)
- [EIP-2304](https://eips.ethereum.org/EIPS/eip-2304)
- [SPACE ID docs](https://docs.space.id)
- [ChainTech Network SPACE ID architecture](https://www.chaintech.network/blog/what-is-space-id-a-multi-chain-name-service/)
- [D3 Series A press release](https://www.prnewswire.com/news-releases/d3-raises-25m-series-a-led-by-paradigm-announces-the-first-blockchain-for-internets-362m-domain-names-302362780.html)
- [BNB Chain SPACE ID offering](https://www.bnbchain.org/en/blog/spaceid-offering-users-a-bnb-multi-chain-identity)
