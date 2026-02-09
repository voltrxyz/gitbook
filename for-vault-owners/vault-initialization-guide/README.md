# Vault Initialization Guide

This guide covers the complete process of initializing a vault on the Voltr protocol — from prerequisites through creation, metadata setup, and configuration.

{% hint style="info" %}
Each vault supports **one asset only** (e.g., USDC, SOL). If you need to manage multiple assets, create separate vaults.
{% endhint %}

## What Happens During Initialization

When you create a vault, the protocol:

1. Creates the vault account on-chain with your specified configuration
2. Sets up the idle token account (where undeployed funds sit)
3. Creates the LP token mint (users receive LP tokens when they deposit)
4. Assigns admin and manager roles to the keypairs you specify

After initialization, your vault exists but has **no strategies** — funds deposited will sit idle until you complete the [Strategy Setup Guide](../strategy-setup-guide/README.md).

## Initialization Steps

| Step | Page | Required |
|------|------|----------|
| 1. Review requirements | [Prerequisites](prerequisites.md) | Yes |
| 2. Create the vault | [Vault Creation](vault-creation.md) | Yes |
| 3. Set up LP token metadata | [Vault Token Metadata](vault-token-metadata.md) | Recommended |
| 4. Update configuration (if needed) | [Vault Configuration Updates](vault-configuration-updates.md) | Optional |

## Core Features

### Asset Management

* **Total Asset Tracking**: Real-time accounting of all assets under management
* **Idle Asset Management**: Tracks assets not currently deployed to strategies
* **Maximum Capacity**: Configurable deposit caps to manage risk

### LP Token Mechanics

* **Token Issuance**: Mints LP tokens representing proportional vault shares
* **Share Price Calculation**: LP token value based on total assets
* **Share-Based Accounting**: User ownership tracked through LP token balances

### Fee Configuration

Fees are set at vault creation and can be updated by the admin afterward. For detailed fee mechanics, see [Fees & Accounting](../fees-and-accounting.md).
