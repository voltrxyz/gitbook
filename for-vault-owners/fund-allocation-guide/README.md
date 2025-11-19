# Fund Allocation Guide

## Voltr Protocol - Fund Allocation Guide for Vault Managers

This guide explains how vault managers can efficiently allocate funds across different strategies in the Voltr Protocol.

### Overview

As a vault manager, you have the authority to:

* Deploy idle funds from the vault into active strategies
* Withdraw funds from strategies back to the vault
* Monitor current allocations and performance
* Handle performance fees and rewards

### Key Concepts

#### 1. Idle vs. Deployed Funds

* **Idle Funds**: Assets sitting in the vault's idle token account
* **Deployed Funds**: Assets actively working in strategies
* **Total Assets**: Sum of idle and deployed funds

#### 2. Fund Flow

```
User Deposits → Vault Idle Account → Strategy Accounts
Strategy Accounts → Vault Idle Account → User Withdrawals
```
