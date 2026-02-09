# Frontend Integration

## Voltr Protocol - Frontend Integration Guide for Vault Deposits & Withdrawals

This guide explains how to integrate deposit and withdrawal functionality into your frontend application for the Voltr Protocol.

{% hint style="info" %}
**API vs SDK**: Use the [Voltr API](https://api.voltr.xyz/docs) for reading vault data (APY, TVL, share price) — it's simpler and doesn't require the SDK. Use the SDK (shown below) for building deposit and withdrawal transactions that users sign with their wallets.
{% endhint %}

### Setup

Import the required dependencies:

```typescript
import { Connection, PublicKey, TransactionInstruction } from "@solana/web3.js";
import { VoltrClient } from "@voltr/vault-sdk";
import { BN } from "@coral-xyz/anchor";
import {
  createAssociatedTokenAccountIdempotentInstruction,
  createSyncNativeInstruction,
  createCloseAccountInstruction,
  getAssociatedTokenAddressSync,
  NATIVE_MINT,
} from "@solana/spl-token";
```

Initialize the client:

```typescript
const connection = new Connection(rpcEndpoint);
const client = new VoltrClient(connection);

// Initialize vault constants
const VAULT_ADDRESS = new PublicKey("your-vault-address");
const VAULT_ASSET_MINT = new PublicKey("your-asset-mint");
```

### Deposit Implementation

#### 1. Create Deposit Component

```typescript
import React, { useState } from 'react';

const VaultDeposit = ({ 
  wallet, 
  vault, 
  vaultAssetMint, 
  assetTokenProgram 
}) => {
  const [amount, setAmount] = useState('');
  const [loading, setLoading] = useState(false);
  
  // Implementation below
};
```

#### 2. Handle SOL/SPL Token Deposits

```typescript
const handleDeposit = async (amount: string) => {
  const instructions: TransactionInstruction[] = [];
  const depositAmount = new BN(amount);
  
  // For SOL deposits, handle wrapping
  if (vaultAssetMint.equals(NATIVE_MINT)) {
    const userWsolAta = getAssociatedTokenAddressSync(
      NATIVE_MINT, 
      wallet.publicKey
    );
    
    instructions.push(
      // Create WSOL account
      createAssociatedTokenAccountIdempotentInstruction(
        wallet.publicKey,
        userWsolAta,
        wallet.publicKey,
        NATIVE_MINT
      ),
      // Transfer SOL to WSOL account
      SystemProgram.transfer({
        fromPubkey: wallet.publicKey,
        toPubkey: userWsolAta,
        lamports: depositAmount.toNumber(),
      }),
      // Sync native instruction
      createSyncNativeInstruction(userWsolAta)
    );
  }
  
  // Create LP token account
  const { vaultLpMint } = client.findVaultAddresses(vault);
  const userLpAta = getAssociatedTokenAddressSync(
    vaultLpMint,
    wallet.publicKey
  );
  
  instructions.push(
    createAssociatedTokenAccountIdempotentInstruction(
      wallet.publicKey,
      userLpAta,
      wallet.publicKey,
      vaultLpMint
    )
  );
  
  // Create deposit instruction
  const depositIx = await client.createDepositVaultIx(
    depositAmount,
    {
      vault,
      userTransferAuthority: wallet.publicKey,
      vaultAssetMint,
      assetTokenProgram,
    }
  );

  instructions.push(depositIx);

  return instructions;
};
```

#### 3. Create Deposit UI

```tsx
return (
  <div>
    <input
      type="number"
      value={amount}
      onChange={(e) => setAmount(e.target.value)}
      placeholder="Enter amount to deposit"
    />
    <button
      onClick={async () => {
        try {
          setLoading(true);
          const instructions = await handleDeposit(amount);
          
          // Send transaction
          const { blockhash } = await connection.getLatestBlockhash();
          const transaction = new Transaction({
            feePayer: wallet.publicKey,
            blockhash,
            lastValidBlockHeight: lastValidBlockHeight,
          }).add(...instructions);
          
          const signed = await wallet.signTransaction(transaction);
          const txId = await connection.sendRawTransaction(
            signed.serialize()
          );
          
          await connection.confirmTransaction(txId);
          
          // Handle success
        } catch (error) {
          // Handle error
        } finally {
          setLoading(false);
        }
      }}
      disabled={loading}
    >
      {loading ? 'Processing...' : 'Deposit'}
    </button>
  </div>
);
```

### Withdrawal Implementation

Voltr uses a **two-phase withdrawal system** for vault withdrawals:

1. **Request Withdrawal** - User requests to withdraw, LP tokens are escrowed
2. **Claim Withdrawal** - After the waiting period, user claims their assets

This design prevents sandwich attacks and ensures fair withdrawal pricing.

#### 1. Create Withdrawal Component

```typescript
import React, { useState } from 'react';

const VaultWithdraw = ({
  wallet,
  vault,
  vaultAssetMint,
  assetTokenProgram
}) => {
  const [amount, setAmount] = useState('');
  const [loading, setLoading] = useState(false);

  // Implementation below
};
```

#### 2. Phase 1: Request Withdrawal

```typescript
const handleRequestWithdraw = async (amount: string) => {
  const instructions: TransactionInstruction[] = [];
  const withdrawAmount = new BN(amount);

  // Create request withdraw instruction
  const requestWithdrawIx = await client.createRequestWithdrawVaultIx(
    {
      amount: withdrawAmount,
      isAmountInLp: false,      // true if amount is in LP tokens
      isWithdrawAll: false,     // true to withdraw entire balance
    },
    {
      payer: wallet.publicKey,
      userTransferAuthority: wallet.publicKey,
      vault,
    }
  );

  instructions.push(requestWithdrawIx);

  return instructions;
};
```

#### 3. Phase 2: Claim Withdrawal (After Waiting Period)

```typescript
const handleClaimWithdraw = async () => {
  const instructions: TransactionInstruction[] = [];

  // Create user's asset token account
  const userAssetAta = getAssociatedTokenAddressSync(
    vaultAssetMint,
    wallet.publicKey
  );

  instructions.push(
    createAssociatedTokenAccountIdempotentInstruction(
      wallet.publicKey,
      userAssetAta,
      wallet.publicKey,
      vaultAssetMint
    )
  );

  // Create withdraw instruction (claim phase)
  const withdrawIx = await client.createWithdrawVaultIx({
    userTransferAuthority: wallet.publicKey,
    vault,
    vaultAssetMint,
    assetTokenProgram,
  });

  instructions.push(withdrawIx);

  // For SOL withdrawals, handle unwrapping
  if (vaultAssetMint.equals(NATIVE_MINT)) {
    instructions.push(
      createCloseAccountInstruction(
        userAssetAta,
        wallet.publicKey,
        wallet.publicKey,
        []
      )
    );
  }

  return instructions;
};
```

#### 4. Cancel Pending Withdrawal (Optional)

```typescript
const handleCancelWithdraw = async () => {
  const cancelIx = await client.createCancelRequestWithdrawVaultIx({
    userTransferAuthority: wallet.publicKey,
    vault,
  });

  return [cancelIx];
};
```

#### 5. Check Pending Withdrawal Status

```typescript
const checkPendingWithdrawal = async () => {
  try {
    const pendingWithdrawal = await client.getPendingWithdrawalForUser(
      vault,
      wallet.publicKey
    );

    return {
      amountToWithdraw: pendingWithdrawal.amountAssetToWithdrawEffective,
      withdrawableFromTs: pendingWithdrawal.withdrawableFromTs,
      isReady: Date.now() / 1000 >= pendingWithdrawal.withdrawableFromTs,
    };
  } catch (error) {
    // No pending withdrawal
    return null;
  }
};
```

#### 6. Create Withdrawal UI

```tsx
return (
  <div>
    <input
      type="number"
      value={amount}
      onChange={(e) => setAmount(e.target.value)}
      placeholder="Enter amount to withdraw"
    />
    <button
      onClick={async () => {
        try {
          setLoading(true);
          const instructions = await handleRequestWithdraw(amount);

          // Send transaction
          const { blockhash, lastValidBlockHeight } =
            await connection.getLatestBlockhash();
          const transaction = new Transaction({
            feePayer: wallet.publicKey,
            blockhash,
            lastValidBlockHeight,
          }).add(...instructions);

          const signed = await wallet.signTransaction(transaction);
          const txId = await connection.sendRawTransaction(
            signed.serialize()
          );

          await connection.confirmTransaction(txId);

          // Handle success - inform user about waiting period
        } catch (error) {
          // Handle error
        } finally {
          setLoading(false);
        }
      }}
      disabled={loading}
    >
      {loading ? 'Processing...' : 'Request Withdraw'}
    </button>

    {/* Claim button for when waiting period is over */}
    <button
      onClick={async () => {
        try {
          setLoading(true);
          const instructions = await handleClaimWithdraw();
          // ... send transaction
        } catch (error) {
          // Handle error
        } finally {
          setLoading(false);
        }
      }}
      disabled={loading}
    >
      {loading ? 'Processing...' : 'Claim Withdrawal'}
    </button>
  </div>
);
```

### Direct Withdrawal (For Enabled Strategies)

Some vaults may enable direct withdrawal from strategies, allowing users to bypass the two-phase withdrawal for specific strategies:

```typescript
const handleDirectWithdraw = async () => {
  const instructions: TransactionInstruction[] = [];

  // Create user's asset token account
  const userAssetAta = getAssociatedTokenAddressSync(
    vaultAssetMint,
    wallet.publicKey
  );

  instructions.push(
    createAssociatedTokenAccountIdempotentInstruction(
      wallet.publicKey,
      userAssetAta,
      wallet.publicKey,
      vaultAssetMint
    )
  );

  const directWithdrawIx = await client.createDirectWithdrawStrategyIx(
    { userArgs: null },
    {
      user: wallet.publicKey,
      vault,
      strategy,
      vaultAssetMint,
      assetTokenProgram,
      remainingAccounts: [
        // Protocol-specific accounts required for direct withdrawal
      ],
    }
  );

  instructions.push(directWithdrawIx);

  return instructions;
};
```

### Important Considerations

#### 1. Token Account Management

* Always check if token accounts exist before transactions
* Create ATAs idempotently to prevent errors
* Handle SOL wrapping/unwrapping correctly
* Clean up temporary token accounts

#### 2. Transaction Handling

* Implement proper error handling
* Show loading states during transactions
* Provide clear feedback to users
* Handle transaction confirmation properly

#### 3. User Experience

* Display proper decimal places for amounts
* Show available balance
* Implement input validation
* Display transaction fees
* Show transaction status updates

#### 4. Error Cases

Handle common error scenarios:

```typescript
function handleTransactionError(error: any) {
  if (error.message.includes("insufficient funds")) {
    return "Insufficient funds for transaction";
  }
  if (error.message.includes("invalid amount")) {
    return "Invalid amount specified";
  }
  // Add other specific error cases
  return "Transaction failed. Please try again.";
}
```

For additional support or questions, refer to the [Voltr SDK documentation](https://voltrxyz.github.io/vault-sdk/) or [example scripts](https://github.com/voltrxyz/client-scripts).
