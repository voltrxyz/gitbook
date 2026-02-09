# Frontend Integration Guide

This guide covers how to build a custom frontend for vault deposits and withdrawals using the Voltr SDK.

{% hint style="info" %}
**Don't need a custom frontend?** Users can deposit and withdraw through the Ranger UI at [voltr.xyz](https://voltr.xyz) once your vault is [indexed and listed](../go-to-market/indexing-and-listing.md). Building a custom frontend is optional.
{% endhint %}

## When to Build a Custom Frontend

* You want to embed vault functionality in your own application
* You need custom UI/UX beyond what the Ranger UI provides
* You want to offer a branded experience for your users

## Approaches

| Approach | Best For | Tools |
|----------|---------|-------|
| **SDK integration** | Full control, custom transaction building | `@voltr/vault-sdk` |
| **API + SDK** | Read data via API, submit transactions via SDK | Voltr API + SDK |

{% hint style="info" %}
**Recommended**: Use the [Voltr API](https://api.voltr.xyz/docs) for reading vault data (APY, share price, TVL) and the SDK for building deposit/withdraw transactions. This separates concerns and simplifies your frontend code.
{% endhint %}

## Sections

{% content-ref url="prerequisites.md" %}
[Prerequisites](prerequisites.md)
{% endcontent-ref %}

{% content-ref url="frontend-integration.md" %}
[Frontend Integration](frontend-integration.md)
{% endcontent-ref %}
