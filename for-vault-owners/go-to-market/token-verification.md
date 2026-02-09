# Token Verification

When users receive LP tokens from your vault, their wallets (Phantom, Solflare, etc.) may display warnings for unverified tokens. Verifying your LP token on Jupiter removes these warnings and builds trust with depositors.

## Why Verification Matters

* **Wallet warnings**: Unverified tokens show warning messages in Phantom and other wallets, which can discourage users from depositing
* **Jupiter listing**: Verified tokens appear on Jupiter's token list, making them recognizable across the Solana ecosystem
* **Trust signal**: Verification indicates the token is legitimate and associated with a known project

## Prerequisites

Before applying for verification:

1. **LP token metadata must be set up** — name, symbol, and image URI
   → See [Vault Token Metadata](../vault-initialization-guide/vault-token-metadata.md)
2. **Metadata must be hosted** at a publicly accessible URL
3. **Logo image** must meet Jupiter's requirements (typically PNG, reasonable resolution)

## Jupiter Verification Process

Jupiter maintains a community-driven token verification system.

### Step 1: Apply at Jupiter Verified

Submit your token for verification at [verified.jup.ag](https://verified.jup.ag).

### Step 2: Provide Required Information

You'll typically need:

* **Token mint address** — your vault's LP token mint
* **Token metadata** — name, symbol, logo URL
* **Project information** — website, social links, description
* **Contact information** — for the review team to reach you

### Step 3: Review

The Jupiter team reviews submissions. Approval timelines vary.

## After Verification

Once verified:

* Wallet warnings are removed for your LP token
* The token appears in Jupiter's verified token list
* Users see your token name, symbol, and logo in their wallets

{% hint style="info" %}
Verification is not strictly required for your vault to function, but it significantly improves the user experience. We strongly recommend completing this step before actively marketing your vault to users.
{% endhint %}

## Troubleshooting

| Issue | Solution |
|-------|---------|
| Metadata not showing in wallets | Ensure `createCreateLpMetadataIx` was executed successfully |
| Logo not displaying | Verify the image URL in your metadata JSON is publicly accessible |
| Verification rejected | Review Jupiter's requirements; ensure metadata is complete and accurate |
