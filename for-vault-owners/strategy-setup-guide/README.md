# Strategy Setup Guide

After creating your vault, you need to set up strategies so funds can be deployed to DeFi protocols. This is a **two-step process**:

1. **Add the adaptor** to your vault (one-time per adaptor program)
2. **Initialize the strategy** for each specific protocol/market you want to deploy to

{% hint style="warning" %}
**Vault creation ≠ Strategy initialization.** A newly created vault has no strategies. Deposited funds will sit idle until you complete the steps in this guide.
{% endhint %}

## Key Concepts

### Adaptors vs. Strategies

* **Adaptor**: An on-chain program that knows how to interact with a category of protocols (e.g., the lending adaptor handles Kamino, Marginfi, Save, etc.)
* **Strategy**: A specific deployment target within an adaptor (e.g., "lend USDC on Kamino lending market")

A vault can have **multiple strategies** across **multiple adaptors**.

### Who Can Do What

| Action | Required Role |
|--------|--------------|
| Add adaptor to vault | Admin |
| Initialize strategy | Admin |
| Allocate funds to strategy | Manager |

## Available Strategies

| Strategy Type | Adaptor | Protocols | Guide |
|--------------|---------|-----------|-------|
| **Lending** | Lending Adaptor | Kamino, Marginfi, Save, Drift Spot, Jupiter Lend | [Lending Strategies](lending-strategies.md) |
| **Drift Perps/JLP** | Drift Adaptor | Drift Protocol | [Drift Strategies](drift-strategies.md) |
| **Raydium CLMM** | Raydium Adaptor | Raydium | [Raydium LP Strategies](raydium-lp-strategies.md) |
| **Off-chain** | Trustful Adaptor | CEX, OTC, MPC | [Trustful Adaptor](trustful-adaptor.md) |

## Step 1: Add Adaptor

Before initializing any strategy, you must add the corresponding adaptor program to your vault. This is a one-time operation per adaptor type.

```typescript
import { VoltrClient } from "@voltr/vault-sdk";
import {
  Connection,
  Keypair,
  PublicKey,
  sendAndConfirmTransaction,
} from "@solana/web3.js";
import fs from "fs";

const connection = new Connection("your-rpc-url");
const client = new VoltrClient(connection);

const adminKp = Keypair.fromSecretKey(
  Uint8Array.from(JSON.parse(fs.readFileSync("/path/to/admin.json", "utf-8")))
);

const vault = new PublicKey("your-vault-pubkey");

// Add adaptor to vault
const addAdaptorIx = await client.createAddAdaptorIx({
  vault,
  admin: adminKp.publicKey,
  payer: adminKp.publicKey,
});

const txSig = await sendAndConfirmTransaction(
  [addAdaptorIx],
  connection,
  [adminKp]
);

console.log("Adaptor added:", txSig);
```

## Step 2: Initialize Strategy

Strategy initialization is protocol-specific. Each protocol requires different accounts and configuration. Use the protocol-specific scripts from the Voltr team:

| Repository | Protocols |
|-----------|----------|
| [voltrxyz/lend-scripts](https://github.com/voltrxyz/lend-scripts) | Project0, Save |
| [voltrxyz/kamino-scripts](https://github.com/voltrxyz/kamino-scripts) | Kamino Vault, Kamino Lending Market |
| [voltrxyz/drift-scripts](https://github.com/voltrxyz/drift-scripts) | Drift Vaults, Drift Lend, Drift Perps |
| [voltrxyz/spot-scripts](https://github.com/voltrxyz/spot-scripts) | Jupiter Swap, Jupiter Lend |
| [voltrxyz/client-raydium-clmm-scripts](https://github.com/voltrxyz/client-raydium-clmm-scripts) | Raydium CLMM Pools |
| [voltrxyz/trustful-scripts](https://github.com/voltrxyz/trustful-scripts) | Centralised Exchanges |

See the individual strategy pages for detailed initialization instructions.

## Lookup Tables

For strategies that require many accounts in a single transaction, you may need to set up Address Lookup Tables (LUTs) to fit within Solana's transaction size limits. See [Lookup Tables (LUT)](lookup-tables.md).

## Next Steps

After initializing strategies:

{% content-ref url="../fund-allocation-guide/README.md" %}
[Fund Allocation Guide](../fund-allocation-guide/README.md)
{% endcontent-ref %}
