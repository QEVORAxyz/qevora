# Cryptography

## Primitive Selection

QEVORA uses exclusively NIST-standardized post-quantum primitives. All selections were finalized in the NIST Post-Quantum Cryptography standardization process completed in 2024.

| Primitive          | Algorithm            | Standard      | Security Level |
|--------------------|----------------------|---------------|----------------|
| Key Encapsulation  | CRYSTALS-Kyber-1024  | NIST FIPS 203 | Level 5        |
| Digital Signatures | CRYSTALS-Dilithium-3 | NIST FIPS 204 | Level 3        |
| Hashing            | SHA-3 / SHAKE-256    | NIST FIPS 202 | 256-bit        |
| Entropy            | QRNG hardware        | ISO 20543     | Stochastic     |

## CRYSTALS-Kyber

Kyber is a key encapsulation mechanism based on the hardness of Module Learning With Errors. QEVORA uses the Kyber-1024 parameter set, which targets NIST security level 5 (equivalent to AES-256 against classical adversaries, and resistant to known quantum attacks).

Kyber is used in QEVORA for agent key pair generation and key material encapsulation during identity provisioning.

**Parameters (Kyber-1024)**

| Parameter       | Value      |
|-----------------|------------|
| Security level  | NIST L5    |
| Public key      | 1568 bytes |
| Ciphertext      | 1568 bytes |
| Shared secret   | 32 bytes   |

## CRYSTALS-Dilithium

Dilithium is a digital signature scheme based on Module LWE and Module SIS. QEVORA uses Dilithium-3, which provides existential unforgeability under adaptive chosen message attack. This means an adversary who can request signatures on arbitrary messages of their choosing cannot produce a valid signature on any message that was not previously signed.

Dilithium is used in QEVORA for all agent operation signing and identity rotation proofs.

**Parameters (Dilithium-3)**

| Parameter       | Value      |
|-----------------|------------|
| Security level  | NIST L3    |
| Public key      | 1952 bytes |
| Private key     | 4000 bytes |
| Signature       | 3293 bytes |

## Module LWE Hardness

Both Kyber and Dilithium reduce to the Module Learning With Errors problem. No known classical or quantum algorithm solves Module LWE in subexponential time. Specifically, Shor's algorithm (which breaks RSA and elliptic curve cryptography on a quantum computer) has no analogue for lattice problems.

The security reductions are tight, meaning the concrete security of the schemes is well understood and does not rely on unproven conjectures beyond Module LWE hardness.

## Entropy Requirements

A minimum of 256 bits of quantum entropy is required per identity provisioning operation. Entropy is sourced from certified QRNG hardware and validated against NIST SP 800-90B before use. Software PRNGs are not permitted in the provisioning path under any circumstances.
