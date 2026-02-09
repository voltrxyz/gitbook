# Getting Started

This guide helps you get from zero to a fully operational vault. There are two paths depending on your needs.

## Choose Your Path

| | UI Path | SDK Path |
|---|---------|---------|
| **Best for** | Quick setup, non-technical users | Full automation, advanced strategies |
| **Vault creation** | Yes | Yes |
| **Config updates** | Yes | Yes |
| **Strategy initialization** | No — requires scripts | Yes |
| **Fund allocation** | No — requires scripts | Yes |
| **Metadata setup** | No — requires SDK | Yes |
| **Rewards claiming** | No — requires scripts | Yes |

{% hint style="info" %}
**Recommendation**: Use the **UI** for vault creation and config management, then use the **SDK + protocol-specific scripts** for strategy initialization and fund allocation. Most vault owners use both.
{% endhint %}

## Common Prerequisites

Regardless of which path you choose, you'll need:

1. **A Solana wallet** with sufficient SOL (~0.15 SOL for vault creation + ongoing transaction fees)
2. **An RPC endpoint** — a reliable Solana RPC provider (e.g., Helius, Triton, QuickNode)
3. **Admin and manager keypairs** — two separate Solana keypairs for role separation
4. **A clear plan for your vault** — which asset, which strategies, target fees

## Decision Checklist

Before you start, decide on:

- [ ] **Asset**: Which SPL token will this vault accept? (e.g., USDC, SOL, JitoSOL)
- [ ] **Strategies**: Which DeFi protocols will you deploy to? (see [Supported Integrations](../current-integrations.md))
- [ ] **Fees**: What performance, management, issuance, and redemption fees? (see [Fees & Accounting](../fees-and-accounting.md))
- [ ] **Vault name**: Max 32 characters
- [ ] **Vault description**: Max 64 characters
- [ ] **Max cap**: Maximum total deposits (or uncapped)
- [ ] **Withdrawal waiting period**: How long users wait to claim withdrawals (0 = immediate)

## Next Steps

{% content-ref url="quick-start-ui.md" %}
[Quick Start (UI)](quick-start-ui.md)
{% endcontent-ref %}

{% content-ref url="quick-start-sdk.md" %}
[Quick Start (SDK)](quick-start-sdk.md)
{% endcontent-ref %}
