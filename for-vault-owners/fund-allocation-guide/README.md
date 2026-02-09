# Fund Allocation Guide

This guide explains how vault managers allocate funds between the vault's idle account and initialized strategies.

{% hint style="info" %}
**SDK for manager actions, API for user actions**: Use the SDK with server-side scripts for fund allocation (requires the manager keypair). Use the [Voltr API](https://api.voltr.xyz/docs) for reading vault data and user-facing queries.
{% endhint %}

## Overview

As a vault manager, you have the authority to:

* Deploy idle funds from the vault into active strategies
* Withdraw funds from strategies back to the vault
* Monitor current allocations and performance
* Claim protocol rewards

## Key Concepts

### Idle vs. Deployed Funds

* **Idle Funds**: Assets sitting in the vault's idle token account, not earning yield
* **Deployed Funds**: Assets actively working in strategies (lending, trading, LP)
* **Total Assets**: Sum of idle and deployed funds

### Fund Flow

```
User Deposits → Vault Idle Account → Strategy Accounts
Strategy Accounts → Vault Idle Account → User Withdrawals
```

{% hint style="warning" %}
Keep some funds idle to service user withdrawals. If all funds are deployed, users may experience delays during the withdrawal process.
{% endhint %}

## Section Contents

{% content-ref url="prerequisites.md" %}
[Prerequisites](prerequisites.md)
{% endcontent-ref %}

{% content-ref url="fund-allocation.md" %}
[Fund Allocation](fund-allocation.md)
{% endcontent-ref %}

{% content-ref url="rewards-and-emissions.md" %}
[Rewards & Emissions](rewards-and-emissions.md)
{% endcontent-ref %}
