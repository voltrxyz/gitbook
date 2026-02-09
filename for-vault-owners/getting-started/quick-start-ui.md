# Quick Start (UI)

The Ranger vault creation UI provides the fastest way to create and configure a vault without writing any code.

## Step 1: Create Your Vault

Navigate to [vaults.ranger.finance/create](https://vaults.ranger.finance/create) and connect your wallet.

### Configuration Parameters

Fill in the following fields:

| Parameter | Description | Constraints |
|-----------|-----------|-------------|
| **Name** | Display name for your vault | Max 32 characters |
| **Description** | Brief description of the vault strategy | **Max 64 characters** |
| **Asset Mint** | The SPL token mint address the vault will accept | Must be a valid SPL token |
| **Admin Wallet** | Pubkey for the admin role | Can be the same as your connected wallet |
| **Manager Wallet** | Pubkey for the manager role | Recommended: separate from admin |
| **Max Cap** | Maximum total deposits allowed | See warning below |
| **Start Time** | When the vault becomes active | 0 = immediate |
| **Locked Profit Degradation** | How long profits stay locked (anti-sandwich) | Recommended: 86400 (24 hours) |
| **Withdrawal Waiting Period** | Delay before users can claim withdrawals | 0 = immediate claim |
| **Performance Fee (Manager)** | Manager's share of performance fees | In basis points (1000 = 10%) |
| **Performance Fee (Admin)** | Admin's share of performance fees | In basis points |
| **Management Fee (Manager)** | Manager's share of management fees | In basis points |
| **Management Fee (Admin)** | Admin's share of management fees | In basis points |
| **Issuance Fee** | Fee on deposits | In basis points |
| **Redemption Fee** | Fee on withdrawals | In basis points |

{% hint style="danger" %}
**maxCap warning**: Setting maxCap to `0` means **zero capacity** (no deposits allowed), NOT unlimited. For an uncapped vault, set maxCap to a very large number (e.g., `18446744073709551615` for u64 max).
{% endhint %}

{% hint style="warning" %}
**Description limit**: The description field is limited to **64 characters**. Exceeding this will cause the transaction to fail. Keep it concise.
{% endhint %}

### Submit the Transaction

Click "Create Vault" and approve the transaction in your wallet. The vault will be created on-chain and you'll receive the vault public key.

**Save your vault public key** — you'll need it for all subsequent operations.

## Step 2: Manage Your Vault

After creation, manage your vault at:

```
https://vaults.ranger.finance/manage/<VAULT_PUBKEY>
```

Replace `<VAULT_PUBKEY>` with your vault's public key.

From the manage page you can:

* View current vault state (TVL, share price, fees)
* Update vault configuration (as admin)
* View connected strategies

## Step 3: Next Steps (Requires SDK/Scripts)

The UI handles vault creation and configuration, but the following operations require the SDK or protocol-specific scripts:

1. **Set up LP token metadata** — so wallets display your vault token correctly
   → See [Vault Token Metadata](../vault-initialization-guide/vault-token-metadata.md)

2. **Initialize strategies** — connect your vault to DeFi protocols
   → See [Strategy Setup Guide](../strategy-setup-guide/README.md)

3. **Allocate funds** — deploy idle funds into strategies
   → See [Fund Allocation Guide](../fund-allocation-guide/README.md)

4. **Set up automation** — bots for rebalancing and rewards claiming
   → See [Running Bots & Scripts](../vault-operations/running-bots-and-scripts.md)

5. **Get listed on Ranger** — make your vault visible to users
   → See [Indexing & Listing on Ranger](../go-to-market/indexing-and-listing.md)

{% hint style="info" %}
Creating a vault via the UI is just the first step. Your vault won't generate yield until you initialize strategies and allocate funds to them using the SDK.
{% endhint %}
