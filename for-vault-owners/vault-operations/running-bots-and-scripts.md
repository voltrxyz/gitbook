# Running Bots & Scripts

Vault operations require automation for optimal performance. This page covers why automation is needed, available scripts, and infrastructure recommendations.

{% hint style="danger" %}
**You must host your own automation.** Voltr/Ranger does not provide managed bot services. You are responsible for deploying, running, and monitoring your own scripts and bots.
{% endhint %}

## Why Automation Is Needed

| Task                    | Why It Needs Automation                                                        |
| ----------------------- | ------------------------------------------------------------------------------ |
| **Rebalancing**         | Yield rates change frequently; manual rebalancing misses optimal allocations   |
| **Reward claiming**     | Protocol rewards accrue continuously; manual claiming leaves value uncollected |
| **Reward swapping**     | Claimed reward tokens need to be swapped to base asset to compound             |
| **Position monitoring** | Raydium CLMM positions go out-of-range; Drift positions need risk monitoring   |
| **Fee harvesting**      | Accumulated fees should be harvested periodically                              |

## Script Repositories

The Voltr team provides reference scripts for common operations:

| Repository                                                                                      | Use Case                                                   |
| ----------------------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| [voltrxyz/lend-scripts](https://github.com/voltrxyz/lend-scripts)                               | Lending strategy init (Project0, Save)                     |
| [voltrxyz/kamino-scripts](https://github.com/voltrxyz/kamino-scripts)                           | Kamino strategy init, rewards claiming                     |
| [voltrxyz/drift-scripts](https://github.com/voltrxyz/drift-scripts)                             | Drift vaults/lend/perps strategy init, position management |
| [voltrxyz/spot-scripts](https://github.com/voltrxyz/spot-scripts)                               | Jupiter Swap/Lend strategy init                            |
| [voltrxyz/client-raydium-clmm-scripts](https://github.com/voltrxyz/client-raydium-clmm-scripts) | Raydium CLMM strategy init                                 |
| [voltrxyz/trustful-scripts](https://github.com/voltrxyz/trustful-scripts)                       | Trustful adaptor strategy init                             |
| [voltrxyz/rebalance-bot-template](https://github.com/voltrxyz/rebalance-bot-template)           | Production-ready rebalance bot (equal-weight allocation)   |

## Rebalance Bot Template

The [rebalance-bot-template](https://github.com/voltrxyz/rebalance-bot-template) is a production-ready bot that handles the core automation tasks listed above. It distributes funds equally across lending strategies on a fixed schedule and includes:

- **Rebalance loop** — equal-weight allocation across all strategies, triggered on interval and on new deposits
- **Refresh loop** — keeps on-chain receipt values up to date
- **Harvest fee loop** — collects protocol/admin/manager fees
- **Claim reward loops** — claims Kamino farm rewards and swaps them back via Jupiter

Supports Drift, Jupiter Lend, Kamino Market, and Kamino Vault strategies out of the box.

[![Run on Replit](https://replit.com/badge/github/voltrxyz/rebalance-bot-template)](https://replit.com/github/voltrxyz/rebalance-bot-template)

```bash
git clone https://github.com/voltrxyz/rebalance-bot-template.git
cd rebalance-bot-template
pnpm install
cp .env.example .env   # fill in your vault addresses and keypair
pnpm run build && pnpm start
```

See the [repository README](https://github.com/voltrxyz/rebalance-bot-template#readme) for full configuration options and Replit deployment instructions.

## Script Structure Example

A basic rebalancing script:

```typescript
import { VoltrClient } from "@voltr/vault-sdk";
import { Connection, Keypair } from "@solana/web3.js";

async function main() {
  const connection = new Connection(process.env.RPC_URL!);
  const client = new VoltrClient(connection);

  const managerKp = Keypair.fromSecretKey(
    Uint8Array.from(JSON.parse(process.env.MANAGER_KEY!))
  );

  // 1. Check current allocations
  const vaultData = await client.getVault(vaultPubkey);
  const idleBalance = vaultData.asset.totalValue; // simplified

  // 2. Check strategy yields (via API or protocol SDKs)
  // 3. Determine optimal allocation
  // 4. Execute rebalance transactions

  console.log("Rebalance complete");
}

main().catch(console.error);
```

## Key Considerations

* **Gas budget**: Ensure your manager wallet has enough SOL for all automated transactions. Monitor and top up regularly.
* **Error handling**: Scripts should handle transaction failures gracefully (retry logic, alerting).
* **Rate limiting**: Respect RPC provider rate limits. Use exponential backoff on failures.
* **Idempotency**: Design scripts to be safely re-runnable in case of partial failures.
