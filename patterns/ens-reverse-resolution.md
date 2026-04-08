---
tags: [pattern, naming, privacy, ethereum]
related_projects: [ens]
ensip: ENSIP-19 (multichain primary names)
---

# Pattern: ENS Reverse Resolution and Primary Names

## Summary

ENS forward resolution maps `alice.eth` to an Ethereum address. Reverse resolution maps an Ethereum address back to the name the owner wants displayed: their **primary name**. This bidirectional relationship is what allows wallets, block explorers, and dApps to show `alice.eth` instead of `0xabc...` when displaying the user's identity. The primary name is a deliberate user choice and requires both directions to be set correctly.

## How it works

### Technical mechanism

1. **Forward record**: the name (`alice.eth`) has an `addr()` record on its resolver pointing to the owner's address.
2. **Reverse record**: the address is registered in the **Reverse Registrar** contract under the `.addr.reverse` namespace. The node for an address is `namehash(<lowercase hex address>.addr.reverse)`.
3. **Claim**: the Reverse Registrar maps `<address>.addr.reverse` to a name. The user sets this via a transaction calling `setName(string name)` on the Reverse Registrar.
4. **Validation requirement**: for a primary name to be considered valid, **both** the forward record (name → address) and the reverse record (address → name) must match. Applications should verify both directions; displaying an unverified primary name is a security risk.

### Resolution flow

```
App wants to display the name for 0xabc...
  1. Compute node = namehash("0xabc....addr.reverse")
  2. Query ENS Registry for resolver of that node
  3. Call name(node) on the resolver -> returns "alice.eth"
  4. Verify: resolve "alice.eth" -> addr() -> should return 0xabc...
  5. If both match, display "alice.eth". Otherwise, display raw address.
```

## Privacy implications

Setting a primary name makes the bidirectional link between a wallet address and a human-readable identity **publicly queryable on-chain**. Implications:

- Any address visible in a transaction (sender, recipient, contract caller) can be reverse-resolved to a name, linking on-chain activity to an identity.
- Block explorers automatically display primary names, reducing pseudonymity.
- The ENS Docs advise users to "consider privacy implications of reverse records" when setting them.
- A grant proposal for verifiable and privacy-preserving ENS resolution (2024) notes that "the resolver sees every request a user makes, and resolvers offer no meaningful privacy guarantees." Source: [ENS DAO Forum](https://discuss.ens.domains/t/grant-proposal-verifiable-and-privacy-preserving-ens-resolution/21108)

Opting out: a user who does not set a primary name retains pseudonymity. However, the social and product pressure to use human-readable identities means many users do set primary names.

## ENSIP-19: Multichain primary names

Traditional primary names are Ethereum-specific. ENSIP-19 extends the pattern to other chains: a user can set a primary name for their Bitcoin address, Solana address, or other chain address. This requires chain-specific reverse registrars or a canonical cross-chain primary name registry.

As of April 2026, ENSIP-19 is a proposed standard. Source: [ENSIP-19](https://docs.ens.domains/ensip/19/)

## Governance and UX notes

- Applications should **never silently change a user's primary name** without explicit consent and clear notification. The ENS Docs warn against this.
- The transaction to set a primary name costs gas. ENS DAO introduced **gasless voting** for governance, but primary name setting still requires an on-chain transaction.
- ENSv2 is expected to improve primary name handling for L2 contexts (see [ENS Blog: L2 Primary Names](https://ens.domains/blog/post/l2-primary-names)).

## RFP relevance

Reverse resolution / primary names reveal a fundamental tension in naming systems between usability and privacy:

1. **Usability case**: users want their chosen name to appear everywhere; bidirectional records make this seamless.
2. **Privacy cost**: primary names make wallet activity deanonymisable. Any privacy-preserving naming system must either omit reverse resolution or make it opt-in and unlinkable.
3. **Design choice for Logos context**: the Waku Naming Service prototype deliberately maps names to public keys, not wallet addresses, specifically to avoid this privacy exposure. A Logos naming RFP should explicitly address whether primary names or reverse resolution are in scope, and if so, what privacy guarantees apply.

## Sources

- [ENS Docs: Primary Names (Reverse)](https://docs.ens.domains/web/reverse/) — accessed 2026-04-08
- [ENS Docs: Reverse Registrars](https://docs.ens.domains/registry/reverse/) — accessed 2026-04-08
- [ENSIP-19: Multichain Primary Names](https://docs.ens.domains/ensip/19/) — accessed 2026-04-08
- [ENS Support: The Primary Name](https://support.ens.domains/en/articles/7890756-the-primary-name) — accessed 2026-04-08
- [ENS DAO Forum: Privacy Grant Proposal](https://discuss.ens.domains/t/grant-proposal-verifiable-and-privacy-preserving-ens-resolution/21108) — accessed 2026-04-08
- [ENS Blog: L2 Primary Names](https://ens.domains/blog/post/l2-primary-names) — accessed 2026-04-08
- [Enscribe: Primary vs Forward-Resolving Names Security](https://www.enscribe.xyz/blog/primary-vs-forward-resolving-names-security-boundary) — accessed 2026-04-08
