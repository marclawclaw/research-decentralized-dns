---
tags: [pattern, governance, dao, ethereum]
related_projects: [ens]
---

# Pattern: ENS DAO Governance

## Summary

The ENS DAO governs the ENS protocol and its treasury via token-weighted voting using the ENS token. It was established in November 2021 alongside the ENS token airdrop. The DAO sets fee parameters, approves treasury expenditure, manages the protocol endowment, and ratifies ENSIPs (improvement proposals). It is one of the more active and financially substantial protocol DAOs on Ethereum.

## Structure

- **ENS token**: 100 million total supply; ERC-20 governance token with voting rights. ~38 million circulating as of early 2026.
- **Voting**: token-weighted; 1 ENS = 1 vote. Tokens must be delegated before a vote commences to count. Gasless voting was introduced in 2024 (via off-chain signing, Snapshot-style), reducing participation barriers. Source: [ENS DAO Basics: Gasless Voting](https://discuss.ens.domains/t/gasless-voting-is-live-for-ens/19070)
- **Proposal process**: Social proposals (Snapshot off-chain) can precede Executable proposals (on-chain, binding). Executable proposals require a 7-day voting period.
- **Stewards**: elected working group leads manage day-to-day operations in domains (Meta-Governance, ENS Ecosystem, Public Goods).
- **Tally / Agora**: on-chain proposal tracking platforms used by ENS DAO.

## Treasury and endowment

Revenue sources:
1. `.eth` name registration fees.
2. `.eth` name renewal fees.
3. Temporary premium fees (Dutch auction on expired names).
4. Endowment DeFi yields.

| Revenue item | 2024 full year | Q1 2025 |
|-------------|---------------|---------|
| Total DAO revenue | $28.77M | $4.94M |
| Registration + renewal fees (Q1 2025) | n/a | $3.47M |
| Premium fees (Q1 2025) | n/a | $585K |

Source: [ENS DAO Revenue Reports](https://discuss.ens.domains/t/ens-revenue-reports/20577/2)

The **Endowment** was initialised in March 2023 with 16,000 ETH, managed by Karpatkey DAO. It invests in DeFi protocols (staking, LP positions) to generate a sustainable yield stream independent of registration demand. The third tranche expansion proposal suggested a further 5,000 ETH. Since inception, the Endowment has generated approximately $2.92M in DeFi net results.

Source: [ENS Endowment FAQ](https://basics.ensdao.org/endowment); [CoinDesk: Karpatkey selection](https://www.coindesk.com/tech/2022/11/23/ethereum-name-service-selects-karpatkey-dao-as-endaoment-fund-manager)

## Key governance decisions

- **Fee structure**: the DAO sets annual registration and renewal fees. A 2022 Vitalik Buterin post suggested demand-based recurring fees; this was not adopted. Short-name premiums (3 and 4 char) were retained.
- **ENSIP ratification**: the DAO votes on ENSIPs that affect protocol behaviour (e.g. ENSIP-15 normalisation, ENSIP-17 gasless DNS).
- **Namechain reversal (February 2026)**: ENS Labs announced abandoning Namechain. This was a strategic and technical decision by ENS Labs, ratified by community consensus rather than a formal on-chain vote, driven by the dramatic reduction in Ethereum gas costs.

## Limitations

- **Voter concentration**: token-weighted voting gives large holders (VCs, early contributors) disproportionate influence. Academic analysis of DAO voting power distributions confirms this as a structural concern across DAOs generally. Source: [ScienceDirect DAO voting power](https://www.sciencedirect.com/article/pii/S2096720924000216)
- **Low retail participation**: despite gasless voting, most token holders do not vote or actively delegate. The distribution monitor tracks this. Source: [ENS Distribution Monitor](https://discuss.ens.domains/t/ens-distribution-monitor/21527)
- **Labs vs DAO tension**: ENS Labs (the core development team) holds significant soft power through its technical expertise and roadmap control. Major decisions like the Namechain reversal originate from ENS Labs, with the DAO effectively ratifying rather than initiating.

## RFP relevance

ENS governance is a mature reference model for protocol DAO governance in a naming context:

1. **Revenue capture**: annual fees that flow to a DAO treasury, combined with a DeFi endowment, provide a sustainable model for protocol maintenance.
2. **Parameter governance**: fee tiers and namespace rules are governable without protocol upgrades.
3. **Working group structure**: stewards reduce governance overhead for operational decisions while keeping major decisions in the hands of token holders.
4. **Governance attack surface**: token-weighted voting is vulnerable to accumulation by well-capitalised adversaries. Any Logos naming RFP should consider governance models that prioritise resistance to plutocratic capture.

## Sources

- [ENS DAO Documentation](https://docs.ens.domains/dao/) — accessed 2026-04-08
- [ENS DAO Basics: Voting](https://basics.ensdao.org/voting) — accessed 2026-04-08
- [ENS DAO Basics: Protocol Revenue](https://basics.ensdao.org/protocol-revenue) — accessed 2026-04-08
- [ENS DAO Revenue Reports](https://discuss.ens.domains/t/ens-revenue-reports/20577/2) — accessed 2026-04-08
- [ENS DAO Governance Forum: Gasless Voting](https://discuss.ens.domains/t/gasless-voting-is-live-for-ens/19070) — accessed 2026-04-08
- [ENS DAO Forum: Distribution Monitor](https://discuss.ens.domains/t/ens-distribution-monitor/21527) — accessed 2026-04-08
- [CoinDesk: ENS Karpatkey Endowment](https://www.coindesk.com/tech/2022/11/23/ethereum-name-service-selects-karpatkey-dao-as-endaoment-fund-manager) — accessed 2026-04-08
- [Vitalik Buterin: ENS demand-based fees](https://vitalik.ca/general/2022/09/09/ens.html) — accessed 2026-04-08
- [ScienceDirect: DAO voting power](https://www.sciencedirect.com/article/pii/S2096720924000216) — accessed 2026-04-08
