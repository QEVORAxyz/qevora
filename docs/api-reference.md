# API Reference

## QevoraClient

Main entry point for all SDK operations.

```typescript
new QevoraClient(config: ClientConfig)
```

**ClientConfig**

| Field     | Type                          | Description                        |
|-----------|-------------------------------|------------------------------------|
| network   | 'mainnet' \| 'devnet'         | Solana network to connect to       |
| rpcUrl    | string (optional)             | Custom RPC endpoint                |
| commitment| 'confirmed' \| 'finalized'    | Solana commitment level            |

---

## identity.provision

Provision a new agent identity from quantum entropy.

```typescript
client.identity.provision(params: ProvisionParams): Promise<Identity>
```

**ProvisionParams**

| Field   | Type     | Description                          |
|---------|----------|--------------------------------------|
| entropy | Uint8Array | 256-bit quantum entropy seed       |
| label   | string   | Human-readable label for the identity|

**Returns: Identity**

| Field      | Type       | Description                          |
|------------|------------|--------------------------------------|
| agentId    | string     | Unique agent identifier (hex)        |
| publicKey  | Uint8Array | Dilithium-3 public key               |
| privateKey | Uint8Array | Dilithium-3 private key (keep secret)|
| createdAt  | number     | Unix timestamp                       |

---

## identity.sign

Sign an operation payload with an agent's private key.

```typescript
client.identity.sign(params: SignParams): Promise<SignedOperation>
```

**SignParams**

| Field      | Type       | Description                    |
|------------|------------|--------------------------------|
| privateKey | Uint8Array | Agent private key              |
| payload    | object     | Operation data to sign         |
| nonce      | Uint8Array | 32-byte unique nonce           |

**Returns: SignedOperation**

| Field     | Type       | Description                          |
|-----------|------------|--------------------------------------|
| payload   | object     | Original operation payload           |
| signature | Uint8Array | Dilithium-3 signature (3293 bytes)   |
| agentId   | string     | Signing agent identifier             |
| nonce     | Uint8Array | Nonce used in this signature         |
| timestamp | number     | Unix timestamp at signing            |

---

## identity.verify

Verify a signed operation against an agent's public key.

```typescript
client.identity.verify(params: VerifyParams): VerifyResult
```

**VerifyParams**

| Field     | Type       | Description                    |
|-----------|------------|--------------------------------|
| publicKey | Uint8Array | Agent public key               |
| signature | Uint8Array | Signature to verify            |
| payload   | object     | Original operation payload     |

**Returns: VerifyResult**

| Field      | Type    | Description                          |
|------------|---------|--------------------------------------|
| valid      | boolean | True if signature is valid           |
| agentId    | string  | Agent identifier derived from key    |
| verifiedAt | number  | Unix timestamp of verification       |

---

## identity.rotate

Rotate an agent's key pair with cryptographic continuity proof.

```typescript
client.identity.rotate(params: RotateParams): Promise<RotationResult>
```

**RotateParams**

| Field         | Type       | Description                    |
|---------------|------------|--------------------------------|
| agentId       | string     | Agent identifier to rotate     |
| oldPrivateKey | Uint8Array | Current private key            |
| newEntropy    | Uint8Array | 256-bit quantum entropy seed   |

**Returns: RotationResult**

| Field        | Type       | Description                          |
|--------------|------------|--------------------------------------|
| newPublicKey | Uint8Array | Replacement public key               |
| rotationProof| Uint8Array | Signed proof of key continuity       |
| rotationCount| number     | Total rotations for this identity    |

---

## identity.revoke

Permanently revoke an agent identity on-chain.

```typescript
client.identity.revoke(params: RevokeParams): Promise<void>
```

**RevokeParams**

| Field      | Type   | Description                    |
|------------|--------|--------------------------------|
| agentId    | string | Agent identifier to revoke     |
| privateKey | Uint8Array | Agent private key          |
| reason     | string | Revocation reason (logged)     |

---

## registry.getPublicKey

Fetch an agent's public key from the Solana on-chain registry.

```typescript
client.registry.getPublicKey(agentId: string): Promise<Uint8Array>
```

---

## registry.getIdentity

Fetch full identity details from the on-chain registry.

```typescript
client.registry.getIdentity(agentId: string): Promise<IdentityRecord>
```

**Returns: IdentityRecord**

| Field         | Type    | Description                          |
|---------------|---------|--------------------------------------|
| agentId       | string  | Agent identifier                     |
| publicKey     | Uint8Array | Current public key                |
| status        | 'active' \| 'revoked' | Identity status        |
| createdAt     | number  | Provisioning timestamp               |
| rotationCount | number  | Number of key rotations performed    |

---

## qrng.generate

Generate quantum entropy from the configured QRNG source.

```typescript
client.qrng.generate(bits: number): Promise<Uint8Array>
```

---

## nonce.generate

Generate a cryptographically unique nonce for operation signing.

```typescript
client.nonce.generate(): Promise<Uint8Array>
```
