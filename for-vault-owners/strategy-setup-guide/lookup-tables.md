# Lookup Tables (LUT)

Address Lookup Tables (LUTs) are a Solana feature that allows transactions to reference more accounts than would normally fit in a single transaction. Some Voltr strategy operations require LUTs due to the number of accounts involved.

## When Are LUTs Needed?

You may need LUTs when:

* **Strategy initialization** requires many protocol-specific accounts
* **Fund allocation** (deposit/withdraw) involves complex protocol interactions
* **Raydium CLMM** strategies (typically have many accounts per transaction)
* **Drift** strategies with multiple market accounts

{% hint style="info" %}
If you're getting transaction size errors (`Transaction too large`), you likely need to set up a LUT.
{% endhint %}

## How LUTs Work

Without a LUT, every account in a transaction requires 32 bytes for the full public key. A LUT stores account addresses in an on-chain table, and transactions can reference them by **1-byte index** instead of the full 32-byte address.

This allows transactions to include significantly more accounts within Solana's 1232-byte transaction limit.

## Creating a LUT

### Step 1: Create the Lookup Table

```typescript
import {
  Connection,
  Keypair,
  AddressLookupTableProgram,
  sendAndConfirmTransaction,
  Transaction,
} from "@solana/web3.js";

const connection = new Connection("your-rpc-url");
const payerKp = Keypair.fromSecretKey(/* ... */);

// Create the lookup table
const slot = await connection.getSlot();
const [createIx, lookupTableAddress] =
  AddressLookupTableProgram.createLookupTable({
    authority: payerKp.publicKey,
    payer: payerKp.publicKey,
    recentSlot: slot,
  });

const tx = new Transaction().add(createIx);
await sendAndConfirmTransaction(connection, tx, [payerKp]);

console.log("LUT created:", lookupTableAddress.toBase58());
```

### Step 2: Extend the LUT with Addresses

Add all accounts that will be used in your strategy transactions:

```typescript
import { AddressLookupTableProgram, PublicKey } from "@solana/web3.js";

// Collect all accounts from your strategy initialization and allocation transactions
const addresses = [
  new PublicKey("account-1"),
  new PublicKey("account-2"),
  // ... all accounts used in your strategy transactions
];

// Extend the lookup table (max 30 addresses per transaction)
for (let i = 0; i < addresses.length; i += 30) {
  const batch = addresses.slice(i, i + 30);

  const extendIx = AddressLookupTableProgram.extendLookupTable({
    lookupTable: lookupTableAddress,
    authority: payerKp.publicKey,
    payer: payerKp.publicKey,
    addresses: batch,
  });

  const tx = new Transaction().add(extendIx);
  await sendAndConfirmTransaction(connection, tx, [payerKp]);
}
```

### Step 3: Wait for Activation

{% hint style="warning" %}
Newly created or extended LUTs require **one epoch (~2 days)** to become active. Plan your setup timeline accordingly.
{% endhint %}

### Step 4: Use the LUT in Transactions

```typescript
import {
  TransactionMessage,
  VersionedTransaction,
  AddressLookupTableAccount,
} from "@solana/web3.js";

// Fetch the lookup table account
const lookupTableAccount = (
  await connection.getAddressLookupTable(lookupTableAddress)
).value;

// Create a versioned transaction with the LUT
const messageV0 = new TransactionMessage({
  payerKey: payerKp.publicKey,
  recentBlockhash: (await connection.getLatestBlockhash()).blockhash,
  instructions: [/* your strategy instructions */],
}).compileToV0Message([lookupTableAccount]);

const transaction = new VersionedTransaction(messageV0);
transaction.sign([payerKp]);

const txSig = await connection.sendTransaction(transaction);
```

## Which Accounts to Include

When building a LUT for a strategy, include all accounts from:

1. **Strategy initialization transaction** — all remaining accounts passed to `createInitializeStrategyIx`
2. **Deposit strategy transaction** — all remaining accounts passed to `createDepositStrategyIx`
3. **Withdraw strategy transaction** — all remaining accounts passed to `createWithdrawStrategyIx`
4. **Common accounts** — vault, strategy PDA, token accounts, program IDs

{% hint style="info" %}
**Tip**: Run your strategy initialization dry (simulate the transaction) to get the complete list of accounts needed, then add all of them to your LUT.
{% endhint %}

## Best Practices

* **One LUT per vault** is usually sufficient — include accounts for all strategies
* **Reuse LUTs** across multiple transactions for the same vault
* **Plan ahead** — create and extend LUTs before you need them (remember the activation delay)
* **Keep the LUT authority** — you may need to extend it later if you add new strategies
