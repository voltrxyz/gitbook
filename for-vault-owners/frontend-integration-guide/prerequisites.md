# Prerequisites

Before integrating vault functionality, ensure you have:

1. A Solana wallet integration (e.g., Phantom, Solflare)
2. Access to a Solana RPC endpoint (e.g., Helius)
3. The Voltr SDK and other required libraries installed in your project:

    ```bash
    npm install @voltr/vault-sdk @solana/web3.js @coral-xyz/anchor @solana/spl-token
    ```

{% hint style="info" %}
**Alternative**: For reading vault data (APY, TVL, share price), you can use the [Voltr API](https://api.voltr.xyz/docs) directly without the SDK. The API provides REST endpoints that work from any frontend framework.
{% endhint %}

Explore SDK documentation:

{% embed url="https://voltrxyz.github.io/vault-sdk/" %}

