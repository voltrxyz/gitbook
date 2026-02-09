# Quick Start (SDK)

This guide provides a condensed end-to-end walkthrough of setting up a vault using the Voltr SDK. Each step links to its detailed documentation page.

## Prerequisites

```bash
npm install @voltr/vault-sdk @solana/web3.js @coral-xyz/anchor @solana/spl-token
```

```typescript
import { VoltrClient } from "@voltr/vault-sdk";
import { Connection } from "@solana/web3.js";

const connection = new Connection("your-rpc-url");
const client = new VoltrClient(connection);
```

## End-to-End Flow

### 1. Create the Vault

Initialize a new vault with your chosen asset and fee configuration.

```typescript
const createVaultIx = await client.createInitializeVaultIx(
  { config: vaultConfig, name: "My Vault", description: "Short description" },
  {
    vault: vaultKp,
    vaultAssetMint: assetMint,
    admin: adminKp.publicKey,
    manager: managerKp.publicKey,
    payer: adminKp.publicKey,
  }
);
```

→ Full details: [Vault Creation](../vault-initialization-guide/vault-creation.md)

### 2. Set Up LP Token Metadata

Create metadata so wallets display your vault's LP token correctly.

```typescript
const metadataIx = await client.createCreateLpMetadataIx(
  { name: "My Vault LP", symbol: "mvLP", uri: "https://..." },
  { vault: vaultPubkey, admin: adminKp.publicKey, payer: adminKp.publicKey }
);
```

→ Full details: [Vault Token Metadata](../vault-initialization-guide/vault-token-metadata.md)

### 3. Add Adaptors

Add the adaptor program(s) your strategies will use.

```typescript
const addAdaptorIx = await client.createAddAdaptorIx({
  vault: vaultPubkey,
  admin: adminKp.publicKey,
  payer: adminKp.publicKey,
});
```

→ Full details: [Strategy Setup Guide](../strategy-setup-guide/README.md)

### 4. Initialize Strategies

Use protocol-specific scripts to initialize strategies for your vault.

```typescript
const initStrategyIx = await client.createInitializeStrategyIx(
  {},
  {
    payer: adminKp.publicKey,
    vault: vaultPubkey,
    manager: managerKp.publicKey,
    strategy: strategyPda,
    remainingAccounts: [/* protocol-specific accounts */],
  }
);
```

→ Full details: [Lending Strategies](../strategy-setup-guide/lending-strategies.md) | [Drift Strategies](../strategy-setup-guide/drift-strategies.md) | [Raydium LP](../strategy-setup-guide/raydium-lp-strategies.md)

### 5. Allocate Funds

Deploy idle vault funds into your initialized strategies.

```typescript
const depositIx = await client.createDepositStrategyIx(
  { depositAmount },
  {
    manager: managerKp.publicKey,
    vault: vaultPubkey,
    vaultAssetMint: assetMint,
    assetTokenProgram: TOKEN_PROGRAM_ID,
    strategy: strategyPda,
    remainingAccounts: [/* protocol-specific accounts */],
  }
);
```

→ Full details: [Fund Allocation](../fund-allocation-guide/fund-allocation.md)

### 6. Set Up Automation

Deploy bots/scripts to handle ongoing operations:

* **Rebalancing** — move funds between strategies based on yields
* **Rewards claiming** — harvest protocol rewards and swap back to base asset
* **Performance monitoring** — track APY and vault health

→ Full details: [Running Bots & Scripts](../vault-operations/running-bots-and-scripts.md)

### 7. Go to Market

Get your vault indexed and listed on Ranger for user discoverability.

→ Full details: [Indexing & Listing on Ranger](../go-to-market/indexing-and-listing.md)

## Best Practices

| Use Case | Recommended Approach |
|----------|---------------------|
| **Manager actions** (fund allocation, strategy init) | SDK with server-side scripts |
| **User-facing actions** (deposit, withdraw, query) | Voltr API (`api.voltr.xyz`) |
| **Reading vault data** (APY, TVL, share price) | Voltr API |
| **Vault creation & config** | UI or SDK — your preference |

{% hint style="info" %}
**SDK for writes, API for reads**: Use the SDK for manager/admin operations that require signing transactions. Use the [Voltr API](https://api.voltr.xyz/docs) for querying vault data, APY, and share prices — it's simpler and doesn't require keypair access.
{% endhint %}
