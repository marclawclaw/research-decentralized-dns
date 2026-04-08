---
tags: [pattern, auction, vickrey, naming, tld, handshake]
pattern: TLD Auction Mechanism
seen-in: [[handshake]]
---

# Pattern: TLD Auction Mechanism (Vickrey Sealed-Bid)

A protocol for allocating top-level domain names via a blind Vickrey auction enforced at the consensus layer, rather than through a central registrar. The name is awarded to the highest bidder, but the price paid is the second-highest bid.

---

## Where It Appears

- [[handshake]] (primary production example; TLD-level auctions)

---

## Mechanism Detail

The auction is implemented as a chain of transaction covenants on the Handshake blockchain. Each phase is a separate covenant type embedded in a UTXO output.

### Phase 1: OPEN (day 0)

Any participant broadcasts an OPEN covenant to initiate an auction for an unclaimed TLD. The 5-day bidding window begins.

### Phase 2: BID (days 1 to 5)

Participants submit blinded bids:

- **bid**: the actual amount the bidder is willing to pay (hidden)
- **blind**: additional HNS added to obscure the true bid
- **lockup** (visible on-chain): bid + blind; what competing bidders can observe

The blind is returned after the reveal phase regardless of outcome. This means competitors cannot determine the actual bid from the lockup alone; they only see an upper bound.

- Source: [Namebase auction tutorial](https://www.namebase.io/blog/tutorial-3-basics-of-handshake-auction-and-bidding/)
- Source: [hsd-dev auction guide](https://hsd-dev.org/guides/auctions.html)

### Phase 3: REVEAL (days 6 to 15)

Bidders must publish a REVEAL covenant within the 10-day reveal window. This reveals their true bid. Bidders who fail to reveal forfeit their lockup (bid + blind are burned as a penalty).

### Phase 4: REGISTER (day 16+)

At the close of the reveal period:

- The highest bidder's REVEAL is selected as the winner
- The winner pays only the second-highest revealed bid (Vickrey pricing)
- The winner's bid minus the second-highest bid is returned to the winner
- All other bidders receive their full lockup back
- The winning payment (second-highest bid amount) is burned (not sent to any party)

The winner broadcasts a REGISTER covenant. The name is now permanently registered to their address.

- Source: [Namebase auction tutorial](https://www.namebase.io/blog/tutorial-3-basics-of-handshake-auction-and-bidding/)
- Source: [CoinGecko HNS auction guide](https://www.coingecko.com/learn/how-to-bid-on-a-handshake-name-auction-using-namebase)

---

## Why Vickrey Pricing?

A Vickrey auction is incentive-compatible: the dominant strategy for each bidder is to bid their true valuation. Over-bidding is punished because the winner pays the second-highest bid, not their own. Under-bidding risks losing to a lower-value competitor. This property is desirable for name allocation because it discourages strategic gaming and produces prices that reflect genuine demand.

In contrast, a first-price auction rewards low-balling; a Dutch auction rewards speed; neither accurately reflects valuation.

---

## Timing Parameters (Handshake)

| Phase | Duration | Notes |
|-------|----------|-------|
| Bid window | ~5 days | ~720 blocks at 10 min/block |
| Reveal window | ~10 days | ~1,440 blocks |
| Total auction cycle | ~15 days | Minimum latency for name acquisition |
| Renewal period | Every 2 years | RENEW covenant; miner fee only; no re-auction |
| Transfer lock | ~2 days | TRANSFER covenant; REVOKE possible during lock |

- Source: [hsd-dev protocol summary](https://hsd-dev.org/protocol/summary.html)
- Source: [Handshake FAQ](https://handshake.org/faq/)

---

## Bid Privacy Assessment

The Vickrey mechanism requires some privacy of bids during the bidding phase. Handshake achieves this via:

1. The blind obfuscates the true bid amount on-chain (only lockup is visible)
2. The blockchain reveals bids only after the reveal phase begins

Limitation: bidders can observe the lockup values of all competing bids in real time during the bidding phase. A sophisticated attacker could attempt to outbid by observing lockup sizes and bidding slightly above the apparent maximum. The blind provides plausible deniability but not strong privacy guarantees.

---

## RFP Implications

- A decentralised naming system could adopt a similar two-phase auction for name allocation, avoiding the need for a central registrar
- The Vickrey mechanism prevents price gouging by auction operators (burned payment goes to no party)
- Renewal-only fees (no re-auction) prevent continuous rent extraction and provide ownership stability
- The 15-day minimum acquisition latency is a usability trade-off vs. instant registration
- Forfeited lockups (for unrevealed bids) require users to manage time-sensitive transactions; poor UX for non-technical users

---

## Sources

- [Namebase: Basics of HNS auction and bidding](https://www.namebase.io/blog/tutorial-3-basics-of-handshake-auction-and-bidding/)
- [hsd-dev auction guide](https://hsd-dev.org/guides/auctions.html)
- [CoinGecko: How to bid on HNS](https://www.coingecko.com/learn/how-to-bid-on-a-handshake-name-auction-using-namebase)
- [Handshake FAQ](https://handshake.org/faq/)
- [hsd-dev protocol summary](https://hsd-dev.org/protocol/summary.html)
