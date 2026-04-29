# Verification Model

QEVORA uses a certificate-free, CA-free verification architecture. Verification is a local computation requiring no network calls beyond an initial registry read.

## How Verification Works

1. The verifier fetches the agent's public key from the Solana on-chain registry (one-time read per agent)
2. The verifier caches the public key locally
3. For each signed operation, the verifier runs `Dilithium.verify(public_key, signature, payload)`
4. The result is a binary outcome derived entirely from lattice arithmetic
5. No trusted third party is consulted at any point

## Properties

**No certificate authorities.** There is no root CA, intermediate CA, or certificate chain. Public keys are self-certifying and anchored directly to Solana.

**No revocation servers.** Revocation status is read directly from the on-chain identity account. No OCSP endpoint, no CRL distribution point.

**No expiry.** Identities do not expire. They are valid until explicitly revoked on-chain.

**Offline verification.** Once the public key is fetched, all verification is local. Verifiers in air-gapped environments can verify signatures without any network access.

**Batch verification.** Dilithium supports batch verification, allowing multiple signatures to be verified in a single pass with better amortized performance.

## Comparison to Classical PKI

| Property              | Classical PKI         | QEVORA                      |
|-----------------------|-----------------------|-----------------------------|
| Root of trust         | Certificate authority | Solana ledger               |
| Revocation check      | OCSP / CRL server     | On-chain account read       |
| Quantum resistance    | No (RSA / ECDSA)      | Yes (Module LWE)            |
| Offline verification  | Partial               | Full                        |
| Single point of failure| CA                   | None                        |
| Key expiry            | Yes                   | No                          |

## Verification Code Example

```typescript
import { QevoraClient } from '@qevora/sdk';

const client = new QevoraClient({ network: 'mainnet' });

const publicKey = await client.registry.getPublicKey(agentId);

const result = client.identity.verify({
  publicKey,
  signature: operation.signature,
  payload:   operation.payload,
});

if (!result.valid) {
  throw new Error('Invalid signature');
}
```
