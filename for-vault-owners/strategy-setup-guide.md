# Strategy Setup Guide

After creating your vault, you need to set up strategies so funds can be deployed to DeFi protocols. This is a **two-step process**:

1. **Add the adaptor** to your vault (one-time per adaptor program, you may skip this if you have done so on the UI)
2. **Initialize the strategy** for each specific protocol/market you want to deploy to

{% hint style="warning" %}
**Vault creation ≠ Strategy initialization.** A newly created vault has no strategies. Deposited funds will sit idle until you complete the steps in this guide.
{% endhint %}

## Key Concepts

### Adaptors vs. Strategies

* **Adaptor**: An on-chain program that knows how to interact with a category of protocols (e.g., the Kamino adaptor interacts with Kamino, Drift adaptor interacts with Drift...)
* **Strategy**: A specific deployment target within an adaptor (e.g., "lend USDC on Kamino Main Market")

A vault can have **multiple strategies** across **multiple adaptors**.

## Available Strategies

| Strategy Type       | Adaptor          | Protocols                                        | Guide                                                       |
| ------------------- | ---------------- | ------------------------------------------------ | ----------------------------------------------------------- |
| **Lending**         | Lending Adaptor  | Kamino, Marginfi, Save, Drift Spot, Jupiter Lend | [Lending Strategies](/broken/pages/SU5G0KzJ6ufc1MMkIkZC)    |
| **Drift Perps/JLP** | Drift Adaptor    | Drift Protocol                                   | [Drift Strategies](/broken/pages/F0MxywGDMM08bmC6Asym)      |
| **Raydium CLMM**    | Raydium Adaptor  | Raydium                                          | [Raydium LP Strategies](/broken/pages/7HnxN62s1ZYCKraz5YCt) |
| **Off-chain**       | Trustful Adaptor | CEX, OTC, MPC                                    | [Trustful Adaptor](/broken/pages/eO9jKr35ufjqOQuPAdNi)      |

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

<table><thead><tr><th width="308.1162109375">Protocol / Adaptor</th><th>Initialization Scripts</th></tr></thead><tbody><tr><td>Kamino Adaptor</td><td><a href="https://github.com/voltrxyz/kamino-scripts/blob/main/src/scripts/manager-initialize-kvault.ts">Kamino Vault</a>, <a href="https://github.com/voltrxyz/kamino-scripts/blob/main/src/scripts/manager-initialize-market.ts">Kamino Lending Market</a>, </td></tr><tr><td>Drift Adaptor</td><td><a href="https://github.com/voltrxyz/drift-scripts/blob/main/src/scripts/manager-init-earn.ts">Drift Lend</a>, <a href="https://github.com/voltrxyz/drift-scripts/blob/main/src/scripts/manager-init-user.ts">Drift Perps</a></td></tr><tr><td>Jupiter Adaptor</td><td><a href="https://github.com/voltrxyz/spot-scripts/blob/main/src/scripts/manager-initialize-spot.ts">Spot via Jupiter Swap</a>, <a href="https://github.com/voltrxyz/spot-scripts/blob/main/src/scripts/manager-initialize-earn.ts">Jupiter Lend</a></td></tr><tr><td>Trustful Adaptor</td><td><a href="https://github.com/voltrxyz/trustful-scripts/blob/main/src/scripts/manager-initialize-arbitrary.ts">Centralised Exchanges</a></td></tr></tbody></table>

## Lookup Tables

For strategies that require many accounts in a single transaction, you may need to set up Address Lookup Tables (LUTs) to fit within Solana's transaction size limits. See [Lookup Tables (LUT)](/broken/pages/brLWJ83cVdg0R6ktV0Xa).

{% content-ref url="fund-allocation-guide/" %}
[fund-allocation-guide](fund-allocation-guide/)
{% endcontent-ref %}
