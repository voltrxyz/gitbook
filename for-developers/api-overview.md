# API Overview

Welcome to the Voltr REST API. This API provides developers with the tools to query on-chain and off-chain vault data, calculate metrics, and construct transactions for interacting with Voltr vaults.

### Base URL

All API endpoints are relative to the following base URL: [https://api.voltr.xyz](https://api.voltr.xyz/)

### Authentication

The Voltr API is public and does not require an API key for access.

### Key Concept: Transaction Building

A critical feature of the Voltr API is its approach to transaction creation. Instead of executing transactions on the server, the API **builds and returns an unsigned, serialized versioned transaction** as a base58 encoded string.

This design ensures that user private keys are never required on the backend, maintaining a non-custodial and secure user experience.

The typical workflow for a developer is:

1. **Request**: Your application sends a `POST` request to a transaction creation endpoint (e.g., `/vault/{pubkey}/deposit`).
2. **Receive**: The API returns a JSON response containing the serialized transaction string.
3. **Sign & Send**: Your client-side application deserializes this string, signs it with the user's wallet, and sends it to the Solana network.

```json
{
  "success": true,
  "transaction": "AQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABAAI..."
}
```

### Interactive Documentation

For live testing and a detailed, interactive view of all endpoints, please visit our Swagger documentation: [https://api.voltr.xyz/docs](https://api.voltr.xyz/docs)
