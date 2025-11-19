# Vault Creation

## Voltr Protocol - Vault Creation Guide

This guide walks through the process of creating and initializing a new vault in the Voltr Protocol.

### Setup

First, import the required dependencies:

```typescript
import { BN } from "@coral-xyz/anchor";
import { VaultConfig, VaultParams, VoltrClient } from "@voltr/vault-sdk";
import {
  Connection,
  Keypair,
  PublicKey,
  sendAndConfirmTransaction,
} from "@solana/web3.js";
import { BN } from "@coral-xyz/anchor";
```

### Step-by-Step Guide

#### 1. Prepare Vault Configuations

```typescript
const vaultConfig: VaultConfig = {
  maxCap: new BN(0),            // Maximum vault capacity
  startAtTs: new BN(0),         // Start timestamp
  managerPerformanceFee: 1000,  // 10% in basis points
  adminPerformanceFee: 500,     // 5% in basis points
  managerManagementFee: 50,     // 0.5% in basis points
  adminManagementFee: 25,       // 0.25% in basis points
};

const vaultParams: VaultParams = {
  config: vaultConfig,
  name: "My Voltr Vault",
  description: "Description of your vault strategy"
};
```

#### 2. Define Required Variables

```typescript
// File paths for keypairs
const adminFilePath = "/path/to/admin.json";
const managerFilePath = "/path/to/manager.json";

// Network and asset configuration
const assetMintAddress = "..."; // Your asset token mint
const solanaRpcUrl = "your-solana-rpc-url";

// load keypairs
const adminKp = Keypair.fromSecretKey(
  Uint8Array.from(JSON.parse(fs.readFileSync(adminFilePath, "utf-8")))
);
const managerKp = Keypair.fromSecretKey(
  Uint8Array.from(JSON.parse(fs.readFileSync(managerFilePath, "utf-8")))
);

// Generate vault keypair
const vaultKp = Keypair.generate();

// Initialize client
const connection = new Connection(solanaRpcUrl);
const client = new VoltrClient(connection);
```

#### 3. Create Vault Initialization Instruction

```typescript
// Create initialization instruction
const createVaultIx = await client.createInitializeVaultIx(
  vaultParams,
  {
    vault: vaultKp,
    vaultAssetMint: new PublicKey(assetMintAddress),
    admin: adminKp.publicKey,
    manager: managerKp.publicKey,
    payer: adminKp.publicKey,
  }
);
```

#### 4. Send and Confirm the Instruction

```typescript
// Send and confirm transaction
const txSig = await sendAndConfirmTransaction(
  [createVaultIx],
  connection,
  [adminKp, vaultKp]
);
```

### Account Structure

#### Vault Account

The vault account contains the following data:

```typescript
interface Vault {
  name: string;           // Max 32 bytes
  description: string;    // Max 64 bytes
  asset: {
    mint: PublicKey;      // Token mint address
    idleAuth: PublicKey;  // Idle token authority
    totalValue: BN;       // Total assets in vault
  };
  config: {
    maxCap: BN;                  // Maximum vault capacity
    startAtTs: BN;               // Start timestamp
    managerPerformanceFee: number; // In basis points
    adminPerformanceFee: number;   // In basis points
    managerManagementFee: number;  // In basis points
    adminManagementFee: number;    // In basis points
  };
}
```

### Important Considerations

#### Security Best Practices

1. **Key Management**:
   * Keep admin and manager keys separate
   * Use different keypairs for different environments
   * Never commit private keys to version control
2. **Fee Configuration**:
   * Management fees: typically 0.25% - 2% (25-200 basis points)
   * Performance fees: typically 5% - 20% (500-2000 basis points)
   * Consider the impact on user returns
3. **Asset Handling**:
   * Validate token decimals match between asset and LP tokens
   * Set appropriate maxCap to prevent overflow
   * Account for minimum deposit requirements

#### Error Handling

Common error scenarios and solutions:

1. **Initialization Failures**:
   * Verify account rent exemption
   * Check authority permissions
   * Ensure unique vault name
2. **Strategy Integration**:
   * Validate strategy program compatibility
   * Check account ownership
   * Verify remaining account requirements
3. **Transaction Failures**:
   * Monitor for insufficient funds
   * Handle RPC timeouts
   * Implement proper retry logic



For additional support or questions, refer to the [Voltr SDK documentation](https://voltrxyz.github.io/vault-sdk/) or [example scripts](https://github.com/voltrxyz/client-scripts).
