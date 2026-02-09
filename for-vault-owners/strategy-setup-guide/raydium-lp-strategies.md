# Raydium LP Strategies

The Raydium adaptor enables concentrated liquidity market making (CLMM) on Raydium. Vault funds are deployed as liquidity positions within specified price ranges.

## Overview

* Supports **all Raydium CLMM pools**
* Vault managers set price ranges for concentrated liquidity positions
* Earns trading fees from swaps that pass through the position's price range
* Subject to impermanent loss (IL)

## Key Concepts

### Concentrated Liquidity

Unlike traditional AMMs that spread liquidity across all prices, CLMM positions concentrate liquidity within a specified price range. This means:

* **Higher capital efficiency** — more fees earned per dollar deployed (when price is in range)
* **Active management required** — positions must be rebalanced when price moves out of range
* **Higher IL risk** — concentrated positions amplify impermanent loss

### Position Management

The vault manager is responsible for:

1. Choosing the pool (token pair)
2. Setting the price range (lower tick, upper tick)
3. Monitoring whether the position is in range
4. Rebalancing when price moves out of range

## Initialization

### Prerequisites

1. Your vault must already be created
2. The **Raydium adaptor** must be added to your vault
3. Admin keypair for strategy initialization

### Adding the Raydium Adaptor

```typescript
import { VoltrClient } from "@voltr/vault-sdk";
import { Connection, Keypair, PublicKey, sendAndConfirmTransaction } from "@solana/web3.js";

const connection = new Connection("your-rpc-url");
const client = new VoltrClient(connection);

const adminKp = Keypair.fromSecretKey(/* ... */);
const vault = new PublicKey("your-vault-pubkey");

const addAdaptorIx = await client.createAddAdaptorIx({
  vault,
  admin: adminKp.publicKey,
  payer: adminKp.publicKey,
});

await sendAndConfirmTransaction([addAdaptorIx], connection, [adminKp]);
```

### Strategy Initialization

Raydium CLMM strategy initialization requires pool-specific accounts. The initialization sets up the position account that the vault will use.

{% hint style="info" %}
Raydium CLMM strategies involve more accounts than lending strategies. You may need to use [Lookup Tables (LUTs)](lookup-tables.md) to fit the transaction within Solana's size limits.
{% endhint %}

## Risk Considerations

{% hint style="warning" %}
**Impermanent loss (IL)** is the primary risk for Raydium CLMM strategies. IL increases as the price moves further from the position's initial entry point. This is fundamentally different from lending strategies.
{% endhint %}

| Risk | Description | Mitigation |
|------|-----------|-----------|
| **Impermanent loss** | Price movement causes IL | Narrow ranges increase IL risk; wider ranges reduce it but earn fewer fees |
| **Out-of-range** | Position earns nothing when price is outside range | Active monitoring and rebalancing |
| **Protocol risk** | Smart contract risk from Raydium | Audit the Raydium program, limit allocation |

## Automation

Raydium CLMM strategies typically require active automation for:

* **Price monitoring** — detect when positions go out of range
* **Rebalancing** — close and re-open positions at new price ranges
* **Fee harvesting** — claim accumulated trading fees

See [Running Bots & Scripts](../vault-operations/running-bots-and-scripts.md) for setting up automation infrastructure.

## Next Steps

After initializing Raydium strategies, [allocate funds](../fund-allocation-guide/fund-allocation.md) to deploy liquidity.
