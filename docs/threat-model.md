# Threat Model

## Adversary Assumptions

QEVORA is designed to remain secure against two classes of adversary.

**Classical adversary:** A computationally unbounded classical computer with full network access, the ability to intercept and replay messages, and access to all public protocol information.

**Quantum adversary:** A fault-tolerant quantum computer capable of running Shor's algorithm and Grover's algorithm at scale. This adversary can break RSA, ECDSA, and Diffie-Hellman in polynomial time.

QEVORA is designed to resist both.

## Threats Addressed

**Identity forgery.** An adversary cannot produce a valid agent identity without access to a certified QRNG source and the corresponding private key. Identity uniqueness is bounded by physical law, not computational hardness.

**Signature forgery.** CRYSTALS-Dilithium provides existential unforgeability under adaptive chosen message attack. An adversary who can request signatures on arbitrary messages cannot forge a valid signature on any new message.

**Replay attacks.** Every signed operation includes a unique nonce. The on-chain ledger records used nonces. A replayed operation is rejected by the program.

**CA compromise.** There is no certificate authority in QEVORA. Compromising a CA has no effect on the protocol.

**Quantum cryptanalysis.** Security reductions to Module LWE ensure that a fault-tolerant quantum computer running Shor's algorithm cannot break Kyber or Dilithium. No known quantum algorithm provides a meaningful speedup against Module LWE.

**Registry tampering.** The Solana ledger is append-only and finalized by Tower BFT consensus. An adversary cannot modify historical identity records without controlling a supermajority of stake.

**MITM on verification.** Verification is a local mathematical computation against a locally cached public key. A man-in-the-middle cannot alter the result of a Dilithium verification.

## Threats Outside Scope

**Physical enclave compromise.** If an adversary gains physical access to the hardware running the agent's signing enclave and extracts the private key, QEVORA cannot prevent misuse. Key rotation should be performed immediately upon suspected hardware compromise.

**QRNG supply chain attacks.** QEVORA assumes the QRNG hardware source is trustworthy. A compromised QRNG that produces predictable output could weaken identity uniqueness. Only hardware certified against ISO 20543 and NIST SP 800-90B should be used in production.

**Side-channel attacks.** Timing attacks, power analysis, and electromagnetic side channels against the signing hardware are outside the scope of the software protocol.

**Social engineering.** Operators who are deceived into revoking or rotating legitimate identities are outside the threat model.

## Security Horizon

QEVORA targets a security horizon of 2080. Parameter selections for Kyber-1024 and Dilithium-3 are conservative relative to current best known attacks, providing substantial margin for algorithmic advances in lattice cryptanalysis over the coming decades.
