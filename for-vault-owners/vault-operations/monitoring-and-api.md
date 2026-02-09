# Monitoring & API

Monitor your vault's performance and health using the Voltr API and SDK.

## Voltr API

The Voltr API provides read-only endpoints for querying vault data. This is the recommended approach for monitoring and user-facing applications.

**Base URL**: `https://api.voltr.xyz`

**Full documentation**: [api.voltr.xyz/docs](https://api.voltr.xyz/docs)

### Key Endpoints

| Endpoint | Description |
|----------|-----------|
| `GET /vaults` | List all vaults with summary data |
| `GET /vault/{address}` | Get detailed vault information |
| `GET /vault/{address}/apy` | Get vault APY data |
| `GET /vault/{address}/share-price` | Get current share price |

### Example: Query Vault Data

```bash
# Get vault details
curl https://api.voltr.xyz/vault/YOUR_VAULT_ADDRESS

# Get vault APY
curl https://api.voltr.xyz/vault/YOUR_VAULT_ADDRESS/apy
```

## SDK Query Methods

The SDK provides methods for querying on-chain vault state directly. Use these when you need real-time data or data not available through the API.

```typescript
import { VoltrClient } from "@voltr/vault-sdk";
import { Connection, PublicKey } from "@solana/web3.js";

const connection = new Connection("your-rpc-url");
const client = new VoltrClient(connection);
const vault = new PublicKey("your-vault-address");
```

### Vault State

```typescript
// Get full vault account data
const vaultData = await client.getVault(vault);
console.log("Total assets:", vaultData.asset.totalValue.toString());
console.log("Admin:", vaultData.admin.toBase58());
console.log("Manager:", vaultData.manager.toBase58());
```

### Fee Information

```typescript
// Get accumulated fees
const adminFees = await client.getAccumulatedAdminFeesForVault(vault);
const managerFees = await client.getAccumulatedManagerFeesForVault(vault);
console.log("Admin fees (LP):", adminFees.toString());
console.log("Manager fees (LP):", managerFees.toString());

// Get high water mark
const hwm = await client.getHighWaterMarkForVault(vault);
console.log("Highest asset per LP:", hwm.highestAssetPerLp);
console.log("Last updated:", new Date(hwm.lastUpdatedTs * 1000));
```

### Share Price & LP Supply

```typescript
// Get current asset per LP ratio (share price)
const assetPerLp = await client.getCurrentAssetPerLpForVault(vault);
console.log("Current asset per LP:", assetPerLp);

// Get LP supply breakdown
const lpBreakdown = await client.getVaultLpSupplyBreakdown(vault);
console.log("Circulating LP:", lpBreakdown.circulating.toString());
console.log("Unharvested fees:", lpBreakdown.unharvestedFees.toString());
console.log("Unrealised fees:", lpBreakdown.unrealisedFees.toString());
console.log("Total LP:", lpBreakdown.total.toString());
```

## Best Practices

{% hint style="info" %}
**API for reads, SDK for writes**: Use the Voltr API for monitoring, dashboards, and user-facing queries. Use the SDK for manager/admin operations that require signing transactions.
{% endhint %}

| Use Case | Recommended |
|----------|-------------|
| Dashboard / UI showing vault data | Voltr API |
| Monitoring vault APY over time | Voltr API |
| Checking fees before harvesting | SDK (real-time) |
| Automation scripts (rebalancing) | SDK |
| User deposit/withdraw UI | Voltr API + SDK |

## What to Monitor

### Key Metrics

* **Share price trend** — is the vault generating positive yield?
* **Total assets vs. idle assets** — what percentage is deployed vs. idle?
* **APY** — is performance meeting expectations?
* **Fee accumulation** — are fees ready to harvest?
* **Strategy health** — are all strategies performing as expected?

### Alert Triggers

Consider setting up alerts for:

* Share price decreasing (potential loss event)
* Idle balance dropping below threshold (withdrawal pressure)
* Strategy returning errors
* SOL balance running low on admin/manager wallets
