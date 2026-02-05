# Fund Allocation

## Voltr Protocol - Deposit & Withdrawal Technical Guide

This technical guide explains how to use the Voltr SDK to deposit and withdraw funds from strategies. It includes complete code examples and important technical considerations.

### Setup

First, import the required dependencies:

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
const managerKpFile = fs.readFileSync(managerFilePath, "utf-8");
const managerKpData = JSON.parse(managerKpFile);
const managerSecret = Uint8Array.from(managerKpData);
const managerKp = Keypair.fromSecretKey(managerSecret);
const manager = managerKp.publicKey;

// Initialize connection and client
const connection = new Connection(rpcUrl);
const client = new VoltrClient(connection);

// Reference vault and asset
const vault = new PublicKey("your-vault-address");
const vaultAssetMint = new PublicKey("your-asset-mint");
```

### Depositing Funds to Strategies

1. **Account Setup**:

```typescript
// Get strategy PDA
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
if (!vaultStrategyAssetAtaAccount) {
  const createVaultStrategyAssetAtaIx = createAssociatedTokenAccountInstruction(
    payer,
    vaultStrategyAssetAta,
    vaultStrategyAuth,
    vaultAssetMint
  );
  transactionIxs.push(createVaultStrategyAssetAtaIx);
}
```

2. **Create Deposit Instruction**:

```typescript
const createDepositStrategyIx = await client.createDepositStrategyIx(
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

transactionIxs.push(createDepositStrategyIx);
```

3. **Send and Confirm Transaction**:

```typescript
const txSig = await sendAndConfirmOptimisedTx(
  transactionIxs,
  rpcUrl,
  managerKp
);
```

### Withdrawing Funds from Strategies

1. **Account Setup**:

```typescript
// Setup is similar to deposit, plus get counterparty authority
const counterPartyTaAuth = await getAccount(
  connection,
  counterPartyTa,
  "confirmed"
).then((account) => account.owner);

// Setup vault strategy asset ATA if needed
let transactionIxs: TransactionInstruction[] = [];
if (!vaultStrategyAssetAtaAccount) {
  const createVaultStrategyAssetAtaIx = createAssociatedTokenAccountInstruction(
    payer,
    vaultStrategyAssetAta,
    vaultStrategyAuth,
    vaultAssetMint
  );
  transactionIxs.push(createVaultStrategyAssetAtaIx);
}
```

2. **Create Withdrawal Instruction**:

```typescript
const createWithdrawStrategyIx = await client.createWithdrawStrategyIx(
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

transactionIxs.push(createWithdrawStrategyIx);
```

3. **Send and Confirm Transaction**:

```typescript
const txSig = await sendAndConfirmOptimisedTx(
  transactionIxs,
  rpcUrl,
  managerKp
);
```

### Required Account Structure

1. **Core Accounts**:
   * `manager`: Strategy manager authority
   * `vault`: The initialized vault
   * `vaultAssetMint`: Vault asset mint
   * `strategy`: Target strategy PDA
   * `assetTokenProgram`: Token program ID
2. **Associated Token Accounts**:
   * `vaultStrategyAssetAta`: Strategy's asset ATA
   * `counterPartyTa`: Protocol's token account
   * `counterPartyTaAuth`: Protocol's token account authority
3. **Protocol-Specific Accounts**:
   * Protocol program
   * Oracle accounts
   * Market accounts
   * State accounts

### Best Practices

1. **Account Management**:
   * Always check ATAs before transactions
   * Create missing accounts at start
   * Verify account ownership
   * Double-check authority permissions
2. **Transaction Optimization**:
   * Batch related instructions
   * Use optimized transaction sending
   * Include proper compute budget
   * Handle rate limiting
3. **Security**:
   * Verify manager authority
   * Validate all accounts
   * Check protocol states
   * Monitor transaction status

### Troubleshooting

1. **Transaction Failure**:
   * Check account permissions
   * Verify ATA initialization
   * Validate amount calculations
   * Check protocol limits
2. **Authority Errors**:
   * Confirm manager keypair
   * Verify protocol authorities
   * Check PDA derivation
   * Validate signatures
3. **Balance Issues**:
   * Check idle funds
   * Verify available funds
   * Account for fees
   * Consider protocol restrictions

For additional support or questions, refer to the [Voltr SDK documentation](https://voltrxyz.github.io/vault-sdk/) or [example scripts](https://github.com/voltrxyz/client-scripts).
