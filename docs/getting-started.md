# Getting Started

## Prerequisites

- Rust 1.75 or later
- Node.js 20 or later
- Python 3.11 or later
- Anchor CLI 0.29 or later
- Solana CLI 1.18 or later
- A certified QRNG hardware source (for production; devnet accepts software entropy)

## Install the SDK

```bash
# TypeScript
npm install @qevora/sdk @solana/web3.js @coral-xyz/anchor

# Python
pip install qevora-sdk

# Rust
cargo add qevora-sdk
```

## Provision an Agent Identity

```typescript
import { QevoraClient } from '@qevora/sdk';

const client = new QevoraClient({ network: 'devnet' });

const identity = await client.identity.provision({
  entropy: await client.qrng.generate(256),
  label: 'my-agent-v1'
});

console.log('Agent ID:   ', identity.agentId);
console.log('Public Key: ', identity.publicKey);
```

## Sign an Operation

```typescript
const signed = await client.identity.sign({
  privateKey: identity.privateKey,
  payload: {
    action: 'transfer',
    amount: 100,
    to: 'agent-xyz'
  },
  nonce: await client.nonce.generate()
});
```

## Verify a Signature

```typescript
const publicKey = await client.registry.getPublicKey(signed.agentId);

const result = client.identity.verify({
  publicKey,
  signature: signed.signature,
  payload:   signed.payload
});

if (result.valid) {
  console.log('Signature valid for agent:', result.agentId);
}
```

## Rotate a Key Pair

```typescript
const rotation = await client.identity.rotate({
  agentId:        identity.agentId,
  oldPrivateKey:  identity.privateKey,
  newEntropy:     await client.qrng.generate(256)
});

console.log('New public key: ', rotation.newPublicKey);
```

## Revoke an Identity

```typescript
await client.identity.revoke({
  agentId:    identity.agentId,
  privateKey: identity.privateKey,
  reason:     'key compromise'
});
```

## Deploy the On-Chain Program (Devnet)

```bash
git clone https://github.com/QEVORAxyz/qevora-node
cd qevora-node
anchor build
anchor deploy --provider.cluster devnet
```

## Next Steps

- [Architecture](architecture.md) — understand how the system layers interact
- [Cryptography](cryptography.md) — deep dive into the cryptographic primitives
- [Identity Lifecycle](identity-lifecycle.md) — provisioning, rotation, and revocation in detail
- [Solana Integration](solana-integration.md) — on-chain program and PDA design
- [Verification Model](verification-model.md) — how zero trust verification works
- [Threat Model](threat-model.md) — adversary assumptions and security scope
