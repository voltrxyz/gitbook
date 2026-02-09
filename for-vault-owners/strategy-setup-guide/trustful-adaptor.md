# Trustful Adaptor

The trustful adaptor enables vault integration with **off-chain venues** that cannot be verified on-chain. This includes centralized exchanges (CEXs), OTC desks, MPC wallets, and other custodial solutions.

{% hint style="danger" %}
**Critical trust assumption**: The trustful adaptor relies entirely on the manager's honest reporting of off-chain positions. There is **no on-chain verification** of the actual position value. Depositors must trust the manager.
{% endhint %}

## How It Works

1. **Deposit**: The manager allocates funds from the vault to the trustful strategy, which transfers tokens to a designated off-chain wallet
2. **Off-chain management**: The manager deploys funds to the external venue (CEX account, MPC wallet, etc.)
3. **Position reporting**: The manager periodically updates the strategy's reported value to reflect the off-chain position
4. **Withdrawal**: The manager returns funds from the external venue, deposits them back, and withdraws from the strategy

### Trust Model

```
Vault ──deposit──→ Trustful Strategy ──transfer──→ Off-chain Wallet
                        ↑
              Manager reports position value
```

The vault's total value includes the **reported** value of trustful positions. If the manager misreports, the vault's share price will be incorrect, affecting all depositors.

## Use Cases

### CEX Trading

Deploy vault funds to centralized exchanges for:

* Market making on centralized order books
* Arbitrage between DEX and CEX
* Trading strategies not available on-chain

### OTC Deals

Execute large trades off-chain to avoid slippage on DEX pools.

### MPC/Custodian Bridges

Hold funds in institutional custody solutions (e.g., Fireblocks, Copper) for:

* Institutional-grade custody
* Cross-chain operations
* Compliance-required holding structures

## Setup

### Prerequisites

1. Your vault must already be created
2. The **trustful adaptor** must be added to your vault
3. A designated off-chain wallet address for receiving funds
4. A robust process for regular position reporting

### Adding the Trustful Adaptor

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

## Risk Disclosure

When using the trustful adaptor, you **must** clearly communicate the following to depositors:

* Funds may be held off-chain with no on-chain verification
* Position values are self-reported by the manager
* There is counterparty risk with the off-chain venue
* Funds may not be instantly withdrawable (depends on off-chain venue liquidity)

{% hint style="warning" %}
**Recommendation**: Only use the trustful adaptor when there is established trust between the vault operator and depositors (e.g., institutional clients, known counterparties). For retail-facing vaults, prefer on-chain strategies where positions can be verified.
{% endhint %}

## Next Steps

After initializing the trustful strategy, see [Fund Allocation](../fund-allocation-guide/fund-allocation.md) for deploying funds.
