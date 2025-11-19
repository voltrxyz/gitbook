# Prerequisites

Before creating a vault, ensure you have:

1. A Solana wallet with sufficient SOL for vault creation and transaction fees (\~0.15 SOL)
2. Access to a Solana RPC endpoint (e.g., Helius)
3. Required keypair files for admin and manager roles
   1. Admins — capable of adding new adaptors, initializing, and changing vault configurations
   2. Managers — capable of allocating funds in the vault between initialized strategies
4.  The Voltr SDK and other required libraries installed in your project:

    ```bash
    npm install @voltr/vault-sdk @solana/web3.js @coral-xyz/anchor
    ```

Explore SDK documentation:

{% embed url="https://voltrxyz.github.io/vault-sdk/" %}

