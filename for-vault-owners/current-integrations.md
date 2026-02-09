# Supported Integrations

{% content-ref url="../security/deployed-programs.md" %}
[deployed-programs.md](../security/deployed-programs.md)
{% endcontent-ref %}

The Voltr core team has created the following adaptors for easy integration with any Voltr vaults. Adaptors are modular and permissionless — anyone can create their own adaptors to connect Voltr vaults with any protocol.

## Adaptor Overview

| Adaptor Program | Integrations | Description | Sample Scripts |
|----------------|-------------|-------------|----------------|
| **Generic Lending Adaptor** | Project0, Save | Lend assets to earn interest on lending protocols | [lend-scripts](https://github.com/voltrxyz/lend-scripts) |
| **Kamino Adaptor** | Kamino Vaults, Kamino Lending Market | Deposit into Kamino vault products or lending reserves | [kamino-scripts](https://github.com/voltrxyz/kamino-scripts) |
| **Drift Adaptor** | Drift Vaults, Drift Lend, Drift Perps | Trade perpetuals or lend via Drift | [drift-scripts](https://github.com/voltrxyz/drift-scripts) |
| **Raydium Adaptor** | All Raydium CLMM Pools | Provide concentrated liquidity on Raydium | [client-raydium-clmm-scripts](https://github.com/voltrxyz/client-raydium-clmm-scripts) |
| **Jupiter Adaptor** | SPL-Tokens (Jupiter Swap), Jupiter Lend | Swap tokens or lend via Jupiter | [spot-scripts](https://github.com/voltrxyz/spot-scripts) |
| **Trustful Adaptor** | Centralised Exchanges | Off-chain venue integration with trust-based reporting | [trustful-scripts](https://github.com/voltrxyz/trustful-scripts) |

***

## Generic Lending Adaptor

The generic lending adaptor supports depositing vault assets into lending protocols to earn interest.

### Project0 Lending Market

Deposit into Project0 lending pools for the vault's asset.

### Save (formerly Solend)

Deposit into Save lending pools. Supports all Save lending reserves for the vault's asset.

**Script**: See [voltrxyz/lend-scripts](https://github.com/voltrxyz/lend-scripts) for initialization scripts.

***

## Kamino Adaptor

Kamino has its own dedicated adaptor with two integration paths:

| Type | Description | Script Repo |
|------|-----------|-------------|
| **Kamino Vault** | Deposit into Kamino's own vault products | [voltrxyz/kamino-scripts](https://github.com/voltrxyz/kamino-scripts) |
| **Kamino Lending Market** | Deposit directly into Kamino lending reserves | [voltrxyz/kamino-scripts](https://github.com/voltrxyz/kamino-scripts) |

{% hint style="info" %}
Kamino vault and Kamino lending market use **different initialization scripts** even though they share the same adaptor program. Make sure you use the correct script for your target.
{% endhint %}

***

## Drift Adaptor

The Drift adaptor enables integration with Drift Protocol for vaults, lending, and perpetual trading.

### Drift Vaults

Deposit into existing Drift vault products.

### Drift Lend

Lend assets on Drift's lending market for interest income.

### Drift Perpetuals

Trade perpetual futures on Drift using vault funds. Key considerations:

* Requires setting up a **delegated authority** for trade-only permissions
* Uses Drift's sub-account system for position isolation
* Supports all Drift perpetual markets

{% hint style="warning" %}
**Drift perps carry significant risk.** Leveraged perpetual trading can result in losses exceeding the initial allocation. Ensure your vault's risk profile and marketing materials clearly communicate this to depositors.
{% endhint %}

| Script Repo | [voltrxyz/drift-scripts](https://github.com/voltrxyz/drift-scripts) |
|------------|---------------------------------------------------------------------|

***

## Raydium Adaptor

The Raydium adaptor supports concentrated liquidity market maker (CLMM) positions on Raydium.

* Supports **all Raydium CLMM pools**
* Vault managers set price ranges for concentrated liquidity positions
* Subject to impermanent loss (IL) — ensure your strategy accounts for this

{% hint style="warning" %}
Raydium CLMM positions carry **impermanent loss risk**. This is fundamentally different from lending strategies where IL is not a factor. Make sure depositors understand the risk profile.
{% endhint %}

| Script Repo | [voltrxyz/client-raydium-clmm-scripts](https://github.com/voltrxyz/client-raydium-clmm-scripts) |
|------------|---------------------------------------------------------------------|

***

## Jupiter Adaptor

The Jupiter adaptor provides two integration types:

### Jupiter Swap

Swap SPL tokens through Jupiter's aggregator. Primarily used as a utility within other strategies (e.g., swapping reward tokens back to the vault's base asset).

### Jupiter Lend

Deposit into Jupiter's lending product for interest income.

| Script Repo | [voltrxyz/spot-scripts](https://github.com/voltrxyz/spot-scripts) |
|------------|------------------------------------------------------------------|

***

## Trustful Adaptor

The trustful adaptor enables integration with **off-chain venues** that cannot be verified on-chain.

### How It Works

1. The manager moves funds from the vault to an external venue (CEX, MPC wallet, custodian)
2. The manager **manually reports** the position value back to the vault
3. The vault's total value reflects the reported amount
4. Users must **trust the manager** to report accurately

### Use Cases

* **CEX trading** — Deploy vault funds to centralized exchanges for market making or trading
* **OTC deals** — Execute large trades off-chain
* **MPC/custodian bridges** — Hold funds in institutional custody solutions

{% hint style="danger" %}
**Trust assumption**: The trustful adaptor relies entirely on the manager's honest reporting. There is no on-chain verification of the actual position value. This should be clearly communicated to depositors. Only use this adaptor when trust in the manager is established.
{% endhint %}

| Script Repo | [voltrxyz/trustful-scripts](https://github.com/voltrxyz/trustful-scripts) |
|------------|---------------------------------------------------------------------|

***

## Script Repositories

All protocol-specific initialization and management scripts are available in the following repositories:

| Repository | Protocols |
|-----------|----------|
| [voltrxyz/lend-scripts](https://github.com/voltrxyz/lend-scripts) | Project0, Save |
| [voltrxyz/kamino-scripts](https://github.com/voltrxyz/kamino-scripts) | Kamino Vault, Kamino Lending Market |
| [voltrxyz/drift-scripts](https://github.com/voltrxyz/drift-scripts) | Drift Vaults, Drift Lend, Drift Perps |
| [voltrxyz/client-raydium-clmm-scripts](https://github.com/voltrxyz/client-raydium-clmm-scripts) | Raydium CLMM Pools |
| [voltrxyz/spot-scripts](https://github.com/voltrxyz/spot-scripts) | Jupiter Swap, Jupiter Lend |
| [voltrxyz/trustful-scripts](https://github.com/voltrxyz/trustful-scripts) | Centralised Exchanges |

{% hint style="info" %}
For detailed strategy initialization instructions, see the [Strategy Setup Guide](strategy-setup-guide/README.md).
{% endhint %}
