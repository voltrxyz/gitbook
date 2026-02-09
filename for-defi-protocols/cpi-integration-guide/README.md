# CPI Integration Guide

This guide covers Cross-Program Invocation (CPI) integration with the Voltr Vault program. If your protocol needs to deposit into or withdraw from Voltr vaults on-chain, use these CPI instructions.

{% hint style="info" %}
**When to use CPI vs. the TypeScript SDK**: CPI is for on-chain programs that need to interact with Voltr vaults programmatically (e.g., a router or aggregator). If you are building off-chain automation or a frontend, use the [TypeScript SDK](https://voltrxyz.github.io/vault-sdk/) instead.
{% endhint %}

### How It Works

Users deposit assets into a vault and receive LP tokens representing their share. Withdrawals follow a **two-step process** to ensure vault stability and manage liquidity:

1. **Request withdrawal** — locks LP tokens into an escrow receipt
2. **Claim withdrawal** — after the vault's waiting period, burns the locked LP tokens and returns the underlying assets

```
Deposit:  User Assets ──► Vault ──► LP Tokens to User

Withdraw: LP Tokens ──► Escrow Receipt ──(waiting period)──► Assets to User
```

### Deployed Address

| Network | Program Address                               |
| ------- | --------------------------------------------- |
| Mainnet | `vVoLTRjQmtFpiYoegx285Ze4gsLJ8ZxgFKVcuvmG1a8` |

### CPI Instructions

| Instruction | Purpose | Reference |
| --- | --- | --- |
| `deposit_vault` | Deposit assets and receive LP tokens | [Deposit](deposit.md) |
| `request_withdraw_vault` | Lock LP tokens into an escrow receipt | [Request Withdraw](request-withdraw.md) |
| `withdraw_vault` | Claim assets after the waiting period | [Withdraw](withdraw.md) |

### PDA Derivation

All PDAs are derived from the Voltr Vault program (`vVoLTRjQmtFpiYoegx285Ze4gsLJ8ZxgFKVcuvmG1a8`):

| Account | Seeds |
| --- | --- |
| `protocol` | `["protocol"]` |
| `vault_asset_idle_auth` | `["vault_asset_idle_auth", vault_key]` |
| `vault_lp_mint_auth` | `["vault_lp_mint_auth", vault_key]` |
| `request_withdraw_vault_receipt` | `["request_withdraw_vault_receipt", vault_key, user_key]` |

{% hint style="warning" %}
Your program does **not** need to provide PDA signer seeds for the vault's internal PDAs. The Voltr Vault program handles its own `invoke_signed` calls internally. You only need to pass the correct PDA addresses in the account list.
{% endhint %}

### Error Handling

Your program should handle the following errors from the Voltr Vault program:

| Error | Cause |
| --- | --- |
| `InvalidAmount` | Input amount is zero or invalid |
| `MaxCapExceeded` | Deposit would exceed the vault's maximum capacity |
| `WithdrawalNotYetAvailable` | `withdraw_vault` called before the waiting period has passed |
| `OperationNotAllowed` | The protocol has globally disabled the attempted operation |

### Reference Repository

Full reference implementations are available at [github.com/voltrxyz/vault-cpi](https://github.com/voltrxyz/vault-cpi).

{% content-ref url="deposit.md" %}
[deposit.md](deposit.md)
{% endcontent-ref %}

{% content-ref url="request-withdraw.md" %}
[request-withdraw.md](request-withdraw.md)
{% endcontent-ref %}

{% content-ref url="withdraw.md" %}
[withdraw.md](withdraw.md)
{% endcontent-ref %}
