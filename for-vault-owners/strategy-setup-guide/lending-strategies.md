# Lending Strategies

Lending strategies deploy vault funds into lending protocols to earn interest. The lending adaptor supports multiple protocols, each with its own initialization script.

## Supported Protocols

| Protocol | Type | Script Repo |
|----------|------|-------------|
| **Kamino** (Vault) | Deposit into Kamino vault products | [voltrxyz/kamino-scripts](https://github.com/voltrxyz/kamino-scripts) |
| **Kamino** (Lending Market) | Deposit into Kamino lending reserves | [voltrxyz/kamino-scripts](https://github.com/voltrxyz/kamino-scripts) |
| **Project0** | Deposit into Project0 lending pools | [voltrxyz/lend-scripts](https://github.com/voltrxyz/lend-scripts) |
| **Save** (formerly Solend) | Deposit into Save lending reserves | [voltrxyz/lend-scripts](https://github.com/voltrxyz/lend-scripts) |
| **Drift Lend** | Lend on Drift lending market | [voltrxyz/drift-scripts](https://github.com/voltrxyz/drift-scripts) |
| **Jupiter Lend** | Deposit into Jupiter lending | [voltrxyz/spot-scripts](https://github.com/voltrxyz/spot-scripts) |

## General Flow

All lending strategies follow the same pattern:

1. **Add the lending adaptor** to your vault (see [Strategy Setup Guide](README.md))
2. **Derive the strategy PDA** from the counterparty token account
3. **Initialize the strategy** with protocol-specific remaining accounts
4. **Allocate funds** using the fund allocation flow

### Strategy PDA Derivation

```typescript
import { SEEDS, LENDING_ADAPTOR_PROGRAM_ID } from "@voltr/vault-sdk";
import { PublicKey } from "@solana/web3.js";

// The counterparty token account is the protocol's token account
// that receives deposits from the vault
const counterPartyTa = new PublicKey("...");

const [strategy] = PublicKey.findProgramAddressSync(
  [SEEDS.STRATEGY, counterPartyTa.toBuffer()],
  LENDING_ADAPTOR_PROGRAM_ID
);
```

### Generic Initialization

```typescript
const initStrategyIx = await client.createInitializeStrategyIx(
  {}, // Optional initialization args
  {
    payer: adminKp.publicKey,
    vault: vaultPubkey,
    manager: managerKp.publicKey,
    strategy,
    remainingAccounts: [
      // Protocol-specific accounts — see protocol sections below
    ],
  }
);
```

## Kamino

{% hint style="warning" %}
Kamino has **two different integration types** — Kamino Vault and Kamino Lending Market. They use different scripts and different remaining accounts. Make sure you use the correct one.
{% endhint %}

### Kamino Vault

Deposits into Kamino's managed vault products. The Kamino vault handles its own rebalancing and strategy management.

**Script**: See [voltrxyz/kamino-scripts](https://github.com/voltrxyz/kamino-scripts) — use the vault initialization script.

### Kamino Lending Market

Deposits directly into Kamino lending reserves. You control which reserve to deposit into.

**Script**: See [voltrxyz/kamino-scripts](https://github.com/voltrxyz/kamino-scripts) — use the lending market initialization script.

## Project0

Deposits into Project0 lending pools for the vault's asset.

**Script**: See [voltrxyz/lend-scripts](https://github.com/voltrxyz/lend-scripts) for the Project0 initialization script.

## Save (formerly Solend)

Deposits into Save lending reserves. Supports all Save lending pools for the vault's asset.

**Script**: See [voltrxyz/lend-scripts](https://github.com/voltrxyz/lend-scripts) for the Save initialization script.

## Drift Lend

Lends assets on Drift's lending market for interest income.

**Script**: See [voltrxyz/drift-scripts](https://github.com/voltrxyz/drift-scripts) for the Drift Lend initialization script.

## Jupiter Lend

Deposits into Jupiter's lending product.

**Script**: See [voltrxyz/spot-scripts](https://github.com/voltrxyz/spot-scripts) for the Jupiter Lend initialization script.

## Best Practices

* **Diversify**: Spread funds across multiple lending protocols to reduce single-protocol risk
* **Monitor rates**: Lending rates change frequently — set up monitoring to rebalance when rates shift
* **Start small**: Test with a small allocation before deploying significant funds
* **Check protocol health**: Verify the lending protocol's utilization rates and liquidity before deploying

## Next Steps

After initializing lending strategies, [allocate funds](../fund-allocation-guide/fund-allocation.md) to start earning yield.
