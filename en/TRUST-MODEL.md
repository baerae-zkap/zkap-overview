# ZKAP Trust Model

*Read this in [한국어](../ko/TRUST-MODEL.md).*

> What ZKAP relies on, and what it does not. Written for someone evaluating
> whether the security properties hold up. For how the system works, see
> [ARCHITECTURE.md](./ARCHITECTURE.md); for terms, [GLOSSARY.md](./GLOSSARY.md).

---

## What ZKAP relies on, and what it doesn't

**Relies on (trust assumptions):**

- **OIDC providers** sign tokens honestly and publish genuine signing keys
  (JWKs). A provider that mints a token for an attacker, or whose key is
  mis-registered on-chain, is a real risk surface.
- **The on-chain issuer directory is maintained correctly** — the Merkle tree of
  trusted issuer keys is updated (under timelock governance) only with genuine
  provider keys.
- **The trusted setup was performed honestly** — the Groth16 setup secret was
  destroyed. If it survives, false proofs are possible.
- **The circuit and contracts are correct** — soundness of the statement and the
  on-chain verifier. Not yet audited.
- **ERC-4337 infrastructure is live** — bundlers (and optionally a paymaster) to
  get operations included.
- **The user's device** protects the passkey used for everyday signing.

**Does *not* rely on:**

- **A ZKAP backend for custody, signing, or recovery.** No backend holds keys or
  can recover an account. The reference wallet makes zero backend calls; signing
  keys live on the device; recovery is enforced on-chain.
- **Any single OIDC provider** — recovery is a *k-of-n* threshold across
  independent issuers.
- **Revealing the token** — the JWT and its claims are never sent to a backend or
  put on-chain.
