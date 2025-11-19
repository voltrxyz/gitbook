# Frontend Integration

## Voltr Protocol - Frontend Integration Guide for Vault Deposits & Withdrawals

This guide explains how to integrate deposit and withdrawal functionality into your frontend application for the Voltr Protocol.

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
      userAuthority: wallet.publicKey,
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

#### 2. Handle Token Withdrawals

```typescript
const handleWithdraw = async (amount: string) => {
  const instructions: TransactionInstruction[] = [];
  const withdrawAmount = new BN(amount);
  
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
  
  // Create withdraw instruction
  const withdrawIx = await client.createWithdrawVaultIx(
    withdrawAmount,
    {
      vault,
      userAuthority: wallet.publicKey,
      vaultAssetMint,
      assetTokenProgram,
    }
  );
  
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

#### 3. Create Withdrawal UI

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
          const instructions = await handleWithdraw(amount);
          
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
      {loading ? 'Processing...' : 'Withdraw'}
    </button>
  </div>
);
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
