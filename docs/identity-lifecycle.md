# Identity Lifecycle

Every QEVORA agent identity passes through four states: provisioned, active, rotated, and revoked.

## 1. Provisioning

Provisioning creates a new agent identity from scratch. It requires a quantum entropy seed and produces a key pair registered on Solana.

**Steps:**

1. Request 256+ bits of entropy from a certified QRNG source
2. Validate entropy quality against NIST SP 800-90B
3. Derive Kyber-1024 key pair from validated entropy
4. Derive Dilithium-3 signing key pair from the same entropy seed
5. Compute agent identifier as `SHA3-256(public_key || timestamp)`
6. Register public key and agent ID as a Solana PDA
7. Return agent ID and public key to caller
8. Store private key in enclave, never expose externally

**On-chain record created:**

```
IdentityAccount {
    agent_id:    [u8; 32],
    public_key:  [u8; 1952],
    created_at:  i64,
    status:      Active,
    rotation_count: u64,
}
```

## 2. Active

An active identity signs all agent operations using Dilithium-3. Each signed operation includes a nonce to prevent replay attacks.

**Signed payload structure:**

```
SignedOperation {
    payload:    Vec<u8>,
    agent_id:   [u8; 32],
    nonce:      [u8; 32],
    timestamp:  i64,
    signature:  [u8; 3293],
}
```

There is no expiry on an active identity. An identity remains active until explicitly rotated or revoked.

## 3. Rotation

Key rotation replaces an agent's key pair while maintaining cryptographic continuity. The old private key signs a rotation proof, which is verified on-chain before the new public key is accepted.

**Steps:**

1. Generate new key pair from fresh quantum entropy
2. Sign rotation proof: `old_private_key.sign(new_public_key || agent_id || nonce)`
3. Submit rotation transaction to Solana with rotation proof
4. On-chain program verifies proof against current public key
5. Registry updates to new public key, increments rotation count
6. Old private key is securely erased

Rotation preserves the agent ID. All historical signatures issued under previous keys remain verifiable using the key that was active at the time of signing.

## 4. Revocation

Revocation permanently deactivates an agent identity. It requires a threshold of authorized signers to co-sign the revocation transaction on-chain.

**On-chain state after revocation:**

```
IdentityAccount {
    ...
    status:       Revoked,
    revoked_at:   i64,
}
```

Revoked identities cannot sign new operations. Historical signatures issued before revocation remain verifiable. Revocation is irreversible.
