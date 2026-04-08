# Pattern: Name Squatting Prevention

**Category:** Economic mechanism / namespace governance
**Observed in:** [[namecoin]] (failure case), [[handshake]] (partial mitigation), ENS (auction mechanism)
**Relevance to RFP:** Critical. Squatting dominance is the primary reason Namecoin failed.

---

## Definition

Name squatting in a decentralised naming system occurs when a registrant acquires names they have no intention of actively using, either to sell at a premium later or to hold indefinitely. Because decentralised systems have no central authority to enforce trademark or brand protection, squatting is structurally incentivised whenever registration costs are low and resale profits are possible.

---

## The Namecoin Case Study

The 2015 Princeton/WEIS empirical study (Kalodner, Carlsten, Ellenbogen, Bonneau, Narayanan) provides the most quantified evidence of squatting failure:

| Metric | Value |
|--------|-------|
| Total registered `.bit` names | ~120,000 |
| Names with non-trivial, non-squatted content | 28 |
| Squatter-to-legitimate ratio | >99.98% squatted or inactive |
| Lifetime name transfers (squatter to legitimate user) | 14 (lower bound) to ~250 (upper bound) |

Source: [Kalodner et al. WEIS 2015](https://econinfosec.org/archive/weis2015/papers/WEIS_2015_kalodner.pdf)

**Root cause in Namecoin:** Registration was priced at 0.01 NMC with no market-based adjustment. This fee was so low that mass squatting required negligible capital. Pretty much every short word, common name, and brand acronym was registered within the first years of operation.

**Structural feedback loop:** Squatting made the namespace useless for legitimate users. Legitimate users stopped registering. This reinforced the dominance of squatters. The namespace became a graveyard.

---

## Anti-Squatting Mechanisms Observed Across Systems

### 1. Fixed Fee (Namecoin approach, FAILED)

- **Mechanism:** A fixed low fee (0.01 NMC) is charged per registration. Renewal is free.
- **Outcome:** Does not prevent squatting. Fee is too low to deter mass registration; no marginal cost for holding names.
- **Verdict:** Insufficient. Low fixed fees are a squatting subsidy, not a deterrent.

### 2. Vickrey (Second-Price Blind) Auction (Handshake, ENS for short names)

- **Mechanism:** Would-be registrants submit sealed bids. The highest bidder wins and pays the second-highest bid price. All bids are locked during the auction period.
- **Properties:**
  - Price discovery reflects actual demand, not fixed cost.
  - Squatters must commit real capital at bid time (locked for the auction duration).
  - The system is incentive-compatible: truthful bidding is the dominant strategy.
- **Handshake implementation:** Blind Vickrey auction; proceeds are burned (deflationary). Auction period is ~720 blocks (~5 days for reveal). Source: [Handshake documentation](https://handshake.org/).
- **ENS implementation:** Vickrey auction used for short (≤6 character) `.eth` names historically; replaced by a simpler annual fee model for most names in ENS 2.0.
- **Verdict:** Effective at price discovery and capital commitment. Does not prevent squatting of valuable names if a squatter has deep pockets; but raises the cost dramatically.

### 3. Annual Renewal Fees (ENS, Unstoppable Domains contrast)

- **Mechanism:** Names require an annual renewal payment. Failure to renew releases the name.
- **Properties:**
  - Creates ongoing cost for holding names, making mass squatting economically unsustainable over time.
  - Squatters must continuously evaluate whether holding a name is worth the renewal cost.
  - ENS charges ~$5/year for 5+ character `.eth` names; shorter names cost more.
- **Contrast:** Unstoppable Domains charges a one-time fee, no renewals. This eliminates squatting friction entirely.
- **Verdict:** Annual fees are a meaningful deterrent for long-term squatting but not for short-term speculation.

### 4. Expiry with Grace Period (Namecoin, Handshake)

- **Mechanism:** Names expire after a fixed block count (Namecoin: 36,000 blocks, ~200 days) unless renewed. A grace period allows the owner to renew before the name opens to re-registration.
- **Properties:**
  - Prevents permanent accumulation of unused names.
  - Squatters who do not renew release names back to the pool.
  - Grace period (Namecoin: semi-expiry at 31,968 blocks, full expiry at 36,000) protects owners from accidental loss.
- **Limitation in Namecoin:** Squatters simply renewed cheaply (renewal was free, only transaction fee). Expiry alone without renewal costs does not prevent squatting.
- **Verdict:** Necessary but not sufficient without meaningful renewal costs.

### 5. Proof-of-Use Requirements

- **Mechanism:** Names that fail to resolve to active content are flagged or eventually revocable.
- **Status:** Proposed in various discussions but not implemented in any major production system as of 2026.
- **Challenge:** Defining "active use" is subjective and introduces centralised judgement into the system.
- **Verdict:** Theoretically attractive but difficult to implement in a trustless, censorship-resistant way.

### 6. Name Burning on Non-Renewal

- **Mechanism:** Unreleased names accumulate a "dust" fee paid to miners or burned on renewal failure.
- **Status:** Proposed for Namecoin in forum discussions; not implemented.
- **Verdict:** Not yet proven in production.

---

## Key Design Implications for a Decentralised Naming RFP

1. **Fixed low fees guarantee squatting dominance.** Any system with a negligible, fixed registration fee will experience Namecoin-style squatting. This is not a hypothetical; it is empirically documented.

2. **Auction-based initial registration creates price discovery.** The Handshake Vickrey auction is the most principled solution. It forces squatters to commit capital proportional to the name's expected value.

3. **Annual renewal fees discourage long-term speculative holding.** A fee that scales with name length or demand is the ENS approach; it creates ongoing cost for holding unused names.

4. **Expiry is necessary but not sufficient.** Names must expire without renewal, but expiry alone (with free renewal) does not deter squatters.

5. **Brand-name conflicts require a dispute resolution mechanism.** Purely first-come-first-served registration guarantees that every major brand name will be squatted. Designers should either: (a) accept this and not target mainstream adoption, or (b) implement a dispute resolution process (UDRP-equivalent) or a brand-claim reservation mechanism.

6. **The namespace cold-start is a chicken-and-egg problem.** A namespace dominated by squatters repels legitimate users. Legitimate user absence reduces name value. The system needs critical mass of legitimate users before squatters make the space economically attractive, or it needs to front-load anti-squatting mechanisms.

---

## Related Notes

- [[namecoin]]
- [[handshake]]
- [[merged-mining-for-naming]]
- [[name-expiry-renewal-mechanism]]

---

## Sources

- [Kalodner et al. WEIS 2015](https://econinfosec.org/archive/weis2015/papers/WEIS_2015_kalodner.pdf)
- [CITP Blog: Empirical Study of Namecoin](https://blog.citp.princeton.edu/2015/05/21/an-empirical-study-of-namecoin-and-lessons-for-decentralized-namespace-design/)
- [Namecoin Forum: Registering is too cheap](https://forum.namecoin.org/viewtopic.php?f=2&t=360)
- [Namecoin Forum: Name Squatting once again](https://forum.namecoin.org/viewtopic.php?t=2387&start=10)
- [Typosquatting 3.0: Characterizing Squatting in Blockchain Naming Systems (eCrime 2024)](https://arxiv.org/abs/2411.00352)
- [Handshake documentation](https://handshake.org/)
