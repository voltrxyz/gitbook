# Drift Strategies

The Drift adaptor enables integration with Drift Protocol for perpetual futures trading and JLP (Jupiter Liquidity Provider) strategies. These are distinct from Drift Spot lending, which uses the [lending adaptor](lending-strategies.md).

## Strategy Types

| Strategy | Description | Script Repo |
|----------|-----------|-------------|
| **Drift Perps** | Trade perpetual futures on Drift | [voltrxyz/drift-scripts](https://github.com/voltrxyz/drift-scripts) |
| **Drift JLP** | Provide liquidity to Drift's JLP market | [voltrxyz/drift-scripts](https://github.com/voltrxyz/drift-scripts) |

## Drift Perpetuals

### Overview

The Drift perps strategy allows vault funds to be used for perpetual futures trading on Drift Protocol. This is an active management strategy — the vault manager (or their bot) actively opens and closes positions.

### Key Setup Requirements

1. **Delegate account**: You must set up a delegate account with **trade-only permissions** on Drift. This allows the manager to execute trades without full account control.
2. **Sub-account**: Drift uses sub-accounts for position isolation. The strategy creates a sub-account on the vault's behalf.
3. **Market configuration**: You specify which Drift perpetual markets the strategy can trade.

### Initialization

Use the [voltrxyz/drift-scripts](https://github.com/voltrxyz/drift-scripts) repository for Drift perps initialization. The script handles:

* Creating the Drift sub-account
* Setting up delegate permissions
* Initializing the strategy PDA with the correct remaining accounts

{% hint style="warning" %}
**Drift perps carry significant risk.** Leveraged perpetual trading can result in losses exceeding the initial allocation. Ensure your vault's risk profile and marketing materials clearly communicate this to depositors.
{% endhint %}

### Post-Initialization

After initializing the Drift perps strategy:

* Fund allocation moves assets from the vault to the Drift sub-account
* The manager (or automation) opens/closes perpetual positions
* Profits and losses are reflected when funds are withdrawn back to the vault

## Drift JLP Market

### Overview

The Drift JLP strategy deposits vault funds into Drift's JLP market — a liquidity pool that backs Drift's perpetual futures. JLP holders earn fees from Drift trading activity.

### Key Differences from Perps

| Aspect | Drift Perps | Drift JLP |
|--------|------------|-----------|
| **Management** | Active (manager trades) | Passive (liquidity provision) |
| **Market index** | Uses perp market index | Uses JLP market index |
| **Risk** | Leveraged directional risk | IL risk + protocol exposure |
| **Returns** | From trading PnL | From trading fees |

### Initialization

Use the [voltrxyz/drift-scripts](https://github.com/voltrxyz/drift-scripts) repository for Drift JLP initialization. The JLP strategy uses a **different market index configuration** from perps — make sure you use the JLP-specific initialization script.

## Common Setup for Both

### Prerequisites

1. Your vault must already be created
2. The **Drift adaptor** must be added to your vault (not the lending adaptor)
3. Admin keypair for strategy initialization

### Adding the Drift Adaptor

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

### Automation

Drift strategies typically require automation for:

* **Perps**: Executing trades, managing positions, risk monitoring
* **JLP**: Monitoring liquidity, rebalancing if needed

See [Running Bots & Scripts](../vault-operations/running-bots-and-scripts.md) for setting up automation infrastructure.

## Next Steps

After initializing Drift strategies, [allocate funds](../fund-allocation-guide/fund-allocation.md) to start deploying capital.
