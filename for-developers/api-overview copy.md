Of course. Adding a dedicated API reference section to your GitBook is a great way to support developers. I will create the necessary files based on the backend code you provided and structure them for clarity.

Here are the new files and the required update to your `SUMMARY.md`.

---

### File to Modify

First, add the new API Reference section to your main table of contents.

#### File: `SUMMARY.md`

**Add the following section to the end of the file:**
```markdown
## For Developers

* [API Overview](for-developers/api-overview.md)
* [Swagger Documentation](https://api.voltr.xyz/docs)
* [Vaults Endpoint](for-developers/endpoints/vaults.md)
* [Vault Endpoint](for-developers/endpoints/vault.md)
```

---

### New Files to Create

Next, create the following new files in the specified directories.

#### File: `for-developers/api-overview.md`
```markdown
# API Overview

Welcome to the Voltr REST API. This API provides developers with the tools to query on-chain and off-chain vault data, calculate metrics, and construct transactions for interacting with Voltr vaults.

### Base URL

All API endpoints are relative to the following base URL:

```
https://api.voltr.xyz
```

### Authentication

The Voltr API is public and does not require an API key for access.

### Key Concept: Transaction Building

A critical feature of the Voltr API is its approach to transaction creation. Instead of executing transactions on the server, the API **builds and returns an unsigned, serialized versioned transaction** as a base58 encoded string.

This design ensures that user private keys are never required on the backend, maintaining a non-custodial and secure user experience.

The typical workflow for a developer is:
1.  **Request**: Your application sends a `POST` request to a transaction creation endpoint (e.g., `/vault/{pubkey}/deposit`).
2.  **Receive**: The API returns a JSON response containing the serialized transaction string.
3.  **Sign & Send**: Your client-side application deserializes this string, signs it with the user's wallet, and sends it to the Solana network.

```json
{
  "success": true,
  "transaction": "AQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABAAI..."
}
```

### Interactive Documentation

For live testing and a detailed, interactive view of all endpoints, please visit our Swagger documentation:

{% embed url="https://api.voltr.xyz/docs" %}
```

#### File: `for-developers/endpoints/vaults.md`
```markdown
# Vaults Endpoint

The `/vaults` endpoint provides methods for retrieving aggregated data across all Voltr vaults.

## Get All Vaults

Retrieves a list of all active Voltr vaults, including both native and external (e.g., Kamino) integrations. This endpoint is ideal for displaying a comprehensive directory of available vaults.

-   **Method**: `GET`
-   **Path**: `/vaults`

#### Success Response (200 OK)

Returns a JSON object containing a list of vault details.

```json
{
  "success": true,
  "vaults": [
    {
      "pubkey": "8oUwteX3SJELMGDPTEuLqz1WSd58yMdkQ6s4hKwB42nJ",
      "name": "Voltr USDC APY Maximizer",
      "theme": "Lending",
      "org": {
        "name": "Voltr",
        "icon": "..."
      },
      "asset": {
        "name": "USDC",
        "icon": "...",
        "decimals": 6,
        "price": 0.9998
      },
      "allocations": [
        {
          "orgName": "Kamino",
          "orgIcon": "..."
        },
        {
          "orgName": "Drift",
          "orgIcon": "..."
        }
      ],
      "age": 1699999999,
      "capacity": 1000000000000,
      "tvl": 50000000000,
      "apy": 12.34
    }
  ]
}
```

---

## Get Total TVL

Calculates and returns the total value locked (TVL) in USD across all vaults, along with a breakdown by asset.

-   **Method**: `GET`
-   **Path**: `/vaults/tvl`

#### Success Response (200 OK)

Returns the total TVL and an asset-by-asset breakdown.

```json
{
  "success": true,
  "data": {
    "totalTvlUsd": 1234567.89,
    "breakdown": [
      {
        "asset": "USDC",
        "tvl": 1000000.5,
        "price": 0.9998,
        "tvlUsd": 999800.5
      },
      {
        "asset": "SOL",
        "tvl": 1500.2,
        "price": 156.48,
        "tvlUsd": 234759.39
      }
    ],
    "lastUpdated": "2025-11-20T10:00:00.000Z"
  }
}
```
```

#### File: `for-developers/endpoints/vault.md`
```markdown
# Vault Endpoint

The `/vault/{pubkey}` endpoint provides detailed information and interaction methods for a specific vault.

### Path Parameter

| Name     | Type   | Description                    |
| :------- | :----- | :----------------------------- |
| `pubkey` | string | The public key of the Voltr vault. |

---

## Vault Data

### Get Vault Details

Retrieves comprehensive information for a single vault, including configuration, asset details, APY, and current allocations.

-   **Method**: `GET`
-   **Path**: `/vault/{pubkey}`

---

### Get Historical Fee Earned

Calculates the total performance and management fees earned by a vault within a specified time range, denominated in the vault's LP token.

-   **Method**: `GET`
-   **Path**: `/vault/{pubkey}/fee-earned`

#### Query Parameters

| Name      | Type   | Description                                           |
| :-------- | :----- | :---------------------------------------------------- |
| `startTs` | number | Optional. Start timestamp (Unix seconds). Defaults to 0. |
| `endTs`   | number | Optional. End timestamp (Unix seconds). Defaults to now. |

---

### Get Historical Share Price

Retrieves the interpolated share price (asset per LP token) for a vault at a specific point in time.

-   **Method**: `GET`
-   **Path**: `/vault/{pubkey}/share-price`

#### Query Parameters

| Name | Type   | Description                                        |
| :--- | :----- | :------------------------------------------------- |
| `ts` | number | Optional. Timestamp (Unix seconds). Defaults to now. |

---

## User-Specific Data

### Get User Vault Balance

Calculates and returns a user's total balance in a vault, denominated in the vault's underlying asset.

-   **Method**: `GET`
-   **Path**: `/vault/{pubkey}/user/{userPubkey}/balance`

### Get User Pending Withdrawal

Checks for and returns a user's pending withdrawal request from a vault, including the withdrawable amount and timestamp.

-   **Method**: `GET`
-   **Path**: `/vault/{pubkey}/user/{userPubkey}/pending-withdrawal`

---

## Transaction Creation

{% hint style="info" %}
**Important:** These endpoints return a serialized, unsigned transaction. Your client application must sign this transaction with the user's wallet and broadcast it to the Solana network. See the [API Overview](for-developers/api-overview.md) for more details.
{% endhint %}

### Create Deposit Transaction

Builds a transaction to deposit assets into a vault.

-   **Method**: `POST`
-   **Path**: `/vault/{pubkey}/deposit`

#### Body Parameters

| Name                | Type    | Description                                      |
| :------------------ | :------ | :----------------------------------------------- |
| `userPubkey`        | string  | The user's wallet public key.                    |
| `lamportAmount`     | string  | The amount of the asset to deposit in lamports. |
| `assetMint`         | string  | Optional. The mint address of the asset.         |
| `assetTokenProgram` | string  | Optional. The token program for the asset.       |

---

### Create Request Withdrawal Transaction

Builds a transaction to initiate a withdrawal request. Funds enter a waiting period before they can be claimed.

-   **Method**: `POST`
-   **Path**: `/vault/{pubkey}/request-withdrawal`

#### Body Parameters

| Name           | Type    | Description                                                                 |
| :------------- | :------ | :-------------------------------------------------------------------------- |
| `userPubkey`   | string  | The user's wallet public key.                                               |
| `lamportAmount`| string  | The amount to withdraw.                                                     |
| `isAmountInLp` | boolean | Optional. `true` if amount is in LP tokens, `false` if in underlying asset. |
| `isWithdrawAll`| boolean | Optional. `true` to withdraw the user's entire balance.                     |

---

### Create Cancel Withdrawal Transaction

Builds a transaction to cancel a pending withdrawal request.

-   **Method**: `POST`
-   **Path**: `/vault/{pubkey}/cancel-withdrawal`

#### Body Parameters

| Name         | Type   | Description                   |
| :----------- | :----- | :---------------------------- |
| `userPubkey` | string | The user's wallet public key. |

---

### Create Claim Withdrawal Transaction

Builds a transaction to claim funds from a completed withdrawal request.

-   **Method**: `POST`
-   **Path**: `/vault/{pubkey}/withdraw`

#### Body Parameters

| Name                | Type   | Description                                |
| :------------------ | :----- | :----------------------------------------- |
| `userPubkey`        | string | The user's wallet public key.              |
| `assetMint`         | string | Optional. The mint address of the asset.   |
| `assetTokenProgram` | string | Optional. The token program for the asset. |

---

### Create Direct Withdrawal Transaction

Builds a transaction for an instant "direct withdrawal" from supported vaults, bypassing the standard waiting period.

-   **Method**: `POST`
-   **Path**: `/vault/{pubkey}/direct-withdraw`

#### Body Parameters

| Name                | Type    | Description                                     |
| :------------------ | :------ | :---------------------------------------------- |
| `userPubkey`        | string  | The user's wallet public key.                   |
| `lamportAmount`     | string  | The amount of the asset to withdraw in lamports.|
| `isWithdrawAll`     | boolean | `true` to withdraw the user's entire balance.   |
| `assetMint`         | string  | Optional. The mint address of the asset.        |
| `assetTokenProgram` | string  | Optional. The token program for the asset.      |
```