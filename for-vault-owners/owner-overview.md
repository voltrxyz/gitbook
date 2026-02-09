---
description: >-
  Everything you need to know about creating, managing, and operating vaults on
  the Voltr protocol.
---

# Owner Overview

<figure><img src="../.gitbook/assets/manager_overview.png" alt=""><figcaption></figcaption></figure>

## What Is a Vault?

A Voltr vault is an on-chain smart contract on Solana that accepts deposits from users in a **single asset** (e.g., USDC, SOL) and deploys those funds into one or more DeFi strategies to generate yield. Users receive LP tokens representing their proportional share of the vault.

{% hint style="info" %}
Each vault supports **one asset only**. If you want to manage multiple assets, you need to create separate vaults for each.
{% endhint %}

## Key Concepts

### Vault vs. Strategy vs. Adaptor

| Concept | What It Is |
|---------|-----------|
| **Vault** | The container that holds user deposits and manages LP token accounting. Created via the `voltr-vault` program. |
| **Strategy** | A specific deployment of funds to a DeFi protocol (e.g., lending USDC on Kamino). A vault can have multiple strategies. |
| **Adaptor** | The on-chain program that connects vaults to DeFi protocols. Each adaptor supports a category of protocols (lending, Drift, Raydium, etc.). |

### Admin vs. Manager Roles

Voltr enforces role separation for security:

| Role | Capabilities |
|------|-------------|
| **Admin** | Add/remove adaptors, initialize strategies, update vault configuration, calibrate high water mark |
| **Manager** | Allocate funds between strategies (deposit/withdraw to strategies), claim protocol rewards |

{% hint style="warning" %}
Keep admin and manager as **separate keypairs**. The admin controls vault structure; the manager controls fund movement. This separation limits damage if a key is compromised.
{% endhint %}

## Vault Lifecycle

A typical vault goes through these stages:

1. **Create the vault** — Initialize the on-chain vault account with your asset, fees, and configuration
2. **Set up metadata** — Create LP token metadata (name, symbol, image) so wallets display it correctly
3. **Add adaptors & initialize strategies** — Connect to DeFi protocols where funds will be deployed
4. **Allocate funds** — Deploy idle vault funds into initialized strategies
5. **Operate** — Monitor performance, rebalance, claim rewards, run automation scripts
6. **Go to market** — Get indexed on Ranger, verify your LP token on Jupiter

{% hint style="danger" %}
Creating a vault alone does **not** generate yield. Funds sit idle until you add adaptors, initialize strategies, and allocate funds to them. This is the most common mistake new vault owners make.
{% endhint %}

## Two Paths: UI vs. SDK

| Path | Best For | What You Can Do |
|------|---------|----------------|
| **UI** ([vaults.ranger.finance](https://vaults.ranger.finance)) | Quick vault creation, config updates, basic management | Create vault, update config, view vault state |
| **SDK** (`@voltr/vault-sdk`) | Full control, automation, strategy initialization, fund allocation | Everything — vault creation, strategy init, fund allocation, rewards claiming, metadata setup |

Most vault owners use **both**: the UI for initial creation and config changes, and the SDK (with scripts) for strategy management and fund allocation.

## Important Notes

{% hint style="warning" %}
**Description field limit**: Vault descriptions are limited to **64 characters**. Plan your description accordingly — this cannot be easily changed later.
{% endhint %}

* **No hosting services**: Voltr/Ranger does not host bots or scripts for you. You are responsible for running your own automation infrastructure (rebalancing, rewards claiming, etc.).
* **Manual indexing**: Vaults are not automatically listed on the Ranger UI. You must contact the Ranger team to get your vault indexed and visible to users.
* **One asset per vault**: Multi-asset vaults are not supported. Create separate vaults for each token you want to manage.

## Next Steps

{% content-ref url="getting-started/README.md" %}
[Getting Started](getting-started/README.md)
{% endcontent-ref %}

{% content-ref url="current-integrations.md" %}
[Supported Integrations](current-integrations.md)
{% endcontent-ref %}

{% content-ref url="fees-and-accounting.md" %}
[Fees & Accounting](fees-and-accounting.md)
{% endcontent-ref %}
