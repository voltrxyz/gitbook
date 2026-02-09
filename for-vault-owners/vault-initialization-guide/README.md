# Vault Initialization Guide

This guide helps you get from zero to a fully operational vault. There are two paths depending on your needs.

## Choose Your Path

|                             | UI Path                                                       | SDK Path        |
| --------------------------- | ------------------------------------------------------------- | --------------- |
| **Best for**                | MPC Wallets requiring browser experience (admin actions only) | Everything else |
| **Vault creation**          | Yes                                                           | Yes             |
| **Config updates**          | Yes                                                           | Yes             |
| **Metadata setup**          | Yes                                                           | Yes             |
| **Strategy initialization** | No — requires scripts                                         | Yes             |
| **Fund allocation**         | No — requires scripts                                         | Yes             |

{% hint style="info" %}
**Recommendation**: Use the **UI** for vault creation and config management, then use the **SDK + protocol-specific scripts** for strategy initialization and fund allocation. Most vault owners use both.
{% endhint %}

## Common Prerequisites

Regardless of which path you choose, you'll need:

1. **A Solana wallet** with sufficient SOL (\~0.15 SOL for vault creation + ongoing transaction fees)
2. **An RPC endpoint** — a reliable Solana RPC provider (e.g., Helius, Triton, QuickNode)
3. **Admin and manager keypairs** — two separate Solana keypairs for role separation
4. **A clear plan for your vault** — which asset, which strategies, target fees

## Next Steps

{% content-ref url="quick-start-ui.md" %}
[quick-start-ui.md](quick-start-ui.md)
{% endcontent-ref %}
