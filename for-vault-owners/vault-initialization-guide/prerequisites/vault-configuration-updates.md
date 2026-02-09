# Vault Configuration Updates

After creating a vault, the **admin** can update various configuration parameters. This is useful for adjusting fees, changing deposit caps, or modifying the withdrawal waiting period.

## What Can Be Updated

| Parameter | Updatable | Updated By |
|-----------|-----------|-----------|
| Max cap | Yes | Admin |
| Locked profit degradation duration | Yes | Admin |
| Withdrawal waiting period | Yes | Admin |
| Performance fees (admin/manager) | Yes | Admin |
| Management fees (admin/manager) | Yes | Admin |
| Issuance fee | Yes | Admin |
| Redemption fee | Yes | Admin |
| Vault name | No | — |
| Vault description | No | — |
| Asset mint | No | — |

{% hint style="info" %}
Vault name, description, and asset mint are set at creation and **cannot be changed** afterward. Choose carefully during vault creation.
{% endhint %}

## Update via UI

The simplest way to update vault configuration is through the Ranger manage page:

```
https://vaults.ranger.finance/manage/<VAULT_PUBKEY>
```

Connect with the **admin wallet** and use the configuration update form.

## Update via SDK

Use `createUpdateVaultConfigIx` to update vault configuration programmatically:

```typescript
import { BN } from "@coral-xyz/anchor";
import { VoltrClient } from "@voltr/vault-sdk";
import {
  Connection,
  Keypair,
  PublicKey,
  sendAndConfirmTransaction,
} from "@solana/web3.js";
import fs from "fs";

// Setup
const connection = new Connection("your-rpc-url");
const client = new VoltrClient(connection);

const adminKp = Keypair.fromSecretKey(
  Uint8Array.from(JSON.parse(fs.readFileSync("/path/to/admin.json", "utf-8")))
);

const vault = new PublicKey("your-vault-pubkey");

// Update configuration
const updateConfigIx = await client.createUpdateVaultConfigIx(
  {
    newConfig: {
      maxCap: new BN("18446744073709551615"), // Update max cap to uncapped
      startAtTs: null,                         // null = don't change
      lockedProfitDegradationDuration: null,   // null = don't change
      managerPerformanceFee: 1500,             // Update to 15%
      adminPerformanceFee: null,               // null = don't change
      managerManagementFee: null,
      adminManagementFee: null,
      redemptionFee: null,
      issuanceFee: null,
      withdrawalWaitingPeriod: null,
    },
  },
  {
    vault,
    admin: adminKp.publicKey,
  }
);

const txSig = await sendAndConfirmTransaction(
  [updateConfigIx],
  connection,
  [adminKp]
);

console.log("Config updated:", txSig);
```

{% hint style="warning" %}
Pass `null` for any field you don't want to change. Only non-null fields will be updated.
{% endhint %}

## Changing Admin or Manager

To transfer the admin or manager role to a different keypair:

```typescript
// Transfer admin role
const updateAdminIx = await client.createUpdateVaultAdminIx({
  vault,
  admin: currentAdminKp.publicKey,
  newAdmin: newAdminPubkey,
});

// Transfer manager role
const updateManagerIx = await client.createUpdateVaultManagerIx({
  vault,
  admin: adminKp.publicKey,
  newManager: newManagerPubkey,
});
```

{% hint style="danger" %}
**Be extremely careful** when transferring admin or manager roles. Once transferred, the old keypair loses all authority. There is no way to reverse this without the new keypair holder's cooperation.
{% endhint %}
