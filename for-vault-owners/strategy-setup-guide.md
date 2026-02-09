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

## Adaptor Program IDs

Each adaptor has a unique on-chain program ID. You'll need these when adding adaptors and initializing strategies.

```typescript
import {
  LENDING_ADAPTOR_PROGRAM_ID,
  DRIFT_ADAPTOR_PROGRAM_ID,
} from "@voltr/vault-sdk";
```

| Adaptor | Program ID |
|---------|-----------|
| **Lending Adaptor** | `aVoLTRCRt3NnnchvLYH6rMYehJHwM5m45RmLBZq7PGz` |
| **Drift Adaptor** | `EBN93eXs5fHGBABuajQqdsKRkCgaqtJa8vEFD6vKXiP` |
| **Kamino Adaptor** | `to6Eti9CsC5FGkAtqiPphvKD2hiQiLsS8zWiDBqBPKR` |

{% hint style="info" %}
`LENDING_ADAPTOR_PROGRAM_ID` and `DRIFT_ADAPTOR_PROGRAM_ID` are exported directly from the SDK. For other adaptors (Kamino, Jupiter, Raydium, Trustful), check the respective script repositories for their program IDs.
{% endhint %}

## Available Strategies

| Strategy Type       | Adaptor          | Protocols                                        | Guide                                                       |
| ------------------- | ---------------- | ------------------------------------------------ | ----------------------------------------------------------- |
| **Lending**         | Lending Adaptor  | Kamino, Marginfi, Save, Drift Spot, Jupiter Lend | [Lending Strategies](/broken/pages/SU5G0KzJ6ufc1MMkIkZC)    |
| **Drift Perps/JLP** | Drift Adaptor    | Drift Protocol                                   | [Drift Strategies](/broken/pages/F0MxywGDMM08bmC6Asym)      |
| **Raydium CLMM**    | Raydium Adaptor  | Raydium                                          | [Raydium LP Strategies](/broken/pages/7HnxN62s1ZYCKraz5YCt) |
| **Off-chain**       | Trustful Adaptor | CEX, OTC, MPC                                    | [Trustful Adaptor](/broken/pages/eO9jKr35ufjqOQuPAdNi)      |

## Step 1: Add Adaptor

Before initializing any strategy, you must add the corresponding adaptor program to your vault. This is a one-time operation per adaptor type. You must pass the `adaptorProgram` parameter specifying which adaptor to add.

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
const adaptorProgramId = new PublicKey("adaptor-program-id"); // See Adaptor Program IDs table above

// Add adaptor to vault
const addAdaptorIx = await client.createAddAdaptorIx({
  vault,
  admin: adminKp.publicKey,
  payer: adminKp.publicKey,
  adaptorProgram: adaptorProgramId,
});

const txSig = await sendAndConfirmTransaction(
  [addAdaptorIx],
  connection,
  [adminKp]
);

console.log("Adaptor added:", txSig);
```

## Step 2: Initialize Strategy

Strategy initialization is protocol-specific — each protocol requires different remaining accounts and an `instructionDiscriminator` that tells the adaptor which operation to perform. You must also pass the `adaptorProgram` in the accounts.

### Generic Code Snippet

```typescript
import { VoltrClient } from "@voltr/vault-sdk";
import { Connection, Keypair, PublicKey } from "@solana/web3.js";

const connection = new Connection("your-rpc-url");
const client = new VoltrClient(connection);

const adminKp = Keypair.fromSecretKey(/* ... */);
const managerKp = Keypair.fromSecretKey(/* ... */);
const vault = new PublicKey("your-vault-pubkey");
const strategy = new PublicKey("strategy-pda"); // Protocol-specific strategy address
const adaptorProgram = new PublicKey("adaptor-program-id");

// The instruction discriminator is protocol-specific (8 bytes)
// Each adaptor defines its own discriminators for init, deposit, withdraw, etc.
const instructionDiscriminator = Buffer.from([/* 8-byte discriminator */]);

const initStrategyIx = await client.createInitializeStrategyIx(
  {
    instructionDiscriminator,
  },
  {
    payer: adminKp.publicKey,
    manager: managerKp.publicKey,
    vault,
    strategy,
    adaptorProgram,
    remainingAccounts: [
      // Protocol-specific accounts required for initialization
      // e.g., lending market, reserve, obligation, program IDs, etc.
    ],
  }
);

const txSig = await sendAndConfirmTransaction(
  [initStrategyIx],
  connection,
  [adminKp]
);

console.log("Strategy initialized:", txSig);
```

{% hint style="info" %}
The `instructionDiscriminator`, `strategy` address, and `remainingAccounts` are all protocol-specific. Use the initialization scripts from the protocol repositories below as reference implementations.
{% endhint %}

### Protocol-Specific Initialization Scripts

<table><thead><tr><th width="308.1162109375">Protocol / Adaptor</th><th>Initialization Scripts</th></tr></thead><tbody><tr><td>Kamino Adaptor</td><td><a href="https://github.com/voltrxyz/kamino-scripts/blob/main/src/scripts/manager-initialize-kvault.ts">Kamino Vault</a>, <a href="https://github.com/voltrxyz/kamino-scripts/blob/main/src/scripts/manager-initialize-market.ts">Kamino Lending Market</a>, </td></tr><tr><td>Drift Adaptor</td><td><a href="https://github.com/voltrxyz/drift-scripts/blob/main/src/scripts/manager-init-earn.ts">Drift Lend</a>, <a href="https://github.com/voltrxyz/drift-scripts/blob/main/src/scripts/manager-init-user.ts">Drift Perps</a></td></tr><tr><td>Jupiter Adaptor</td><td><a href="https://github.com/voltrxyz/spot-scripts/blob/main/src/scripts/manager-initialize-spot.ts">Spot via Jupiter Swap</a>, <a href="https://github.com/voltrxyz/spot-scripts/blob/main/src/scripts/manager-initialize-earn.ts">Jupiter Lend</a></td></tr><tr><td>Trustful Adaptor</td><td><a href="https://github.com/voltrxyz/trustful-scripts/blob/main/src/scripts/manager-initialize-arbitrary.ts">Centralised Exchanges</a></td></tr></tbody></table>

## Lookup Tables

For strategies that require many accounts in a single transaction, you may need to set up Address Lookup Tables (LUTs) to fit within Solana's transaction size limits. See [Lookup Tables (LUT)](/broken/pages/brLWJ83cVdg0R6ktV0Xa).

{% content-ref url="fund-allocation-guide/" %}
[fund-allocation-guide](fund-allocation-guide/)
{% endcontent-ref %}
