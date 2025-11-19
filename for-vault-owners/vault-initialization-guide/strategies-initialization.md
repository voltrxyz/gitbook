# Strategies Initialization

## Voltr Protocol - Strategy Integration Guide

This guide explains how to add adaptors and initialize strategies in the Voltr Protocol. It covers the process of adding adaptors to vaults and managing strategy lifecycles.

### Setup

Import the required dependencies:

```typescript
import { BN } from "@coral-xyz/anchor";
import { VoltrClient, SEEDS, DEFAULT_ADAPTOR_PROGRAM_ID} from "@voltr/vault-sdk";
import {
  Connection,
  Keypair,
  PublicKey,
  sendAndConfirmTransaction,
} from "@solana/web3.js";
```

Vault owners can integrate with the default adaptor or custom adaptors created by DeFi teams.

### Adding Adaptor

1. Define Required Variables:

```typescript
// File paths for keypairs
const adminFilePath = "/path/to/admin.json";

// Network and asset configuration 
const assetMintAddress = "..."; // Your asset token mint
const solanaRpcUrl = "your-solana-rpc-url";

// Load keypairs
const adminKp = Keypair.fromSecretKey(
  Uint8Array.from(JSON.parse(fs.readFileSync(adminFilePath, "utf-8")))
);

// Reference your vault
const vault = new PublicKey("previously-initialized-vault");

// Initialize client
const connection = new Connection(solanaRpcUrl);
const client = new VoltrClient(connection);
```

2. Create Add Adaptor Instruction:

```typescript
const createAddAdaptorIx = await client.createAddAdaptorIx({
  vault,
  admin: adminKp.publicKey,
  payer: adminKp.publicKey,
});
```

3. Send and Confirm Instruction:

```typescript
const txSig = await sendAndConfirmTransaction(
  [createAddAdaptorIx],
  connection,
  [adminKp]
);
```

### Initializing Strategies

1. Derive Strategy Address:

```typescript
// the counterParty's token account the vault will be transferring to
const counterPartyTa = new PublicKey("...");

const [strategy] = PublicKey.findProgramAddressSync(
  [SEEDS.STRATEGY, counterPartyTa.toBuffer()],
  DEFAULT_ADAPTOR_PROGRAM_ID
);
```

2. Create Strategy Initialization Instruction:

```typescript
const createInitializeStrategyIx = await client.createInitializeStrategyIx(
  {}, // Optional initialization args
  {
    payer,
    vault,
    manager: payer,
    strategy,
    remainingAccounts: [
      // Protocol-specific accounts required for initialization
      { pubkey: protocolProgram, isSigner: false, isWritable: false },
      // Additional required accounts...
    ],
  }
);
```

3. Send and Confirm Transaction:

```typescript
const txSig = await sendAndConfirmTransaction(
  [createInitializeStrategyIx],
  connection,
  [adminKp]
);
```

### Required Account Structure

The strategy initialization requires several key accounts:

1. Core Accounts:
   * `payer`: Account paying for the transaction
   * `vault`: The initialized vault
   * `manager`: Vault manager authority
   * `strategy`: Derived strategy PDA
   * `protocolProgram`: Target protocol program ID
2. Protocol-Specific Accounts:
   * Protocol program account
   * Any required protocol state accounts
   * Token accounts and authorities
   * System accounts (RENT, etc.)

### Best Practices

1. **Strategy Selection**:
   * Choose strategies that match your vault's risk profile
   * Understand the underlying protocol's mechanisms
   * Verify strategy program compatibility
2. **Risk Management**:
   * Start with small deposits to test strategy
   * Monitor strategy performance regularly
   * Have a withdrawal plan for emergencies
3. **Security**:
   * Keep admin keys secure
   * Test on devnet first
   * Verify all account permissions
   * Double-check program IDs
4. **Operations**:
   * Always check strategy balance before removal
   * Maintain accurate records of deployed assets
   * Monitor gas costs for operations

### Troubleshooting

1. **Strategy Creation Fails**:
   * Verify admin authority
   * Check counterparty token account exists
   * Ensure protocol program ID is correct
2. **Strategy Addition Fails**:
   * Verify vault admin authority
   * Check strategy account exists
   * Ensure adaptor program matches
3. **Strategy Removal Fails**:
   * Verify strategy has zero balance
   * Check admin authority
   * Ensure all funds are withdrawn
4. **Initialization Fails**:
   * Check account permissions
   * Verify rent exemption
   * Validate program IDs

For additional support or questions, refer to the [Voltr SDK documentation](https://voltrxyz.github.io/vault-sdk/) or [example scripts](https://github.com/voltrxyz/client-scripts).
