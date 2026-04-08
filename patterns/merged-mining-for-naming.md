# Pattern: Merged Mining for Naming

**Category:** Consensus / security mechanism
**Observed in:** [[namecoin]] (primary example), Elastos, Myriad
**Relevance to RFP:** Historical/architectural. Relevant as a comparison point for naming-chain security models.

---

## Definition

Merged mining (also called auxiliary proof-of-work, AuxPoW) allows miners to submit valid proof-of-work solutions simultaneously to two or more blockchains using the same hash computation. One chain is designated the "parent" (the higher-value chain, e.g. Bitcoin), and the other is the "auxiliary" chain (e.g. Namecoin). The same SHA-256 hash that satisfies Bitcoin's difficulty can also satisfy Namecoin's (lower) difficulty target, at no additional energy cost to the miner.

---

## How It Works in Namecoin

Namecoin activated merged mining at block 19,200 (approximately mid-2011). The mechanism:

1. **Bitcoin miner includes Namecoin data**: A Bitcoin miner includes a reference (Merkle root) to a Namecoin block candidate in the Bitcoin coinbase transaction.
2. **Same hash, two uses**: When the miner finds a hash satisfying Bitcoin's difficulty, they check if it also satisfies Namecoin's (lower) difficulty. If yes, a valid Namecoin block is submitted alongside the Bitcoin block.
3. **Separate blockchains**: Bitcoin data does not enter the Namecoin chain and vice versa. They remain structurally separate. The miner simply runs both clients and submits to both networks from the same hashing effort.
4. **Parent chain flexibility**: Namecoin's merged mining can use any Hashcash-SHA-256d chain as the parent, not only Bitcoin. Bitcoin Cash (BCH) has also been used.

Source: [Namecoin Wiki: Merged Mining](https://github.com/namecoin/wiki/blob/master/Merged-Mining.mediawiki), [Namecoin FAQ](https://www.namecoin.org/docs/faq/)

---

## Security Properties

### Benefits

- **Bitcoin-grade hash rate at no marginal cost**: Namecoin's security is directly tied to the fraction of Bitcoin's hash rate that chooses to merge-mine. Since merge-mining is free for miners (same energy expenditure), adoption is high among large Bitcoin mining pools.
- **No independent mining ecosystem required**: Namecoin does not need to attract dedicated miners or maintain a separate token price high enough to fund mining rewards.
- **51% attack cost**: Attacking Namecoin would require outpacing a significant portion of Bitcoin's mining power, making attacks economically infeasible.

### Limitations

- **Miner control risk**: The security is only as distributed as the merged-mining participants. If a single large Bitcoin mining pool dominates Namecoin merge-mining, they could theoretically reorganise the Namecoin chain. In practice, this has not been exploited, but the risk is structural.
- **Mining pool centralisation inherited from Bitcoin**: Namecoin inherits Bitcoin's mining centralisation tendencies. As of 2024/2025, a small number of pools control the majority of Bitcoin hash rate, and by extension much of Namecoin's security.
- **No incentive alignment for naming quality**: Miners are economically incentivised to maximise block rewards, not to curate the quality of names or prevent squatting. Merged mining provides Sybil resistance but not governance.
- **Dependent on Bitcoin ecosystem health**: If Bitcoin's proof-of-work model is eventually abandoned or significantly changed, Namecoin's security model collapses.

---

## Comparison: Handshake vs Namecoin on Consensus

| Feature | Namecoin (merged mining) | Handshake (independent PoW) |
|---------|-------------------------|----------------------------|
| Hash algorithm | SHA-256 (Bitcoin-compatible) | blake2b + sha3 (custom) |
| Security source | Bitcoin hash rate (indirect) | Dedicated HNS miners |
| Miner incentive | NMC block reward (+ merge-mining benefit) | HNS block reward |
| Dependence on Bitcoin | Yes | No |
| 51% attack cost | Very high (Bitcoin-scale) | Lower (dedicated HNS hash rate) |

Handshake deliberately chose a custom PoW algorithm (not SHA-256) precisely to avoid merged mining and ensure Handshake miners are independent of Bitcoin's mining landscape.

---

## Relevance for Decentralised Naming RFP Design

1. **Merged mining is a viable security bootstrap for a new naming chain.** If the naming system is implemented on a PoW chain, merged mining with Bitcoin can provide strong security without needing an independent mining economy.

2. **Merged mining does not solve adoption.** Namecoin proves that even strong cryptographic security (via Bitcoin hash rate) cannot compensate for economic and UX failures. Security is necessary but not sufficient.

3. **Non-PoW architectures avoid the merged mining question entirely.** Smart-contract systems (ENS on Ethereum), DHT-based systems (GNS, IPNS), and rollup-based systems have their own security models that do not depend on PoW mining. For a naming RFP focused on DHT or smart-contract architectures, merged mining is not directly relevant.

4. **Mutualist security relationship with a parent chain introduces dependency risk.** A naming system that derives its security from Bitcoin inherits Bitcoin's governance, upgrade, and centralisation risks.

---

## Related Notes

- [[namecoin]]
- [[handshake]]
- [[name-squatting-prevention]]
- [[name-expiry-renewal-mechanism]]

---

## Sources

- [Namecoin Wiki: Merged Mining](https://github.com/namecoin/wiki/blob/master/Merged-Mining.mediawiki)
- [Namecoin FAQ](https://www.namecoin.org/docs/faq/)
- [BitcoinWiki: Namecoin](https://bitcoinwiki.org/wiki/namecoin)
- [Namecoin Wikipedia](https://en.wikipedia.org/wiki/Namecoin)
