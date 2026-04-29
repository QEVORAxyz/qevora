# Architecture

QEVORA is composed of four layers that operate independently but compose into a complete identity system.

## Layers

### 1. Entropy Layer

The foundation of every agent identity is hardware quantum entropy. QEVORA interfaces with certified QRNG devices that measure irreducible physical phenomena such as photonic vacuum fluctuations and radioactive decay timing. These measurements produce entropy that cannot be predicted or reproduced, even by an adversary with full knowledge of the system state.

Entropy quality is validated against NIST SP 800-90B before use in key derivation. Any entropy source failing validation is rejected and an exception is raised before any cryptographic material is produced.

### 2. Cryptographic Layer

Raw entropy feeds into the key derivation pipeline, which uses CRYSTALS-Kyber-1024 for key encapsulation and CRYSTALS-Dilithium-3 for digital signatures. Both are NIST-standardized post-quantum constructions whose security reduces to the hardness of Module LWE.

Private keys never leave the cryptographic layer. All signing operations occur in an isolated enclave context. Public keys and agent identifiers are the only outputs exposed to higher layers.

### 3. On-Chain Registry Layer

Agent identities are anchored to Solana through a set of Program Derived Accounts managed by the QEVORA on-chain program. Each agent identity corresponds to a unique PDA storing the agent's public key, metadata, and revocation status.

This layer provides:
- Globally accessible identity resolution
- Append-only inscription of identity events
- Decentralized revocation without a central authority
- Finality backed by Solana's Proof of History and Tower BFT consensus

### 4. Verification Layer

Signature verification is a local operation. The verifier fetches the agent's public key from the on-chain registry once, then performs all subsequent verifications offline using CRYSTALS-Dilithium. No network calls are required during verification. No trusted parties are involved.

## Component Interactions

```
  QRNG Hardware
       |
       v
  Key Derivation (Kyber + Dilithium)
       |
       +---> Private Key (stays in enclave)
       |
       +---> Public Key + Agent ID
                   |
                   v
            Solana Registry (PDA)
                   |
                   v
            Verifier (local Dilithium check)
```
