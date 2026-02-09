# Rewards & Emissions

Many DeFi protocols distribute rewards beyond the base yield (e.g., protocol token emissions, trading incentives). This page explains how to handle rewards and emissions for your vault.

## Claiming Protocol Rewards

When protocols distribute reward tokens (e.g., JTO from Jito, MNDE from Marginfi), you need to:

1. **Claim the reward tokens** from the protocol using a claim instruction
2. **Swap the reward tokens** back to the vault's base asset (e.g., swap JTO → USDC)
3. The swapped amount is now part of the vault's idle funds and increases the vault's total value

### Example: Claim and Swap Flow

```
Protocol Rewards (JTO) → Claim → Swap via Jupiter → USDC → Vault Idle Account
```

{% hint style="info" %}
Reward claiming is protocol-specific. Use the protocol's script repository for the correct claim instruction and accounts. After claiming, use Jupiter swap (via the swap adaptor) to convert rewards to the vault's base asset.
{% endhint %}

### Who Can Claim

Reward claiming is typically a **manager** action. The manager's keypair signs the claim transaction.

## Custom Emissions / Incentives

Voltr does not currently have a native emissions program for distributing additional tokens to vault depositors. However, there is a workaround for vault owners who want to incentivize deposits:

### Workaround: Deposit + Burn LP

1. **Deposit additional tokens** into the vault (as the base asset)
2. **Burn the LP tokens** you receive from the deposit

This effectively:
* Increases the vault's total assets without increasing LP supply
* Raises the share price for all existing LP holders
* Functions as a "distribution" to depositors proportional to their holdings

{% hint style="warning" %}
This workaround is a manual process and requires you to fund the incentives from your own resources. There is no automated emissions schedule or distribution mechanism built into the protocol.
{% endhint %}

### How to Execute

```typescript
import { VoltrClient } from "@voltr/vault-sdk";
import { Connection, Keypair, PublicKey } from "@solana/web3.js";
import { BN } from "@coral-xyz/anchor";

const connection = new Connection("your-rpc-url");
const client = new VoltrClient(connection);

// Step 1: Deposit the incentive amount into the vault
const depositIx = await client.createDepositVaultIx(
  new BN("1000000"), // Amount to deposit as incentive
  {
    vault: vaultPubkey,
    userTransferAuthority: incentivePayerKp.publicKey,
    vaultAssetMint: assetMint,
    assetTokenProgram: TOKEN_PROGRAM_ID,
  }
);

// Step 2: Burn the LP tokens received
// (Use SPL token burn instruction on the LP tokens)
```

## Reward Frequency

How often you claim rewards depends on:

* **Protocol emission schedule** — some distribute per epoch, others continuously
* **Gas costs** — each claim transaction costs SOL
* **Compounding benefit** — more frequent claiming = more compounding, but higher gas

{% hint style="info" %}
**Recommendation**: Set up automated reward claiming as part of your bot infrastructure. Daily or weekly claiming is typical for most strategies. See [Running Bots & Scripts](../vault-operations/running-bots-and-scripts.md).
{% endhint %}

## Script Repositories

| Repository | Reward Claiming Support |
|-----------|------------------------|
| [voltrxyz/lend-scripts](https://github.com/voltrxyz/lend-scripts) | Lending protocol rewards |
| [voltrxyz/kamino-scripts](https://github.com/voltrxyz/kamino-scripts) | Kamino reward claiming |
| [voltrxyz/drift-scripts](https://github.com/voltrxyz/drift-scripts) | Drift reward claiming |
| [voltrxyz/spot-scripts](https://github.com/voltrxyz/spot-scripts) | Jupiter Lend rewards |
