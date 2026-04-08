---
tags: [pattern, naming, governance, economics]
related_projects: [ens]
---

# Pattern: ENS Name Expiry and Renewal

## Summary

ENS `.eth` names are leased, not owned permanently. Annual fees must be paid to maintain registration. A structured lifecycle governs expiry: a grace period protects current owners; a Dutch auction on re-registration discourages speculation on expired names. This model contrasts with Unstoppable Domains and Handshake, which offer perpetual or near-permanent ownership.

## Name lifecycle

```
[Active] --expires--> [Grace Period: 90 days] --grace ends--> [Premium Auction: 21 days] --auction ends--> [Open Registration]
```

### Active period

- Owner controls the name, sets records, can transfer or sell.
- Renewals can be made at any time (prepaying future years).
- Gas cost for renewal is the annual fee (at current ETH/USD rate) plus a small transaction fee.

### Grace period (90 days post-expiry)

- Name is expired but the original owner retains exclusive renewal rights.
- Records remain resolvable during the grace period (TTL-dependent).
- Transfers and secondary market trading are blocked.
- Name Wrapper contract adds further restrictions: resolver cannot be edited, subnames cannot be created.

Source: [ENS Grace Period Support](https://support.ens.domains/en/articles/8046905-grace-period)

### Temporary premium (Dutch auction, 21 days)

- After the grace period, the name enters a decaying-price premium auction.
- Initial price: $2,000 above the regular annual fee.
- Price decays linearly to the regular annual fee over 21 days.
- Anyone can register by paying the current premium price.
- Purpose: discourages bots from instantly registering desirable expired names; rewards those who actually want to use the name over speculators paying the highest premium.

Source: [ENS Name Lifecycle](https://support.ens.domains/en/articles/8046877-eth-name-lifecycle)

### Open re-registration

After the 21-day auction, the name is available for registration at the standard annual fee with no premium.

## Security concern: name takeover

Academic research ([arXiv: ENS Good, Bad and Ugly](https://arxiv.org/pdf/2104.05185)) identifies name expiry as an attack vector:

- Attackers monitor expiring names and re-register them for profit or phishing.
- Previous owner's wallet associations, reputation, or ENS-linked credentials are effectively hijacked.
- ENS Labs sends email reminders but no on-chain notification mechanism exists.
- The severity is proportional to the name's established reputation or linked services.

Mitigation in use: ENS Labs email notifications; the decaying auction reduces profitability of immediate re-registration by bots.

## Economic rationale for annual fees

Per ENS DAO documentation and Vitalik Buterin's analysis ([vitalik.ca](https://vitalik.ca/general/2022/09/09/ens.html)):

- The primary purpose is an anti-squatting mechanism: names that no one actively maintains eventually return to the pool.
- Secondary purpose: protocol sustainability. Registration and renewal fees are the main revenue source for the ENS DAO treasury.
- The DAO debated demand-based recurring fees (higher fees for popular names) but did not adopt them as of April 2026.

## Comparison to perpetual ownership models

| Model | Example | Benefit | Risk |
|-------|---------|---------|------|
| Annual lease | ENS | Names return to pool; anti-squatting | Owner must track renewals; risk of accidental lapse |
| One-time purchase | Unstoppable Domains | No renewal friction | Namespace permanently squatted by early registrants |
| Blockchain-native expiry | Namecoin (~200 days) | Short expiry clears namespace quickly | Frequent maintenance burden on active users |

## RFP relevance

Name expiry is a governance and UX decision with significant security implications:

1. **Lease vs perpetual**: lease models reduce long-term namespace pollution but create renewal UX friction and lapse risk.
2. **Grace period length**: 90 days is a reasonable default; shorter periods increase takeover risk; longer periods delay namespace recycling.
3. **Auction mechanics**: the decaying premium auction is a well-tested anti-sniping mechanism; worth considering for any system where desirable names may expire.
4. **Notification infrastructure**: purely on-chain renewal is a UX failure; supporting off-chain reminders is operationally important.

## Sources

- [ENS Support: Grace Period](https://support.ens.domains/en/articles/8046905-grace-period) — accessed 2026-04-08
- [ENS Support: Name Lifecycle](https://support.ens.domains/en/articles/8046877-eth-name-lifecycle) — accessed 2026-04-08
- [ENS Support: Renewals](https://support.ens.domains/en/articles/7900608-renewals) — accessed 2026-04-08
- [arXiv: ENS Good, Bad and Ugly](https://arxiv.org/pdf/2104.05185) — accessed 2026-04-08
- [Vitalik Buterin: Demand-based fees](https://vitalik.ca/general/2022/09/09/ens.html) — accessed 2026-04-08
