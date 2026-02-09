# Gas Fees & ATA Costs

Understanding the cost structure of vault operations helps you budget and plan your infrastructure effectively.

## Who Pays What

| Operation | Payer | Cost Type |
|-----------|-------|----------|
| **Vault creation** | Admin | Account rent (~0.15 SOL) + gas |
| **Add adaptor** | Admin | Gas (~0.000005 SOL) |
| **Initialize strategy** | Admin | Account rent + gas |
| **Create LP metadata** | Admin | Account rent + gas |
| **Update vault config** | Admin | Gas |
| **Deposit to strategy** | Manager | Gas + ATA rent (if new) |
| **Withdraw from strategy** | Manager | Gas + ATA rent (if new) |
| **Claim rewards** | Manager | Gas |
| **User deposit to vault** | User | Gas + ATA rent (if new LP account) |
| **User withdraw from vault** | User | Gas |
| **Harvest fees** | Anyone (permissionless) | Gas |

## Associated Token Account (ATA) Costs

ATAs are Solana accounts that hold SPL tokens. Each ATA requires rent to exist on-chain.

### ATA Rent Cost

Each ATA costs approximately **0.00203928 SOL** in rent (rent-exempt minimum).

### ATA Behavior in Voltr

{% hint style="info" %}
**ATAs are NOT closed** between deposit and withdraw operations. Once an ATA is created for a vault-strategy pair, it persists and can be reused. The rent cost is a **one-time expense**.
{% endhint %}

### Who Creates Which ATAs

| ATA | Created By | When |
|-----|-----------|------|
| Vault strategy asset ATA | Manager | First deposit to a strategy |
| User LP token ATA | User | First deposit to vault |
| User asset ATA | User | First withdrawal from vault |

## One-Time vs. Ongoing Costs

### One-Time Costs (Setup)

| Cost | Amount | Payer |
|------|--------|-------|
| Vault account rent | ~0.15 SOL | Admin |
| LP metadata account rent | ~0.01 SOL | Admin |
| Per-strategy ATA rent | ~0.002 SOL each | Manager |
| Strategy account rent | ~0.005 SOL each | Admin |
| Lookup table rent (if needed) | ~0.003 SOL | Admin/Manager |

### Ongoing Costs (Operations)

| Operation | Approximate Gas Cost | Frequency |
|-----------|---------------------|-----------|
| Deposit to strategy | ~0.000005 SOL | Per rebalance |
| Withdraw from strategy | ~0.000005 SOL | Per rebalance |
| Claim rewards | ~0.000005 SOL | Per claim cycle |
| Swap rewards (Jupiter) | ~0.000005 SOL | Per claim cycle |
| Harvest fees | ~0.000005 SOL | Per harvest |

{% hint style="info" %}
Gas costs on Solana are extremely low (~5000 lamports per transaction). The primary costs are account rent (one-time) and RPC provider fees (ongoing).
{% endhint %}

## Operational Cost Estimation

### Example: USDC Lending Vault with 3 Strategies

**One-time setup**:
* Vault creation: 0.15 SOL
* LP metadata: 0.01 SOL
* 3 strategy ATAs: 0.006 SOL
* 3 strategy accounts: 0.015 SOL
* **Total: ~0.18 SOL**

**Monthly operations** (assuming daily rebalance + weekly reward claims):
* ~30 rebalance transactions: 0.00015 SOL
* ~4 reward claims + swaps: 0.00004 SOL
* ~4 fee harvests: 0.00002 SOL
* **Total: ~0.0002 SOL/month** (gas only)

{% hint style="info" %}
The real ongoing cost is your **infrastructure** (server, RPC provider) and **time**, not Solana gas fees. Budget accordingly.
{% endhint %}

## Tips for Managing Costs

* **Fund both keypairs**: Ensure admin and manager wallets have SOL before operations
* **Monitor SOL balances**: Set alerts when balances drop below a threshold (e.g., 0.1 SOL)
* **Batch transactions**: Combine multiple operations into single transactions where possible
* **RPC costs**: A paid RPC provider typically costs $50-200/month depending on request volume
