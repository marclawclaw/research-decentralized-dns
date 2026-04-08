# Pattern: Name Expiry and Renewal Mechanism

**Category:** Namespace lifecycle management
**Observed in:** [[namecoin]] (primary example), ENS (annual renewal), Handshake (renewal covenants)
**Relevance to RFP:** High. Expiry design directly affects squatting dynamics, user experience, and namespace liveness.

---

## Definition

A name expiry mechanism forces name owners to periodically renew their registration or lose ownership. The name then becomes available for others to register. Expiry is a tool for:

1. Preventing permanent accumulation of unused names (anti-squatting)
2. Returning expired names to the pool for legitimate use
3. Creating a namespace "garbage collection" process
4. (Potentially) generating ongoing protocol revenue through renewal fees

---

## Namecoin's Two-Stage Expiry Model

Namecoin implements a two-stage expiry process designed to balance namespace freshness with owner protection against accidental loss.

**Stage 1: Semi-Expiry (warning state)**
- Triggered after **31,968 blocks** (~222 days) without a `name_update` or renewal transaction.
- Effect: the name **stops resolving** for end users (DNS lookups return nothing).
- Ownership: the registrant **retains exclusive rights** to renew.
- Duration of semi-expiry window: **4,032 blocks** (~28 days).

**Stage 2: Full Expiry (open for re-registration)**
- Triggered after **36,000 blocks** (~250 days total) without renewal.
- Effect: the name becomes **available for re-registration** by anyone.
- There is no grace period after full expiry; first-come-first-served re-registration applies.

**Renewal cost:** Only the standard blockchain transaction fee. There is no additional registration fee for renewals. This is a key design flaw: see [[name-squatting-prevention]] for analysis of how free renewal subsidises squatting.

Source: [Namecoin FAQ](https://www.namecoin.org/docs/faq/), [Namecoin Wikipedia](https://en.wikipedia.org/wiki/Namecoin)

---

## Comparison Across Systems

| System | Expiry period | Renewal cost | Grace period | Re-registration |
|--------|--------------|--------------|--------------|-----------------|
| Namecoin | ~250 days (36,000 blocks) | Free (tx fee only) | ~28 days (semi-expiry) | First-come-first-served |
| ENS | Annual (365 days) | ~$5/yr (5+ char names) | 90-day grace period | First-come-first-served after grace |
| Handshake | Renewal via covenant tx | Tx fee only | Depends on covenant | Auction on expiry |
| EmerDNS | Up to 30 years | Tx fee | Not specified | First-come-first-served |
| IPNS | 48 hours (re-publish) | None (no chain fee) | None | Instant (key rotation) |

---

## Design Tradeoffs

### Short Expiry (e.g. 250 days, as in Namecoin)

**Pros:**
- Namespace stays fresh; abandoned names return to pool quickly.
- Reduces long-term squatting (squatters must repeatedly pay renewal costs, if any).
- Incentivises active use.

**Cons:**
- Increases operational burden on legitimate owners (must monitor and renew).
- Risk of accidental loss if renewal infrastructure fails (e.g. service outage, lost keys).
- Names linked to long-lived services (e.g. a 10-year project) require reliable renewal automation.

### Long Expiry or No Expiry (e.g. Unstoppable Domains)

**Pros:**
- Zero maintenance burden for owners.
- No risk of accidental expiry.
- Simpler UX.

**Cons:**
- No mechanism to reclaim abandoned or lost names.
- Squatters hold names indefinitely at zero ongoing cost.
- Namespace degrades over time as abandoned names accumulate.

### Semi-Expiry (Namecoin's grace period approach)

**Pros:**
- Protects active owners from losing names due to temporary inattention.
- Stops-resolving warning gives owners a signal before permanent loss.
- More user-friendly than immediate hard expiry.

**Cons:**
- Adds complexity to the expiry state machine (two states instead of one).
- Resolution clients must correctly handle the semi-expired state.

---

## Interaction with Squatting

Expiry alone does not prevent squatting if renewal is free or cheap. In Namecoin:
- Squatters registered names in bulk at 0.01 NMC each.
- Renewal cost zero additional NMC (only transaction fees, which were negligible).
- Squatters simply renewed names indefinitely with near-zero marginal cost.
- Expiry had essentially no anti-squatting effect in practice.

**Lesson:** Expiry must be paired with meaningful renewal fees or auction-based mechanisms to deter speculative holding. Expiry without renewal cost is a feature that helps legitimate users (namespace garbage collection) but does not hurt squatters.

---

## Automation Requirements

For any production naming system, name owners need tooling to:
1. Monitor expiry timelines across all registered names.
2. Automate renewal transactions before semi-expiry.
3. Handle key management for renewal signatures.
4. Alert operators if renewal is approaching or has lapsed.

Failure to provide reliable renewal tooling led to legitimate Namecoin owners losing names through accidental non-renewal, even when they intended to keep them.

---

## Design Implications for a Decentralised Naming RFP

1. **Expiry period should balance freshness against operational burden.** Annual renewal (ENS model) is a reasonable default for general-purpose naming. 250-day expiry (Namecoin) is relatively short and increases renewal management burden.

2. **Grace periods are user-friendly and should be included.** A window where the name stops resolving but the owner retains renewal rights is a practical protection against accidental loss.

3. **Renewal fees are necessary for anti-squatting effectiveness.** Free renewal subsidises squatting. Meaningful renewal fees force squatters to continuously evaluate whether holding a name is economically rational.

4. **Renewal automation must be first-class tooling.** Any system that requires periodic on-chain transactions must provide reliable renewal automation (cron-style tooling, guardian services) or users will lose names inadvertently.

5. **Expiry state transitions must be clearly defined and well-indexed.** Indexers and resolvers must correctly handle names in each state (active, semi-expired, fully expired) to provide accurate resolution.

---

## Related Notes

- [[namecoin]]
- [[name-squatting-prevention]]
- [[merged-mining-for-naming]]

---

## Sources

- [Namecoin FAQ](https://www.namecoin.org/docs/faq/)
- [Namecoin Wikipedia](https://en.wikipedia.org/wiki/Namecoin)
- [ENS documentation: expiry and renewal](https://docs.ens.domains/)
- [Kalodner et al. WEIS 2015](https://econinfosec.org/archive/weis2015/papers/WEIS_2015_kalodner.pdf)
