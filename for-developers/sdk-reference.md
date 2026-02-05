# SDK Reference

Complete reference for the Voltr Vault SDK (`@voltr/vault-sdk`).

## Installation

```bash
npm install @voltr/vault-sdk
# or
yarn add @voltr/vault-sdk
```

## Client Initialization

```typescript
import { VoltrClient } from "@voltr/vault-sdk";
import { Connection, Keypair } from "@solana/web3.js";

const connection = new Connection("https://api.mainnet-beta.solana.com");

// Basic initialization (for read-only operations)
const client = new VoltrClient(connection);

// With wallet (for signing transactions)
const wallet = Keypair.fromSecretKey(/* your secret key */);
const client = new VoltrClient(connection, wallet);
```

---

## Constants

```typescript
import {
  VAULT_PROGRAM_ID,
  LENDING_ADAPTOR_PROGRAM_ID,
  DRIFT_ADAPTOR_PROGRAM_ID,
  METADATA_PROGRAM_ID,
  SEEDS,
} from "@voltr/vault-sdk";

// Program IDs
VAULT_PROGRAM_ID        // vVoLTRjQmtFpiYoegx285Ze4gsLJ8ZxgFKVcuvmG1a8
LENDING_ADAPTOR_PROGRAM_ID  // aVoLTRCRt3NnnchvLYH6rMYehJHwM5m45RmLBZq7PGz
DRIFT_ADAPTOR_PROGRAM_ID    // EBN93eXs5fHGBABuajQqdsKRkCgaqtJa8vEFD6vKXiP
METADATA_PROGRAM_ID     // metaqbxxUerdq28cj1RbAWkYQm3ybzjb6a8bt518x1s

// PDA Seeds
SEEDS.VAULT_LP_MINT
SEEDS.VAULT_ASSET_IDLE_AUTH
SEEDS.STRATEGY_INIT_RECEIPT
SEEDS.VAULT_STRATEGY_AUTH
SEEDS.ADAPTOR_ADD_RECEIPT
SEEDS.DIRECT_WITHDRAW_INIT_RECEIPT_SEED
SEEDS.REQUEST_WITHDRAW_VAULT_RECEIPT
SEEDS.STRATEGY
SEEDS.METADATA
```

---

## Types

### VaultConfig

Configuration for initializing or updating a vault:

```typescript
interface VaultConfig {
  maxCap: BN;                              // Maximum vault capacity (0 = unlimited)
  startAtTs: BN;                           // Activation timestamp (0 = immediate)
  lockedProfitDegradationDuration: BN;     // Locked profit degradation in seconds
  managerManagementFee: number;            // Manager's management fee (basis points)
  managerPerformanceFee: number;           // Manager's performance fee (basis points)
  adminManagementFee: number;              // Admin's management fee (basis points)
  adminPerformanceFee: number;             // Admin's performance fee (basis points)
  redemptionFee: number;                   // Redemption fee (basis points)
  issuanceFee: number;                     // Issuance fee (basis points)
  withdrawalWaitingPeriod: BN;             // Withdrawal waiting period in seconds
}
```

### VaultParams

Parameters for vault initialization:

```typescript
interface VaultParams {
  config: VaultConfig;
  name: string;         // Max 32 bytes
  description: string;  // Max 64 bytes
}
```

### RequestWithdrawVaultArgs

Arguments for requesting a withdrawal:

```typescript
interface RequestWithdrawVaultArgs {
  amount: BN;           // Amount to withdraw
  isAmountInLp: boolean;  // true = amount is in LP tokens, false = in assets
  isWithdrawAll: boolean; // true = withdraw entire balance
}
```

### VaultConfigField

Enum for updating individual vault configuration fields:

```typescript
enum VaultConfigField {
  MaxCap = "maxCap",
  StartAtTs = "startAtTs",
  LockedProfitDegradationDuration = "lockedProfitDegradationDuration",
  WithdrawalWaitingPeriod = "withdrawalWaitingPeriod",
  ManagerPerformanceFee = "managerPerformanceFee",
  AdminPerformanceFee = "adminPerformanceFee",
  ManagerManagementFee = "managerManagementFee",
  AdminManagementFee = "adminManagementFee",
  RedemptionFee = "redemptionFee",
  IssuanceFee = "issuanceFee",
  Manager = "manager",
}
```

---

## PDA Finding Methods

### findVaultLpMint

Finds the vault's LP mint address.

```typescript
const vaultLpMint = client.findVaultLpMint(vault);
```

### findVaultAssetIdleAuth

Finds the vault's asset idle authority address.

```typescript
const vaultAssetIdleAuth = client.findVaultAssetIdleAuth(vault);
```

### findVaultAddresses

Finds all vault-related addresses.

```typescript
const { vaultLpMint, vaultAssetIdleAuth } = client.findVaultAddresses(vault);
```

### findVaultStrategyAuth

Finds the vault strategy authority address.

```typescript
const vaultStrategyAuth = client.findVaultStrategyAuth(vault, strategy);
```

### findStrategyInitReceipt

Finds the strategy init receipt address.

```typescript
const strategyInitReceipt = client.findStrategyInitReceipt(vault, strategy);
```

### findDirectWithdrawInitReceipt

Finds the direct withdraw init receipt address.

```typescript
const directWithdrawInitReceipt = client.findDirectWithdrawInitReceipt(vault, strategy);
```

### findVaultStrategyAddresses

Finds all vault-strategy related addresses.

```typescript
const {
  vaultStrategyAuth,
  strategyInitReceipt,
  directWithdrawInitReceipt,
} = client.findVaultStrategyAddresses(vault, strategy);
```

### findRequestWithdrawVaultReceipt

Finds the request withdraw vault receipt address for a user.

```typescript
const receipt = client.findRequestWithdrawVaultReceipt(vault, user);
```

### findLpMetadataAccount

Finds the LP metadata account address.

```typescript
const lpMetadataAccount = client.findLpMetadataAccount(vault);
```

---

## Vault Instructions

### createInitializeVaultIx

Creates an instruction to initialize a new vault.

```typescript
const ix = await client.createInitializeVaultIx(
  {
    config: {
      maxCap: new BN(0),
      startAtTs: new BN(0),
      lockedProfitDegradationDuration: new BN(86400),
      managerManagementFee: 50,
      managerPerformanceFee: 1000,
      adminManagementFee: 25,
      adminPerformanceFee: 500,
      redemptionFee: 10,
      issuanceFee: 10,
      withdrawalWaitingPeriod: new BN(0),
    },
    name: "My Vault",
    description: "Vault description"
  },
  {
    vault: vaultPubkey,
    vaultAssetMint: assetMintPubkey,
    admin: adminPubkey,
    manager: managerPubkey,
    payer: payerPubkey,
  }
);
```

### createUpdateVaultConfigIx

Creates an instruction to update a specific vault configuration field.

```typescript
import { VaultConfigField } from "@voltr/vault-sdk";

// Update max cap
const newMaxCap = new BN(1_000_000_000_000);
const data = newMaxCap.toArrayLike(Buffer, "le", 8);

const ix = await client.createUpdateVaultConfigIx(
  VaultConfigField.MaxCap,
  data,
  {
    vault: vaultPubkey,
    admin: adminPubkey,
  }
);

// Update management fee (requires LP mint)
const newFee = 100; // 1%
const feeData = Buffer.alloc(2);
feeData.writeUInt16LE(newFee, 0);

const ix = await client.createUpdateVaultConfigIx(
  VaultConfigField.ManagerManagementFee,
  feeData,
  {
    vault: vaultPubkey,
    admin: adminPubkey,
    vaultLpMint: client.findVaultLpMint(vaultPubkey),
  }
);

// Update manager
const newManager = new PublicKey("...");
const managerData = newManager.toBuffer();

const ix = await client.createUpdateVaultConfigIx(
  VaultConfigField.Manager,
  managerData,
  {
    vault: vaultPubkey,
    admin: adminPubkey,
  }
);
```

### createDepositVaultIx

Creates an instruction to deposit assets into a vault.

```typescript
const ix = await client.createDepositVaultIx(
  new BN(1_000_000_000), // amount
  {
    userTransferAuthority: userPubkey,
    vault: vaultPubkey,
    vaultAssetMint: assetMintPubkey,
    assetTokenProgram: TOKEN_PROGRAM_ID,
  }
);
```

### createRequestWithdrawVaultIx

Creates an instruction to request a withdrawal from a vault (Phase 1).

```typescript
const ix = await client.createRequestWithdrawVaultIx(
  {
    amount: new BN(1_000_000_000),
    isAmountInLp: false,
    isWithdrawAll: false,
  },
  {
    payer: payerPubkey,
    userTransferAuthority: userPubkey,
    vault: vaultPubkey,
  }
);
```

### createCancelRequestWithdrawVaultIx

Creates an instruction to cancel a pending withdrawal request.

```typescript
const ix = await client.createCancelRequestWithdrawVaultIx({
  userTransferAuthority: userPubkey,
  vault: vaultPubkey,
});
```

### createWithdrawVaultIx

Creates an instruction to complete a withdrawal (Phase 2, after waiting period).

```typescript
const ix = await client.createWithdrawVaultIx({
  userTransferAuthority: userPubkey,
  vault: vaultPubkey,
  vaultAssetMint: assetMintPubkey,
  assetTokenProgram: TOKEN_PROGRAM_ID,
});
```

---

## Strategy Instructions

### createAddAdaptorIx

Creates an instruction to add an adaptor to a vault.

```typescript
const ix = await client.createAddAdaptorIx({
  vault: vaultPubkey,
  payer: payerPubkey,
  admin: adminPubkey,
  adaptorProgram: LENDING_ADAPTOR_PROGRAM_ID, // optional, defaults to LENDING_ADAPTOR_PROGRAM_ID
});
```

### createRemoveAdaptorIx

Creates an instruction to remove an adaptor from a vault.

```typescript
const ix = await client.createRemoveAdaptorIx({
  vault: vaultPubkey,
  admin: adminPubkey,
  adaptorProgram: LENDING_ADAPTOR_PROGRAM_ID,
});
```

### createInitializeStrategyIx

Creates an instruction to initialize a strategy for a vault.

```typescript
const ix = await client.createInitializeStrategyIx(
  {
    instructionDiscriminator: null, // optional
    additionalArgs: null,           // optional
  },
  {
    payer: payerPubkey,
    vault: vaultPubkey,
    manager: managerPubkey,
    strategy: strategyPubkey,
    adaptorProgram: LENDING_ADAPTOR_PROGRAM_ID,
    remainingAccounts: [
      { pubkey: protocolProgram, isSigner: false, isWritable: false },
      // Additional protocol-specific accounts...
    ],
  }
);
```

### createDepositStrategyIx

Creates an instruction to deposit assets into a strategy.

```typescript
const ix = await client.createDepositStrategyIx(
  {
    depositAmount: new BN(1_000_000_000),
    instructionDiscriminator: null,
    additionalArgs: null,
  },
  {
    manager: managerPubkey,
    vault: vaultPubkey,
    vaultAssetMint: assetMintPubkey,
    strategy: strategyPubkey,
    assetTokenProgram: TOKEN_PROGRAM_ID,
    adaptorProgram: LENDING_ADAPTOR_PROGRAM_ID,
    remainingAccounts: [
      { pubkey: counterPartyTa, isSigner: false, isWritable: true },
      { pubkey: protocolProgram, isSigner: false, isWritable: false },
      // Additional protocol-specific accounts...
    ],
  }
);
```

### createWithdrawStrategyIx

Creates an instruction to withdraw assets from a strategy.

```typescript
const ix = await client.createWithdrawStrategyIx(
  {
    withdrawAmount: new BN(1_000_000_000),
    instructionDiscriminator: null,
    additionalArgs: null,
  },
  {
    manager: managerPubkey,
    vault: vaultPubkey,
    vaultAssetMint: assetMintPubkey,
    strategy: strategyPubkey,
    assetTokenProgram: TOKEN_PROGRAM_ID,
    adaptorProgram: LENDING_ADAPTOR_PROGRAM_ID,
    remainingAccounts: [
      { pubkey: counterPartyTaAuth, isSigner: false, isWritable: true },
      { pubkey: counterPartyTa, isSigner: false, isWritable: true },
      { pubkey: protocolProgram, isSigner: false, isWritable: false },
      // Additional protocol-specific accounts...
    ],
  }
);
```

### createInitializeDirectWithdrawStrategyIx

Creates an instruction to initialize direct withdrawal for a strategy.

```typescript
const ix = await client.createInitializeDirectWithdrawStrategyIx(
  {
    instructionDiscriminator: null,
    additionalArgs: null,
    allowUserArgs: false,
  },
  {
    payer: payerPubkey,
    admin: adminPubkey,
    vault: vaultPubkey,
    strategy: strategyPubkey,
    adaptorProgram: LENDING_ADAPTOR_PROGRAM_ID,
  }
);
```

### createDirectWithdrawStrategyIx

Creates an instruction for users to directly withdraw from a strategy.

```typescript
const ix = await client.createDirectWithdrawStrategyIx(
  { userArgs: null },
  {
    user: userPubkey,
    vault: vaultPubkey,
    strategy: strategyPubkey,
    vaultAssetMint: assetMintPubkey,
    assetTokenProgram: TOKEN_PROGRAM_ID,
    adaptorProgram: LENDING_ADAPTOR_PROGRAM_ID,
    remainingAccounts: [
      // Protocol-specific accounts...
    ],
  }
);
```

### createCloseStrategyIx

Creates an instruction to close a strategy.

```typescript
const ix = await client.createCloseStrategyIx({
  payer: payerPubkey,
  manager: managerPubkey,
  vault: vaultPubkey,
  strategy: strategyPubkey,
});
```

---

## Fee Management

### createHarvestFeeIx

Creates an instruction to harvest accumulated fees.

```typescript
const ix = await client.createHarvestFeeIx({
  harvester: harvesterPubkey,
  vaultManager: vaultManagerPubkey,
  vaultAdmin: vaultAdminPubkey,
  protocolAdmin: protocolAdminPubkey,
  vault: vaultPubkey,
});
```

### createCalibrateHighWaterMarkIx

Creates an instruction to calibrate the high water mark.

```typescript
const ix = await client.createCalibrateHighWaterMarkIx({
  vault: vaultPubkey,
  admin: adminPubkey,
});
```

### createCreateLpMetadataIx

Creates an instruction to create LP token metadata.

```typescript
const ix = await client.createCreateLpMetadataIx(
  {
    name: "My Vault LP",
    symbol: "MVLP",
    uri: "https://example.com/metadata.json",
  },
  {
    payer: payerPubkey,
    admin: adminPubkey,
    vault: vaultPubkey,
  }
);
```

---

## Query Methods

### Account Fetching

```typescript
// Fetch vault account data
const vaultAccount = await client.fetchVaultAccount(vaultPubkey);

// Fetch strategy init receipt
const receipt = await client.fetchStrategyInitReceiptAccount(receiptPubkey);

// Fetch adaptor add receipt
const adaptorReceipt = await client.fetchAdaptorAddReceiptAccount(receiptPubkey);

// Fetch request withdraw vault receipt
const withdrawReceipt = await client.fetchRequestWithdrawVaultReceiptAccount(receiptPubkey);
```

### Bulk Fetching

```typescript
// Fetch all strategy init receipts
const allReceipts = await client.fetchAllStrategyInitReceiptAccounts();

// Fetch all strategy init receipts for a vault
const vaultReceipts = await client.fetchAllStrategyInitReceiptAccountsOfVault(vaultPubkey);

// Fetch all adaptor add receipts for a vault
const adaptorReceipts = await client.fetchAllAdaptorAddReceiptAccountsOfVault(vaultPubkey);

// Fetch all pending withdrawal receipts for a vault
const withdrawalReceipts = await client.fetchAllRequestWithdrawVaultReceiptsOfVault(vaultPubkey);
```

### Helper Methods

```typescript
// Get position and total values for a vault
const { totalValue, strategies } = await client.getPositionAndTotalValuesForVault(vaultPubkey);

// Get accumulated admin fees
const adminFees = await client.getAccumulatedAdminFeesForVault(vaultPubkey);

// Get accumulated manager fees
const managerFees = await client.getAccumulatedManagerFeesForVault(vaultPubkey);

// Get current asset per LP ratio
const assetPerLp = await client.getCurrentAssetPerLpForVault(vaultPubkey);

// Get high water mark information
const hwm = await client.getHighWaterMarkForVault(vaultPubkey);
// Returns: { highestAssetPerLp: number, lastUpdatedTs: number }

// Get pending withdrawal for a user
const pending = await client.getPendingWithdrawalForUser(vaultPubkey, userPubkey);
// Returns: {
//   user: PublicKey,
//   amountAssetToWithdrawEffective: number,
//   amountAssetToWithdrawAtRequest: number,
//   amountAssetToWithdrawAtPresent: number,
//   amountLpEscrowed: number,
//   withdrawableFromTs: number,
// }

// Get all pending withdrawals for a vault
const allPending = await client.getAllPendingWithdrawalsForVault(vaultPubkey);

// Get LP supply breakdown
const breakdown = await client.getVaultLpSupplyBreakdown(vaultPubkey);
// Returns: {
//   circulating: BN,
//   unharvestedFees: BN,
//   unrealisedFees: BN,
//   total: BN,
// }
```

---

## Calculation Helpers

### calculateAssetsForWithdraw

Calculates the amount of assets that would be received for a given LP token amount.

```typescript
const assetsToReceive = await client.calculateAssetsForWithdraw(
  vaultPubkey,
  new BN(1_000_000_000) // LP amount
);
```

### calculateLpForWithdraw

Calculates the amount of LP tokens that would be burned for a given asset amount.

```typescript
const lpToBurn = await client.calculateLpForWithdraw(
  vaultPubkey,
  new BN(1_000_000_000) // asset amount
);
```

### calculateLpForDeposit

Calculates the amount of LP tokens that would be received for a given asset deposit.

```typescript
const lpToReceive = await client.calculateLpForDeposit(
  vaultPubkey,
  new BN(1_000_000_000) // asset amount
);
```

---

## Deprecated Methods

### createUpdateVaultIx (Deprecated)

Use `createUpdateVaultConfigIx` instead for more granular configuration updates.

```typescript
// DEPRECATED - avoid using
const ix = await client.createUpdateVaultIx(vaultConfig, { vault, admin });

// USE INSTEAD
const ix = await client.createUpdateVaultConfigIx(
  VaultConfigField.MaxCap,
  data,
  { vault, admin }
);
```
