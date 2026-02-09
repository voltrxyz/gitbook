# Via User Interface

The Ranger vault creation UI provides the fastest way to create and configure a vault without writing any code.

## Step 1: Create Your Vault

Navigate to [vaults.ranger.finance/create](https://vaults.ranger.finance/create) and connect your wallet.

### Submit the Transaction

Click "Initialize Vault" and approve the transaction in your wallet. The vault will be created on-chain and you'll receive the vault public key.

**Save your vault public key** — you'll need it for all subsequent operations.

## Step 2: Add Adapators and Manage Your Vault

After creation, manage your vault at:

```
https://vaults.ranger.finance/manage/<VAULT_PUBKEY>
```

Replace `<VAULT_PUBKEY>` with your vault's public key.

From the manage page you can:

* View, add, and remove Adaptors
* Update vault configuration (as admin)

## Step 3: Next Steps (Requires SDK/Scripts)

The UI handles vault creation and configuration, but the following operations require the SDK or protocol-specific scripts:

1. **Initialize strategies** — connect your vault to DeFi protocols → See [Strategy Setup Guide](../strategy-setup-guide.md)
2. **Allocate funds** — deploy idle funds into strategies → See [Fund Allocation Guide](../fund-allocation-guide/)
3. **Set up automation** — bots for rebalancing and rewards claiming → See [Running Bots & Scripts](../vault-operations/running-bots-and-scripts.md)
4. **Get listed on Ranger** — make your vault visible to users → See [Indexing & Listing on Ranger](../go-to-market/indexing-and-listing.md)

{% hint style="info" %}
Creating a vault via the UI is just the first step. Your vault won't generate yield until you initialize strategies and allocate funds to them using the SDK.
{% endhint %}
