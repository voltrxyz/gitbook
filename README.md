---
description: Modular infrastructure layer for structured yield strategies
---

# Introduction to Voltr

<figure><img src=".gitbook/assets/Twitter banner.png" alt=""><figcaption></figcaption></figure>

Voltr opens up access to sophisticated yield generation strategies on Solana to anyone. Through our Vaults, users can participate in automated yield optimization without requiring deep knowledge of underlying DeFi protocols or complex trading strategies.

Voltr is a permissionless framework enabling anyone – from experienced fund managers to AI agents – to create and manage yield-generating vaults. These vaults automatically identify and allocate capital to the most attractive opportunities, depending on vaults' allocation objectives (e.g. maximize returns), across Solana's DeFi ecosystem for all participants.

Whether you want to earn passive yields with just a few clicks, build and share your own DeFi strategies, or deploy AI-powered yield automation. We've made it simple for everyone to participate in sophisticated yield generation on Solana. Start earning, building, or automating with Voltr today.

## Allocation Strategies Examples

Below are examples of the different allocation strategies each yield-generating vaults can employ:

{% tabs %}
{% tab title="APY Maximizer" %}
* Monitor real-time APYs across all integrated lending protocols (Solend, Drift, Marginfi, Kamino)
* Calculate true APY including:
  * Base lending rates
  * Rewards token values
  * Gas costs for rebalancing
* Dynamically reallocate capital to highest yielding protocol
* Implement minimum time locks between reallocations to prevent excessive churn
{% endtab %}

{% tab title="Risk-Weighted" %}
* Assign risk scores to protocols based on:
  * TVL (Total Value Locked)
  * Code audit status
  * Historical exploit incidents
  * Protocol age/maturity
* Weight allocations inversely to risk scores
* Set maximum allocation caps per risk tier
* Maintain minimum diversification across 3+ protocols
{% endtab %}

{% tab title="Liquidation Protection" %}
* Monitor health factors across lending positions
* Keep buffer in low-risk protocols for quick deleverage
* Set dynamic allocation limits based on:
  * Market volatility
  * Protocol utilization rates
  * Collateral factors
* Automatically reduce positions approaching liquidation thresholds
{% endtab %}

{% tab title="Delta-Neutral" %}
* Maintain market-neutral exposure through:
  * Long spot positions as collateral in a perpetuals exchange like Drift
  * Short corresponding futures positions
* Dynamic hedge ratios based on:
  * Funding rate trends
  * Market skew
{% endtab %}
{% endtabs %}
