# Running Bots & Scripts

Vault operations require automation for optimal performance. This page covers why automation is needed, available scripts, and infrastructure recommendations.

{% hint style="danger" %}
**You must host your own automation.** Voltr/Ranger does not provide managed bot services. You are responsible for deploying, running, and monitoring your own scripts and bots.
{% endhint %}

## Why Automation Is Needed

| Task | Why It Needs Automation |
|------|----------------------|
| **Rebalancing** | Yield rates change frequently; manual rebalancing misses optimal allocations |
| **Reward claiming** | Protocol rewards accrue continuously; manual claiming leaves value uncollected |
| **Reward swapping** | Claimed reward tokens need to be swapped to base asset to compound |
| **Position monitoring** | Raydium CLMM positions go out-of-range; Drift positions need risk monitoring |
| **Fee harvesting** | Accumulated fees should be harvested periodically |

## Script Repositories

The Voltr team provides reference scripts for common operations:

| Repository | Use Case |
|-----------|---------|
| [voltrxyz/lend-scripts](https://github.com/voltrxyz/lend-scripts) | Lending strategy init (Project0, Save) |
| [voltrxyz/kamino-scripts](https://github.com/voltrxyz/kamino-scripts) | Kamino strategy init, rewards claiming |
| [voltrxyz/drift-scripts](https://github.com/voltrxyz/drift-scripts) | Drift vaults/lend/perps strategy init, position management |
| [voltrxyz/spot-scripts](https://github.com/voltrxyz/spot-scripts) | Jupiter Swap/Lend strategy init |
| [voltrxyz/client-raydium-clmm-scripts](https://github.com/voltrxyz/client-raydium-clmm-scripts) | Raydium CLMM strategy init |
| [voltrxyz/trustful-scripts](https://github.com/voltrxyz/trustful-scripts) | Trustful adaptor strategy init |

## Infrastructure Recommendations

### Minimum Setup

For a basic vault with lending strategies:

* A server or cloud VM (AWS EC2, DigitalOcean, Hetzner, etc.)
* Node.js runtime
* Cron jobs for scheduled tasks
* Secure keypair storage

### Example Cron Setup

```bash
# Rebalance allocations every 6 hours
0 */6 * * * cd /path/to/scripts && node rebalance.js >> /var/log/vault/rebalance.log 2>&1

# Claim and swap rewards daily at 00:00 UTC
0 0 * * * cd /path/to/scripts && node claim-rewards.js >> /var/log/vault/rewards.log 2>&1

# Harvest fees weekly on Monday at 00:00 UTC
0 0 * * 1 cd /path/to/scripts && node harvest-fees.js >> /var/log/vault/fees.log 2>&1

# Monitor vault health every 15 minutes
*/15 * * * * cd /path/to/scripts && node monitor.js >> /var/log/vault/monitor.log 2>&1
```

### Production Recommendations

For production vaults managing significant TVL:

| Component | Recommendation |
|-----------|---------------|
| **Compute** | Dedicated VM with monitoring (not serverless — you need persistent connections) |
| **Keypair security** | Use environment variables or secret managers (AWS Secrets Manager, HashiCorp Vault) — never store keypairs on disk in plaintext |
| **Monitoring** | Set up alerts for failed transactions, low SOL balance, performance anomalies |
| **Logging** | Centralized logging (CloudWatch, Datadog, or similar) for debugging |
| **Redundancy** | Consider failover for critical operations |
| **RPC** | Use a paid RPC provider with rate limits appropriate for your bot frequency |

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
