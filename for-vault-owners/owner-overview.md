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

## Role Based Access Control

### Admin vs. Manager Roles

Voltr enforces role separation for security:

| Role        | Capabilities                                                                                      |
| ----------- | ------------------------------------------------------------------------------------------------- |
| **Admin**   | Add/remove adaptors, initialize strategies, update vault configuration, calibrate high water mark |
| **Manager** | Allocate funds between strategies (deposit/withdraw to strategies)                                |

{% hint style="warning" %}
Keep admin and manager as **separate keypairs**. The admin controls vault structure; the manager controls fund movement. This separation limits damage if a key is compromised.
{% endhint %}

## Vault Lifecycle

A typical vault goes through these stages:

1. **Create the vault** — Initialize the on-chain vault account with your asset, fees, and configuration
2. **Set up metadata** — Create LP token metadata (name, symbol, image) so wallets display it correctly
3. **Add adaptors & initialize strategies** — Connect to DeFi protocols where funds will be deployed
4. **Allocate funds** — Deploy idle vault funds into initialized strategies
5. **Operate** — Monitor performance, rebalance, run automation scripts
6. **Go to market** — Get indexed on Ranger, verify your LP token on Jupiter

## Next Steps

{% content-ref url="/broken/pages/pTlOBXiJA8VhSq0068Ah" %}
[Broken link](/broken/pages/pTlOBXiJA8VhSq0068Ah)
{% endcontent-ref %}

{% content-ref url="current-integrations.md" %}
[current-integrations.md](current-integrations.md)
{% endcontent-ref %}

{% content-ref url="fees-and-accounting.md" %}
[fees-and-accounting.md](fees-and-accounting.md)
{% endcontent-ref %}
