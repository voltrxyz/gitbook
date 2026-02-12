# Fund Allocation

This guide explains how to use the Voltr SDK to deposit and withdraw funds between the vault's idle account and strategies.

## Setup

Import the required dependencies:

```typescript
import { Connection, Keypair, PublicKey, TransactionInstruction } from "@solana/web3.js";
import { VoltrClient, LENDING_ADAPTOR_PROGRAM_ID, SEEDS } from "@voltr/vault-sdk";
import { BN } from "@coral-xyz/anchor";
import {
  createAssociatedTokenAccountInstruction,
  getAssociatedTokenAddressSync,
  getAccount,
  TOKEN_PROGRAM_ID,
} from "@solana/spl-token";
import fs from "fs";
```

Initialize the client and configuration:

```typescript
// Load manager keypair
const managerKp = Keypair.fromSecretKey(
  Uint8Array.from(JSON.parse(fs.readFileSync("/path/to/manager.json", "utf-8")))
);
const manager = managerKp.publicKey;

// Initialize connection and client
const connection = new Connection("your-rpc-url");
const client = new VoltrClient(connection);

// Reference vault and asset
const vault = new PublicKey("your-vault-address");
const vaultAssetMint = new PublicKey("your-asset-mint");
```

## Depositing Funds to Strategies

Accounts required for deposit is protocol-specific. Each protocol requires different accounts and configuration. Use the protocol-specific scripts from the Voltr team:

<table><thead><tr><th width="308.1162109375">Protocol / Adaptor</th><th>Deposit Scripts</th></tr></thead><tbody><tr><td>Kamino Adaptor</td><td><a href="https://github.com/voltrxyz/kamino-scripts/blob/main/src/scripts/manager-deposit-kvault.ts">Kamino Vault</a>, <a href="https://github.com/voltrxyz/kamino-scripts/blob/main/src/scripts/manager-deposit-market.ts">Kamino Lending Market</a></td></tr><tr><td>Drift Adaptor</td><td><a href="https://github.com/voltrxyz/drift-scripts/blob/main/src/scripts/manager-deposit-earn.ts">Drift Lend</a>, <a href="https://github.com/voltrxyz/drift-scripts/blob/main/src/scripts/manager-init-user.ts">Drift Perps</a></td></tr><tr><td>Jupiter Adaptor</td><td><a href="https://github.com/voltrxyz/spot-scripts/blob/main/src/scripts/manager-buy-spot.ts">Spot via Jupiter Swap</a>, <a href="https://github.com/voltrxyz/spot-scripts/blob/main/src/scripts/manager-initialize-earn.ts">Jupiter Lend</a></td></tr><tr><td>Trustful Adaptor</td><td><a href="https://github.com/voltrxyz/trustful-scripts/blob/main/src/scripts/manager-deposit-arbitrary.ts">Centralised Exchanges</a></td></tr></tbody></table>

### 1. Account Setup

```typescript
// Get strategy PDA
const counterPartyTa = new PublicKey("protocol-token-account");

const [strategy] = PublicKey.findProgramAddressSync(
  [SEEDS.STRATEGY, counterPartyTa.toBuffer()],
  LENDING_ADAPTOR_PROGRAM_ID
);

// Get vault strategy authority
const { vaultStrategyAuth } = client.findVaultStrategyAddresses(vault, strategy);

// Setup vault strategy asset ATA
const vaultStrategyAssetAta = getAssociatedTokenAddressSync(
  vaultAssetMint,
  vaultStrategyAuth,
  true
);

// Check and create ATA if needed
let transactionIxs: TransactionInstruction[] = [];
try {
  await getAccount(connection, vaultStrategyAssetAta);
} catch {
  transactionIxs.push(
    createAssociatedTokenAccountInstruction(
      manager,
      vaultStrategyAssetAta,
      vaultStrategyAuth,
      vaultAssetMint
    )
  );
}
```

{% hint style="info" %}
**ATA behavior**: ATAs created for strategy operations are **not closed** between deposit/withdraw cycles. The rent cost (\~0.002 SOL per ATA) is a one-time cost paid by the manager. See [Gas Fees & ATA Costs](/broken/pages/U6iBbWz0O1yu70lcZW1C) for details.
{% endhint %}

### 2. Create Deposit Instruction

```typescript
const depositAmount = new BN("1000000"); // Amount in smallest unit (e.g., 1 USDC = 1000000)

const depositIx = await client.createDepositStrategyIx(
  { depositAmount },
  {
    manager,
    vault,
    vaultAssetMint,
    assetTokenProgram: TOKEN_PROGRAM_ID,
    strategy,
    remainingAccounts: [
      { pubkey: counterPartyTa, isSigner: false, isWritable: true },
      { pubkey: protocolProgram, isSigner: false, isWritable: false },
      // Additional protocol-specific accounts...
    ],
  }
);

transactionIxs.push(depositIx);
```

### 3. Send Transaction

```typescript
const txSig = await sendAndConfirmOptimisedTx(
  transactionIxs,
  "your-rpc-url",
  managerKp
);
```

## Withdrawing Funds from Strategies

Accounts required for withdraw is protocol-specific. Each protocol requires different accounts and configuration. Use the protocol-specific scripts from the Voltr team:

<table><thead><tr><th width="308.1162109375">Protocol / Adaptor</th><th>Withdraw Scripts</th></tr></thead><tbody><tr><td>Kamino Adaptor</td><td><a href="https://github.com/voltrxyz/kamino-scripts/blob/main/src/scripts/manager-withdraw-kvault.ts">Kamino Vault</a>, <a href="https://github.com/voltrxyz/kamino-scripts/blob/main/src/scripts/manager-deposit-market.ts">Kamino Lending Market</a></td></tr><tr><td>Drift Adaptor</td><td><a href="https://github.com/voltrxyz/drift-scripts/blob/main/src/scripts/manager-init-earn.ts">Drift Lend</a>, <a href="https://github.com/voltrxyz/drift-scripts/blob/main/src/scripts/manager-init-user.ts">Drift Perps</a></td></tr><tr><td>Jupiter Adaptor</td><td><a href="https://github.com/voltrxyz/spot-scripts/blob/main/src/scripts/manager-buy-spot.ts">Spot via Jupiter Swap</a>, <a href="https://github.com/voltrxyz/spot-scripts/blob/main/src/scripts/manager-initialize-earn.ts">Jupiter Lend</a></td></tr><tr><td>Trustful Adaptor</td><td><a href="https://github.com/voltrxyz/trustful-scripts/blob/main/src/scripts/manager-deposit-arbitrary.ts">Centralised Exchanges</a></td></tr></tbody></table>

### 1. Account Setup

```typescript
// Get counterparty authority
const counterPartyTaAuth = await getAccount(
  connection,
  counterPartyTa,
  "confirmed"
).then((account) => account.owner);

// Setup ATA if needed (same as deposit)
let transactionIxs: TransactionInstruction[] = [];
try {
  await getAccount(connection, vaultStrategyAssetAta);
} catch {
  transactionIxs.push(
    createAssociatedTokenAccountInstruction(
      manager,
      vaultStrategyAssetAta,
      vaultStrategyAuth,
      vaultAssetMint
    )
  );
}
```

### 2. Create Withdrawal Instruction

```typescript
const withdrawAmount = new BN("500000"); // Amount to withdraw

const withdrawIx = await client.createWithdrawStrategyIx(
  { withdrawAmount },
  {
    manager,
    vault,
    vaultAssetMint,
    assetTokenProgram: TOKEN_PROGRAM_ID,
    strategy,
    remainingAccounts: [
      { pubkey: counterPartyTaAuth, isSigner: false, isWritable: true },
      { pubkey: counterPartyTa, isSigner: false, isWritable: true },
      { pubkey: protocolProgram, isSigner: false, isWritable: false },
      // Additional protocol-specific accounts...
    ],
  }
);

transactionIxs.push(withdrawIx);
```

### 3. Send Transaction

```typescript
const txSig = await sendAndConfirmOptimisedTx(
  transactionIxs,
  "your-rpc-url",
  managerKp
);
```

## Best Practices

* **Keep idle reserves**: Don't deploy 100% of funds — leave a buffer for user withdrawals
* **Batch operations**: Combine ATA creation and allocation in a single transaction
* **Monitor allocations**: Track how funds are distributed across strategies
* **Automate**: Use bots/scripts for regular rebalancing (see [Running Bots & Scripts](../vault-operations/running-bots-and-scripts.md))

## Troubleshooting

| Issue                 | Solution                                         |
| --------------------- | ------------------------------------------------ |
| Transaction too large | Use Lookup Tables                                |
| Insufficient funds    | Check idle balance, ensure enough SOL for gas    |
| Authority error       | Verify manager keypair matches vault's manager   |
| ATA not found         | Create the ATA before the allocation instruction |

For additional support, refer to the [Voltr SDK documentation](https://voltrxyz.github.io/vault-sdk/).
