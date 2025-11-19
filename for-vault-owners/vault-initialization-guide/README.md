# Vault Initialization Guide

The Vault component (`voltr-vault`) serves as the primary interface for user interactions and asset management within the Voltr protocol. Each vault is an independent smart contract that manages deposits, withdrawals, LP token issuance, and adaptor interactions.

## Core Features

### Asset Management

* **Total Asset Tracking**: Maintains real-time accounting of all assets under management
* **Idle Asset Management**: Tracks assets not currently deployed to strategies
* **Asset Valuation**: Calculates current value of total assets including generated yields
* **Maximum Capacity**: Enforces configurable deposit caps to manage risk

### LP Token Mechanics

* **Token Issuance**: Mints LP tokens representing proportional shares of the vault
* **Share Price Calculation**: Determines LP token value based on total assets
* **Token Burning**: Handles redemption of LP tokens during withdrawals
* **Share-Based Accounting**: Tracks user ownership through LP token balances

## Fee Structure

**Performance Fees**:

* Calculated on realized profits
* Configurable percentage
* Automatically collected and distributed

**Management Fees**:

* Regular fees on assets under management
* Customizable fee rates
* Periodic collection mechanism
