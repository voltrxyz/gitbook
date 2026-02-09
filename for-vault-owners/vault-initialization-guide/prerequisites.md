# Prerequisites

Before creating a vault, ensure you have the following:

## 1. SOL for Transaction Fees

You need approximately **0.15 SOL** for vault creation (account rent) plus additional SOL for ongoing transaction fees (strategy initialization, fund allocation, etc.).

{% hint style="info" %}
The admin keypair pays for vault creation. The manager keypair pays for fund allocation transactions. Make sure **both** keypairs are funded with SOL.
{% endhint %}

## 2. Solana RPC Endpoint

A reliable Solana RPC endpoint is required for all on-chain operations. Recommended providers:

* [Helius](https://helius.dev)
* [Triton](https://triton.one)
* [QuickNode](https://quicknode.com)

## 3. Admin and Manager Keypairs

Voltr enforces role separation between admin and manager:

| Role | Responsibilities |
|------|-----------------|
| **Admin** | Add/remove adaptors, initialize strategies, update vault config, calibrate high water mark |
| **Manager** | Allocate funds between strategies, claim protocol rewards |

{% hint style="warning" %}
**Use separate keypairs** for admin and manager. This limits the blast radius if one key is compromised. The admin controls vault structure; the manager controls fund movement.
{% endhint %}

## 4. Voltr SDK & Dependencies

Install the SDK and required libraries:

```bash
npm install @voltr/vault-sdk @solana/web3.js @coral-xyz/anchor
```

Explore SDK documentation:

{% embed url="https://voltrxyz.github.io/vault-sdk/" %}

## 5. Asset Token Mint Address

Know the mint address of the SPL token your vault will accept (e.g., USDC: `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v`).

{% hint style="info" %}
You can also create a vault via the UI at [vaults.ranger.finance/create](https://vaults.ranger.finance/create) without installing the SDK. See [Quick Start (UI)](../getting-started/quick-start-ui.md).
{% endhint %}
