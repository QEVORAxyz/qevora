# Solana Integration

QEVORA uses Solana as its on-chain registry and settlement layer. The choice of Solana is driven by throughput, finality speed, and the composability of Anchor-based programs.

## Why Solana

- **Throughput:** 65,000+ TPS supports high-frequency agent transaction volumes
- **Finality:** ~400ms block times provide near-instant identity registration confirmation
- **Cost:** Sub-cent transaction fees make per-operation signing economically viable
- **Composability:** Solana programs can be composed, allowing other protocols to integrate QEVORA identity checks natively
- **Decentralization:** No single point of failure for the identity registry

## On-Chain Program

The QEVORA Solana program is written using the Anchor framework. It manages identity accounts, processes instructions, and enforces the cryptographic rules of the protocol.

### Program ID

Devnet: TBD
Mainnet: TBD

### Accounts

**IdentityAccount**

Stores the agent's public key, status, and metadata. Derived as a PDA from the agent ID.

```
seeds: [b"identity", agent_id]
bump:  canonical
```

**LedgerAccount**

Stores a record of each signed operation for audit purposes. Derived from agent ID and nonce.

```
seeds: [b"ledger", agent_id, nonce]
bump:  canonical
```

### Instructions

| Instruction | Description                                      | Signers Required        |
|-------------|--------------------------------------------------|-------------------------|
| provision   | Register a new agent identity on-chain           | payer                   |
| rotate      | Replace key pair with cryptographic continuity   | agent keypair           |
| revoke      | Permanently deactivate an identity               | threshold of authorities|
| verify      | Record a verified operation in the ledger        | verifier                |

### PDA Derivation

All identity accounts use Program Derived Addresses to ensure deterministic, collision-resistant account locations.

```typescript
const [identityPda] = PublicKey.findProgramAddressSync(
  [Buffer.from("identity"), agentId],
  QEVORA_PROGRAM_ID
);
```

## Transaction Flow

```
  Client
    |
    | build instruction
    v
  qevora-sdk
    |
    | serialize + sign (Dilithium proof included in ix data)
    v
  Solana RPC
    |
    | broadcast
    v
  Validator
    |
    | execute QEVORA program
    v
  IdentityAccount updated on-chain
```

## Devnet Setup

```bash
solana config set --url devnet
anchor build
anchor deploy --provider.cluster devnet
```
