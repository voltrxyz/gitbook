# Frontend Integration Guide

This guide covers how to build a custom frontend for vault deposits and withdrawals using the Voltr SDK.

{% hint style="info" %}
**Don't need a custom frontend?** Users can deposit and withdraw through the Ranger UI at [voltr.xyz](https://voltr.xyz) once your vault is [indexed and listed](go-to-market/indexing-and-listing.md). Building a custom frontend is optional.
{% endhint %}

## When to Build a Custom Frontend

* You want to embed vault functionality in your own application
* You need custom UI/UX beyond what the Ranger UI provides
* You want to offer a branded experience for your users

## Approaches

| Approach            | Best For                                                        | Tools                                                                    |
| ------------------- | --------------------------------------------------------------- | ------------------------------------------------------------------------ |
| **UI Template**     | Quickest path to a branded vault frontend                       | [voltrxyz/basic-ui](https://github.com/voltrxyz/basic-ui)               |
| **SDK integration** | Full control, real-time custom querying                         | `@voltr/vault-sdk`                                                       |
| **API + SDK**       | Read data via API, and create transactions.                     | Voltr API                                                                |

{% hint style="info" %}
**Recommended**: Start with the [open-source UI template](https://github.com/voltrxyz/basic-ui) for a production-ready frontend out of the box, or use the [Voltr API](https://api.voltr.xyz/docs) if you need to integrate vault functionality into an existing application.
{% endhint %}

## Open-Source Vaults UI Template

The fastest way to launch a custom vault frontend is to fork the **Voltr Vaults UI Template** — a self-contained, production-ready Next.js application available open source at [github.com/voltrxyz/basic-ui](https://github.com/voltrxyz/basic-ui).

### What's Included

* **Vault Discovery** — Browse and filter vaults with real-time stats (TVL, capacity, fees)
* **Deposits & Withdrawals** — Full deposit and two-step withdrawal flow with wallet integration
* **Vault Manager** — Create new vaults and update configuration directly from the UI
* **Real-time Updates** — On-chain account listeners for live position tracking

### Tech Stack

* Next.js 15 (App Router), React 19, TypeScript, Tailwind CSS with shadcn/ui
* `@voltr/vault-sdk` and `@solana/web3.js` for on-chain interaction
* All transactions built client-side — no backend required

### Quick Start

```bash
git clone https://github.com/voltrxyz/basic-ui.git
cd basic-ui
pnpm install
cp .env.example .env
```

Set your environment variables in `.env`:

```
NEXT_PUBLIC_RPC_URL=<your-solana-rpc-url>
NEXT_PUBLIC_VAULT_PUBKEYS=<comma-separated-vault-pubkeys>
```

Then start the development server:

```bash
pnpm dev
```

The template is optimized for deployment on Vercel — push to GitHub and import into Vercel with the same environment variables.
