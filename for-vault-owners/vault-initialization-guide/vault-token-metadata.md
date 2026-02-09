# Vault Token Metadata

When users deposit into your vault, they receive LP tokens. By default, these tokens have no metadata — wallets will display them as "Unknown Token." Setting up metadata ensures a proper user experience.

## Why Metadata Matters

* Wallets (Phantom, Solflare, etc.) display the token name, symbol, and image
* Jupiter and other aggregators use metadata for token identification
* Required for [token verification on Jupiter](../go-to-market/token-verification.md)

## Metadata JSON Format

Host a JSON file with the following structure at a publicly accessible URL:

```json
{
  "name": "My Vault LP",
  "symbol": "mvLP",
  "description": "LP token for My Vault on Voltr",
  "image": "https://your-domain.com/vault-logo.png"
}
```

### Hosting Options

| Option | Pros | Cons |
|--------|------|------|
| **GitHub repository** | Free, version controlled, reliable | Public repo required |
| **Arweave/IPFS** | Permanent, decentralized | Small cost, immutable (can't update) |
| **Your own domain** | Full control | Requires hosting |

{% hint style="info" %}
**Recommended**: Host your metadata JSON and logo image in a GitHub repository. See [github.com/ranger-finance/assets](https://github.com/ranger-finance/assets) for an example of how to structure your assets.
{% endhint %}

## Create Metadata via SDK

Use the `createCreateLpMetadataIx` instruction to attach metadata to your vault's LP token:

```typescript
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

// Create metadata instruction
const metadataIx = await client.createCreateLpMetadataIx(
  {
    name: "My Vault LP",
    symbol: "mvLP",
    uri: "https://your-domain.com/metadata.json",
  },
  {
    vault,
    admin: adminKp.publicKey,
    payer: adminKp.publicKey,
  }
);

// Send transaction
const txSig = await sendAndConfirmTransaction(
  [metadataIx],
  connection,
  [adminKp]
);

console.log("Metadata created:", txSig);
```

{% hint style="warning" %}
Only the **admin** can create or update LP token metadata.
{% endhint %}

## Update Existing Metadata

If you need to change the metadata after initial creation (e.g., update the logo or description), use `createUpdateLpMetadataIx`:

```typescript
const updateMetadataIx = await client.createUpdateLpMetadataIx(
  {
    name: "Updated Vault LP",
    symbol: "uvLP",
    uri: "https://your-domain.com/updated-metadata.json",
  },
  {
    vault,
    admin: adminKp.publicKey,
  }
);

const txSig = await sendAndConfirmTransaction(
  [updateMetadataIx],
  connection,
  [adminKp]
);
```

## Next Steps

After setting up metadata:

1. [Initialize strategies](../strategy-setup-guide/README.md) to connect your vault to DeFi protocols
2. Consider [verifying your token on Jupiter](../go-to-market/token-verification.md) to avoid wallet warnings
