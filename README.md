# QEVORA

**Post-quantum cryptographic identity infrastructure for autonomous AI agents, anchored on Solana.**

As machine economies scale beyond human oversight, identity becomes the foundational primitive. QEVORA provides every autonomous agent with a cryptographic identity that is physically rather than algorithmically unforgeable, built on NIST-standardized post-quantum primitives over module lattices and registered on-chain through Solana.

---

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Repository Structure](#repository-structure)
- [Core Modules](#core-modules)
- [Identity Lifecycle](#identity-lifecycle)
- [Cryptographic Stack](#cryptographic-stack)
- [Verification Model](#verification-model)
- [Security Guarantees](#security-guarantees)
- [Getting Started](#getting-started)
- [Roadmap](#roadmap)

---

## Overview

QEVORA is identity infrastructure for a world where AI agents operate autonomously, execute transactions, and communicate with each other at machine speed. Classical public key infrastructure was designed for humans and human-scale systems. It breaks under the demands of autonomous agents and it will break further when fault-tolerant quantum computers arrive.

QEVORA solves both problems at once. Every agent identity is derived from hardware quantum entropy, making it stochastically irreducible and physically unique. Every operation is signed with post-quantum digital signatures that remain secure against both classical and quantum adversaries through the 2080 security horizon. Agent identities are registered as on-chain accounts on Solana, giving every identity a decentralized and globally-accessible anchor with the throughput and finality that autonomous agent economies require.

The protocol requires no certificate authorities, no trust anchors, and no online dependencies. Any Solana network participant can verify any signature through direct lattice arithmetic.

---

## Architecture

### System Overview

```
+------------------------------------------------------------------+
|                         QEVORA NETWORK                           |
+------------------------------------------------------------------+
|                                                                  |
|  +-----------------+   +-----------------+   +-----------------+ |
|  |  Quantum Layer  |   |  Identity Core  |   |   Agent SDK     | |
|  |                 |   |                 |   |                 | |
|  | QRNG Hardware   +-->| Key Derivation  +-->| Sign / Verify   | |
|  | Entropy Source  |   | CRYSTALS-Kyber  |   | CRYSTALS        | |
|  | Stochastic RNG  |   | Module LWE      |   | Dilithium       | |
|  +-----------------+   +-----------------+   +-----------------+ |
|          |                     |                     |           |
|          +---------------------+---------------------+           |
|                                |                                 |
|          +---------------------v---------------------+           |
|          |            PERSISTENCE LAYER              |           |
|          |   Solana On-Chain Registry (PDAs)         |           |
|          |   Signature Ledger   +   State Store      |           |
|          +---------------------+---------------------+           |
|                                |                                 |
|             +------------------+------------------+              |
|             |                                     |              |
|  +----------+----------+             +------------+-----------+  |
|  |    Verification     |             |     Agent Runtime      |  |
|  |    Zero Trust       |             |     Execution Env      |  |
|  |    Solana On-Chain  |             |     Transaction Mgr    |  |
|  +---------------------+             +------------------------+  |
|                                                                  |
+------------------------------------------------------------------+
```

### Identity Provisioning Flow

```
  Agent Spawn Request
          |
          v
  +-------+--------+
  |  Quantum RNG   |  Hardware entropy source
  |  Seed Gen      |  Stochastic, physically irreducible
  +-------+--------+
          |
          | 512-bit entropy seed
          v
  +-------+--------+
  |  Key Derivation|  CRYSTALS-Kyber-1024 (NIST FIPS 203)
  |  Module LWE    |  Module Learning With Errors
  +-------+--------+
          |
     +----+----+----+
     |         |    |
     v         v    v
  +------+ +-------+ +----------+
  |Pub   | |Priv   | | Agent ID |
  |Key   | |Key    | | hash+ts  |
  +------+ +-------+ +----------+
     |                    |
     v                    v
  +------+          +----------+
  |Publish|         | Solana   |
  |Key   |          | Registry |
  +------+          +----------+
```

### Agent Signing Flow

```
  Agent Operation
          |
          v
  +-------+----------+
  | Operation Payload |
  | + Nonce           |  Replay-resistant
  +-------+-----------+
          |
          v
  +-------+----------+
  | CRYSTALS-        |
  | Dilithium-3      |  Existential unforgeability
  | Signature Engine |  Adaptive chosen message attack
  +-------+----------+
          |
          v
  +-------+----------+
  | Signed Operation  |
  | payload           |
  | signature         |
  | agent_id          |
  | nonce             |
  +-------+----------+
          |
       broadcast
          |
     +----+----+
     |         |
     v         v
  Peer A    Peer B
  verify    verify
  on-chain  on-chain
```

### Zero Trust Verification

```
  Input: { payload, signature, agent_public_key }
          |
          v
  +-------+----------+
  | Fetch public key |  From Solana on-chain registry
  | (one-time read)  |  No CA lookup. No OCSP. Self-certifying.
  +-------+----------+
          |
          v
  +-------+----------+
  | Lattice Verify   |  CRYSTALS-Dilithium
  | sig over payload |  Reducible to Module LWE hardness
  +-------+----------+
          |
     +----+----+
     |         |
   VALID    INVALID
     |         |
     v         v
  Accept    Reject
```

---

## Repository Structure

This organization is split into focused repositories. Each handles a distinct layer of the stack.

```
QEVORAxyz/
|
+-- qevora/                              (this repo — protocol spec and docs)
|   |-- docs/
|   |   |-- architecture.md             full system architecture
|   |   |-- cryptography.md             cryptographic primitive details
|   |   |-- identity-lifecycle.md       provisioning, rotation, revocation
|   |   |-- verification-model.md       zero trust verification protocol
|   |   |-- solana-integration.md       on-chain registry and PDA design
|   |   |-- threat-model.md             adversary assumptions and scope
|   |   |-- api-reference.md            SDK and program API reference
|   |   |-- getting-started.md          quickstart guide for developers
|   |-- README.md
|   |-- SPEC.md
|   |-- SECURITY.md
|
+-- qevora-core/                         (cryptographic primitives)
|   |-- src/
|   |   |-- kyber/
|   |   |   |-- keygen.rs               key pair generation
|   |   |   |-- encaps.rs               key encapsulation
|   |   |   |-- decaps.rs               key decapsulation
|   |   |   |-- params.rs               Kyber-1024 parameters
|   |   |-- dilithium/
|   |   |   |-- keygen.rs               signing key generation
|   |   |   |-- sign.rs                 signature creation
|   |   |   |-- verify.rs               signature verification
|   |   |   |-- params.rs               Dilithium-3 parameters
|   |   |-- qrng/
|   |   |   |-- interface.rs            hardware QRNG abstraction
|   |   |   |-- hardware.rs             physical device drivers
|   |   |   |-- fallback.rs             NIST SP 800-90B compliant fallback
|   |   |-- lwe/
|   |   |   |-- arithmetic.rs           module lattice arithmetic
|   |   |   |-- reduction.rs            noise reduction routines
|   |   |-- hash/
|   |       |-- sha3.rs                 SHA-3 implementation
|   |       |-- shake.rs                SHAKE-256 XOF
|   |-- tests/
|   |   |-- kyber_vectors.rs            NIST test vectors
|   |   |-- dilithium_vectors.rs        NIST test vectors
|   |   |-- qrng_entropy.rs             entropy quality tests
|   |-- Cargo.toml
|
+-- qevora-sdk/                          (agent integration SDK)
|   |-- typescript/
|   |   |-- src/
|   |   |   |-- identity/
|   |   |   |   |-- provision.ts        identity provisioning
|   |   |   |   |-- rotate.ts           key rotation
|   |   |   |   |-- revoke.ts           identity revocation
|   |   |   |-- signing/
|   |   |   |   |-- sign.ts             operation signing
|   |   |   |   |-- verify.ts           signature verification
|   |   |   |   |-- nonce.ts            replay-resistant nonce generation
|   |   |   |-- solana/
|   |   |   |   |-- registry.ts         on-chain registry interface
|   |   |   |   |-- pda.ts              PDA derivation helpers
|   |   |   |-- client.ts               main SDK client
|   |   |   |-- index.ts                public API surface
|   |   |-- package.json
|   |-- python/
|   |   |-- qevora/
|   |   |   |-- identity.py
|   |   |   |-- signing.py
|   |   |   |-- registry.py
|   |   |-- setup.py
|   |-- rust/
|   |   |-- src/
|   |   |   |-- lib.rs
|   |   |   |-- identity.rs
|   |   |-- Cargo.toml
|
+-- qevora-node/                         (Solana on-chain program — Anchor)
|   |-- programs/
|   |   |-- qevora/
|   |   |   |-- src/
|   |   |   |   |-- lib.rs              program entrypoint
|   |   |   |   |-- instructions/
|   |   |   |   |   |-- provision.rs    register new agent identity
|   |   |   |   |   |-- rotate.rs       rotate agent key pair
|   |   |   |   |   |-- revoke.rs       revoke agent identity
|   |   |   |   |   |-- verify.rs       on-chain signature verification
|   |   |   |   |-- state/
|   |   |   |   |   |-- identity.rs     identity account schema
|   |   |   |   |   |-- ledger.rs       signature ledger account schema
|   |   |   |   |-- error.rs            program error types
|   |-- tests/
|   |   |-- provision.ts
|   |   |-- rotation.ts
|   |   |-- revocation.ts
|   |-- Anchor.toml
|   |-- Cargo.toml
|
+-- qevora-cli/                          (command line interface)
    |-- src/
    |   |-- commands/
    |   |   |-- provision.rs             create and register agent identity
    |   |   |-- sign.rs                  sign an operation payload
    |   |   |-- verify.rs                verify a signature against registry
    |   |   |-- rotate.rs                rotate key pair with continuity proof
    |   |   |-- revoke.rs                revoke an identity on-chain
    |   |   |-- inspect.rs               inspect identity or signature details
    |   |-- config.rs                    CLI configuration and keypair loading
    |   |-- main.rs                      entrypoint
    |-- Cargo.toml
```

### Dependency Graph

```
qevora-core
    |
    +---------> qevora-sdk
    |               |
    |               +---------> TypeScript / Python / Rust bindings
    |
    +---------> qevora-node
    |               |
    |               +---------> Solana devnet / mainnet
    |
    +---------> qevora-cli
                    |
                    +---------> developer tooling
```

---

## Core Modules

### qevora-core

The cryptographic foundation. All other modules depend on this.

| Component          | Algorithm               | Standard      | Security Level |
|--------------------|-------------------------|---------------|----------------|
| Key Encapsulation  | CRYSTALS-Kyber-1024     | NIST FIPS 203 | Level 5        |
| Digital Signatures | CRYSTALS-Dilithium-3    | NIST FIPS 204 | Level 3        |
| Hash Functions     | SHA-3 / SHAKE-256       | NIST FIPS 202 | 256-bit        |
| Entropy Source     | QRNG hardware interface | ISO 20543     | Stochastic     |

### qevora-sdk

Developer-facing API for agent identity operations.

```
qevora.identity.provision(entropy_seed)
    -> { public_key, agent_id, created_at }

qevora.identity.sign(private_key, payload, nonce)
    -> { signature, agent_id, nonce, timestamp }

qevora.identity.verify(public_key, signature, payload)
    -> { valid: bool, agent_id, verified_at }

qevora.identity.rotate(old_private_key, new_entropy_seed)
    -> { new_public_key, rotation_proof }
```

### qevora-node

Solana on-chain program that maintains the identity registry and processes identity operations.

```
  +---------------------+
  |   Registry API      |  REST + gRPC endpoints
  +---------------------+
  |   Identity PDAs     |  Program Derived Accounts per agent
  +---------------------+
  |   Signature Ledger  |  Indexed by agent_id + nonce
  +---------------------+
  |   Ix Handlers       |  provision / rotate / revoke / verify
  +---------------------+
  |   Consensus         |  Solana Proof of History + Tower BFT
  +---------------------+
```

---

## Identity Lifecycle

```
  1. PROVISIONING
     Quantum entropy -> key derivation -> Solana registry inscription
     Single operation at agent spawn time

  2. ACTIVE
     Agent signs all operations with Dilithium
     Signatures are noninteractive and replay-resistant
     No expiry, no renewal process

  3. ROTATION
     Triggered by policy or compromise detection
     Old key signs rotation proof
     New key published on-chain with cryptographic continuity

  4. REVOCATION
     Threshold of validators co-sign revocation
     Solana registry marks key as revoked
     Historical signatures remain verifiable
```

---

## Cryptographic Stack

### Why Module LWE

The hardness of Learning With Errors over module lattices is the security foundation for both Kyber and Dilithium. Unlike RSA or elliptic curve cryptography, no known quantum algorithm provides exponential speedup against LWE. Shor's algorithm, which breaks RSA and ECC in polynomial time on a quantum computer, has no analogue for lattice problems.

Security reductions from Kyber and Dilithium to Module LWE are tight, meaning the security of the scheme degrades gracefully and predictably as parameter sizes change.

### Parameter Selection

```
  Kyber-1024
  Security level    256-bit post-quantum (NIST Level 5)
  Public key size   1568 bytes
  Ciphertext size   1568 bytes
  Shared secret     32 bytes

  Dilithium-3
  Security level    128-bit post-quantum (NIST Level 3)
  Public key size   1952 bytes
  Signature size    3293 bytes
  Private key size  4000 bytes
```

### Entropy Requirements

Agent identities require a minimum of 256 bits of quantum entropy from a certified QRNG source. Software pseudorandom number generators are explicitly excluded from the provisioning path. The stochastic irreducibility of quantum measurement means no two identity seeds can be identical even under adversarial conditions, as this would require predicting quantum state collapse.

---

## Verification Model

QEVORA uses a certificate-free verification architecture. There are no certificate authorities, no certificate chains, no revocation servers, and no expiry timestamps that require online validation.

Verification works as follows:

1. The verifier fetches the agent public key from the on-chain identity registry on Solana.
2. The verifier runs Dilithium verification locally using the public key and the received signature.
3. The result is a binary valid or invalid outcome derived entirely from lattice arithmetic.
4. No trusted third parties. No single points of failure. The Solana ledger is the source of truth.

This architecture inherits Solana's decentralization for registry reads while keeping verification itself a pure local computation. It scales with the network and does not degrade as the number of agents grows.

---

## Security Guarantees

| Property                | Guarantee                                                                 |
|-------------------------|---------------------------------------------------------------------------|
| Identity Uniqueness     | Guaranteed by quantum entropy irreducibility                              |
| Unforgeability          | Existential unforgeability under adaptive chosen message attack           |
| Replay Resistance       | Nonce inclusion in all signed payloads                                    |
| Quantum Resistance      | Security reducible to Module LWE                                          |
| Verifier Independence   | No CA, no trust anchors, Solana ledger as source of truth                 |
| Forward Security        | Key rotation with cryptographic continuity proofs                         |
| Security Horizon        | Conservative parameter selection targeting 2080                           |

### Threat Model

```
  Threats QEVORA addresses
  [x] Classical adversary with full network access
  [x] Fault-tolerant quantum adversary (post-2030)
  [x] Replay attacks on agent operations
  [x] Identity spoofing
  [x] CA compromise (no CA exists)
  [x] MITM on verification path (math-based, not channel-based)

  Out of scope
  [ ] Physical compromise of agent hardware enclave
  [ ] Supply chain compromise of QRNG hardware
  [ ] Side-channel attacks on signing hardware
  [ ] Social engineering of human operators
```

---

## Getting Started

### Prerequisites

- Rust 1.75 or later (core and node)
- Node.js 20 or later (SDK TypeScript bindings)
- Python 3.11 or later (SDK Python bindings)
- Anchor CLI 0.29 or later (Solana program deployment)
- A certified QRNG hardware source for production deployments

### Install the SDK

```bash
# TypeScript
npm install @qevora/sdk @solana/web3.js @coral-xyz/anchor

# Python
pip install qevora-sdk

# Rust
cargo add qevora-sdk
```

### Provision an Agent Identity

```typescript
import { QevoraClient } from '@qevora/sdk';

const client = new QevoraClient({ network: 'mainnet' });

const identity = await client.identity.provision({
  entropy: await client.qrng.generate(256),
  label: 'my-agent-v1'
});

console.log(identity.agentId);
console.log(identity.publicKey);
```

### Sign an Operation

```typescript
const signed = await client.identity.sign({
  privateKey: identity.privateKey,
  payload: { action: 'transfer', amount: 100, to: 'agent-xyz' },
  nonce: await client.nonce.generate()
});
```

### Verify a Signature

```typescript
const result = await client.identity.verify({
  publicKey: remoteAgentPublicKey,
  signature: signed.signature,
  payload: signed.payload
});

if (result.valid) {
  // proceed
}
```

### Deploy the On-Chain Program

```bash
git clone https://github.com/QEVORAxyz/qevora-node
cd qevora-node
anchor build
anchor deploy --provider.cluster devnet
```

---

## Roadmap

```
  Phase 1 - Foundation (current)
  [x] Protocol specification
  [x] Core cryptographic primitives (Kyber + Dilithium)
  [x] SDK design and API surface
  [ ] TypeScript SDK v0.1
  [ ] Python SDK v0.1
  [ ] Developer documentation

  Phase 2 - Network
  [ ] Solana on-chain program v0.1
  [ ] Identity registry on devnet
  [ ] PDA account structure
  [ ] Signature ledger
  [ ] Anchor instruction handlers

  Phase 3 - Tooling
  [ ] CLI v1.0
  [ ] Key management interface
  [ ] Registry explorer
  [ ] Agent monitoring dashboard

  Phase 4 - Production
  [ ] Security audit (third-party)
  [ ] Mainnet deployment
  [ ] QRNG hardware integration partnerships
  [ ] Enterprise identity onboarding
  [ ] SDK v1.0 stable release

  Phase 5 - Ecosystem
  [ ] Solana dApp integrations
  [ ] Cross-program identity composability
  [ ] Compliance and attestation frameworks
  [ ] Institutional key management support
```

---

## Contributing

Pull requests are welcome. For significant changes, open an issue first to discuss what you want to change.

All cryptographic contributions must include a reference to the relevant NIST publication or academic paper, test vectors from the specification, and independent review by a second contributor before merge.

---

## License

MIT License. See [LICENSE](LICENSE) for details.

---

## Contact

Website: [qevora.xyz](https://qevora.xyz)
Twitter: [@QEVORAxyz](https://twitter.com/QEVORAxyz)
GitHub: [QEVORAxyz](https://github.com/QEVORAxyz)
