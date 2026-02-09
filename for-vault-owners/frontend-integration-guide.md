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

| Approach            | Best For                                                 | Tools              |
| ------------------- | -------------------------------------------------------- | ------------------ |
| **SDK integration** | Full control, real-time custom querying                  | `@voltr/vault-sdk` |
| **API + SDK**       | Read data via API, and create transactions transactions. | Voltr API          |

{% hint style="info" %}
**Recommended**: Use the [Voltr API](https://api.voltr.xyz/docs) for reading vault data (APY, share price, TVL) and for building deposit/withdraw transactions. This separates concerns and simplifies your frontend code.
{% endhint %}
