# ZKAP Glossary

*Read this in [한국어](../ko/GLOSSARY.md).*

> Plain-English definitions of the terms used across the ZKAP docs and repos.
> Back to the overview: [README.md](./README.md) ·
> [ARCHITECTURE.md](./ARCHITECTURE.md) · [TRUST-MODEL.md](./TRUST-MODEL.md) ·
> [REPOS.md](./REPOS.md).

Grouped four ways: zero-knowledge proving, social login, account abstraction,
and ZKAP-specific terms.

---

## Zero-knowledge & proving

- **Zero-knowledge proof (ZK proof).** A proof that a statement is true while
  revealing nothing else. In ZKAP the prover shows a login token is valid
  *without revealing the token*.
- **ZK-SNARK.** A zero-knowledge proof that is *succinct* (small) and
  *non-interactive* (one message, no back-and-forth) and cheap to verify.
- **Groth16.** The specific ZK-SNARK system ZKAP uses. Proofs are a fixed small
  size (256 bytes / 8 field elements) and cheap to check on-chain.
- **BN254 (alt_bn128).** The elliptic curve the proofs are built on. The EVM has
  precompiles for it (`0x06`/`0x07`/`0x08`), which is what makes on-chain
  verification affordable.
- **Poseidon.** A hash function designed to be cheap *inside* a circuit (far
  fewer constraints than SHA-256). Used for the Merkle tree, the anchor, and the
  nonce.
- **R1CS (Rank-1 Constraint System).** The arithmetic form a statement is
  compiled into for a SNARK. ZKAP's circuit is an R1CS over BN254.
- **arkworks.** The Rust cryptography library ecosystem the circuit is built on.
- **Trusted setup.** A one-time, per-circuit step that produces the proving and
  verifying keys. Its secret randomness ("toxic waste") must be destroyed; if it
  leaks, false proofs become possible — which is why it is run as a multi-party
  ceremony.
- **CRS / CRS bundle (Common Reference String).** The trusted-setup output the
  SDK proves against: proving key, verifying keys, the Solidity verifier, the
  circuit config, and a manifest. See *manifest*.
- **Proving key (pk) / verifying key (vk) / prepared verifying key (pvk).** `pk`
  generates a proof; `vk`/`pvk` verify it (`pvk` is a precomputed-for-speed form
  of `vk`).
- **Public inputs.** The values both prover and verifier see, that a proof is
  asserted *about*. Each ZKAP proof carries an 8-element public-input vector.

## Social login (OIDC / JWT)

- **OIDC (OpenID Connect).** The identity layer on top of OAuth 2.0 that "Sign in
  with Google / Apple" is built on.
- **JWT (JSON Web Token).** The signed token a provider issues when a user logs
  in. ZKAP proves a JWT is genuine without revealing it.
- **ID token.** The OIDC JWT that asserts who the user is.
- **JWK (JSON Web Key).** A provider's *public* signing key, published so anyone
  can verify the JWTs it signed.
- **kid (key ID).** A field in the JWT header naming which JWK signed the token,
  so a verifier picks the right key.
- **aud (audience).** The OAuth client ID a token was issued for. ZKAP commits it
  on-chain (see *hAudList*); a proof must match the committed audience.
- **iss (issuer).** The provider that issued the token (e.g.
  `https://accounts.google.com`).
- **sub (subject).** The provider's stable identifier for the user.
- **nonce.** A value the client puts in the login request that the provider
  echoes back inside the JWT. ZKAP binds it to a specific operation (see
  *zkNonce*).
- **RS256 / RSA-2048.** The signature algorithm (RSA + SHA-256, 2048-bit key)
  most OIDC providers sign JWTs with. The circuit verifies this signature.

## Account abstraction (ERC-4337)

- **Account abstraction.** Making an account a programmable smart contract
  instead of a bare keypair, so the rules for who may sign and how are code.
- **ERC-4337.** The Ethereum standard for account abstraction that needs no
  protocol change; user intents flow through a shared EntryPoint.
- **EntryPoint.** The single canonical contract that validates and executes
  UserOperations.
- **UserOperation (UserOp).** The account-abstraction "transaction": a packet of
  intent plus signature, submitted to the EntryPoint by a bundler.
- **Bundler.** An off-chain service that collects UserOps and submits them
  on-chain to the EntryPoint.
- **Paymaster.** A contract that can sponsor gas so a user transacts without
  holding ETH. Optional in ZKAP.
- **Account factory.** The contract that deploys a smart account deterministically
  (`ZkapAccountFactory`).
- **Smart account.** A wallet that *is* a contract (here, `ZkapAccount`), not an
  EOA.
- **EOA (externally owned account).** A traditional keypair-controlled Ethereum
  account.
- **CREATE2.** An opcode that makes a contract's address deterministic from its
  inputs — so the address is known before deployment.
- **Counterfactual address.** A wallet's address, known and fundable *before* the
  wallet is deployed (thanks to CREATE2); the first UserOp deploys it.

## ZKAP-specific

- **ZKAP (Zero-Knowledge Authentication Protocol).** The protocol this hub
  documents: prove a social login in zero knowledge and use it as the root
  authority of an Ethereum smart account.
- **ZkapAccount.** The ERC-4337 smart-account contract at the center of a ZKAP
  wallet.
- **Master key (ZK-OAuth).** The high-value authority — ownership, recovery, key
  changes — spent with a ZK proof of social login.
- **TX key.** The everyday authority — transactions — spent with a device-held
  passkey or ECDSA signature.
- **Passkey / WebAuthn / secp256r1.** The device-held credential (FIDO2 /
  WebAuthn, over the P-256 / secp256r1 curve) used to sign everyday transactions.
- **ECDSA.** The signature scheme used for EOA / address-key signing.
- **Threshold anchor.** A commitment registered on-chain that binds recovery to a
  *k-of-n* set of OIDC identities via a Vandermonde / Shamir-style polynomial
  scheme — so no single issuer is required to recover.
- **k-of-n.** The threshold rule: any *k* valid proofs out of *n* registered
  identities authorize recovery.
- **Vandermonde / Shamir scheme.** The polynomial math behind the threshold
  anchor.
- **hAudList / audience hash.** The Poseidon commitment of the allowed
  audience(s) (`aud`) stored in the account; a proof's `h_aud_list` must match it
  to be accepted.
- **Issuer Merkle directory.** An on-chain Poseidon Merkle tree of trusted
  issuers' RSA public keys; the circuit proves the signing key is a member
  (`PoseidonMerkleTreeDirectory`).
- **Merkle leaf / path / root.** Standard Merkle pieces: a *leaf* is one issuer-key
  commitment; a *path* proves the leaf is in the tree under the *root*.
- **tree_height.** The circuit/SDK Merkle path length (15). The on-chain contract
  depth is `tree_height + 1` (16).
- **manifest.** `manifest.json` inside a CRS bundle: per-file hashes plus an
  optional signature. It is the *single trust gate* — the loader checks it, and
  `prove()` then trusts the bundle and re-verifies nothing.
- **zkNonce.** `nonce = Poseidon(userOpHash, random)`. Binding the JWT nonce to a
  specific UserOp makes a proof valid for that one operation only — replay
  protection.
- **Groth16Verifier / on-chain verifier.** The Solidity verifier *generated from
  the circuit* that checks proofs on-chain via the BN254 pairing precompile.
