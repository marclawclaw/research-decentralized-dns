---
tags: [pattern, naming, cryptography, ethereum]
related_projects: [ens, unstoppable-domains, space-id]
ensip: ENSIP-1 (EIP-137)
---

# Pattern: ENS Namehash

## Summary

Namehash is a recursive algorithm that converts a human-readable domain name into a 32-byte cryptographic node used throughout ENS smart contracts. The algorithm ensures that parent-level ownership implies child-level authority: only the holder of `example.eth` can register `sub.example.eth`.

## Algorithm

```
namehash('') = 0x0000...0000   (32 zero bytes)
namehash(name) = keccak256(namehash(parent(name)) ++ keccak256(label(name)))
```

Where `label(name)` is the leftmost DNS label (e.g. `alice` in `alice.eth`) and `parent(name)` is the remainder (`eth` in `alice.eth`).

Example:
- `namehash('eth')` = `keccak256(namehash('') ++ keccak256('eth'))`
- `namehash('alice.eth')` = `keccak256(namehash('eth') ++ keccak256('alice'))`

The resulting 32-byte value is called the **node**.

## Normalisation (ENSIP-15)

Before namehashing, the input string must be normalised per ENSIP-15:

- Unicode normalisation using the latest stable Unicode version
- UTS-46 processing for ASCII compatibility encoding
- UTS-51 emoji sequence processing (UTS-46 alone is insufficient for emoji)
- Rejection of invalid codepoints, zero-width characters, and mixed-script confusables
- Case folding (names are case-insensitive; `Alice.ETH` and `alice.eth` resolve to the same node)

Failure to normalise consistently across implementations produces different node values for what users perceive as the same name, causing resolution failures or spoofing vulnerabilities.

Libraries: `@adraffy/ens-normalize` (JS), `ens-normalize-python` (Python). Both implement ENSIP-15.

## Security implications

- **Homograph attacks**: unicode allows visually identical characters from different scripts (e.g. Cyrillic 'а' vs Latin 'a'). ENSIP-15 mitigates most cases by restricting mixed-script names, but client-side enforcement is required.
- **Zero-width character insertion**: ENSIP-15 bans zero-width joiners and similar invisible codepoints in name identifiers.
- **Confusable names**: some character combinations remain visually ambiguous even after normalisation. Applications should display warnings for names containing potentially deceptive graphemes.

Source: [ENSIP-15](https://docs.ens.domains/ensip/15/); [SEAL ENS Name Handling](https://frameworks.securityalliance.org/ens/name-handling-normalization/)

## RFP relevance

Any naming protocol must define a canonical name-to-identifier transformation. Key design decisions:

1. **Determinism**: the same input must always produce the same identifier on any compliant implementation.
2. **Namespace hierarchy**: the algorithm should reflect ownership hierarchy so parent-owned namespaces can delegate to children without a central registrar.
3. **Unicode handling**: supporting non-ASCII names requires a clear normalisation standard; ENS's ENSIP-15 is a production-tested reference.
4. **Homograph resistance**: pure technical normalisation is insufficient; client-side display warnings are also required.

## Sources

- [EIP-137: ENS Specification](https://eips.ethereum.org/EIPS/eip-137) — accessed 2026-04-08
- [ENSIP-15: Name Normalization](https://docs.ens.domains/ensip/15/) — accessed 2026-04-08
- [ENS Name Processing Docs](https://docs.ens.domains/resolution/names/) — accessed 2026-04-08
