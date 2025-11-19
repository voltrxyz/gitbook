# Solana Agent Kit

This guide explains how AI agents can interact with Voltr vaults using the Solana Agent Kit. The integration enables agents to perform strategy deposits and withdrawals on behalf of vault managers.

### Available Functions

#### 1. Get Asset Amount

```typescript
async function voltrGetAssetAmount(
    agent: SolanaAgentKit,
    vault: PublicKey
): Promise<string>
```

* Retrieves the total asset amount and current amounts for each strategy in a vault
* Returns a JSON string containing:
  * `totalAmount`: Total assets in the vault
  * `strategies`: Array of strategy information including:
    * `strategyId`: Public key of the strategy
    * `amount`: Current amount in the strategy

#### 2. Deposit Strategy

```typescript
async function voltrDepositStrategy(
    agent: SolanaAgentKit,
    depositAmount: BN,
    vault: PublicKey,
    strategy: PublicKey
): Promise<string>
```

* Deposits assets into a specific strategy within a vault
* Returns transaction signature
* Handles remaining accounts fetching automatically
* Uses confirmed commitment level for transaction confirmation

#### 3. Withdraw Strategy

```typescript
async function voltrWithdrawStrategy(
    agent: SolanaAgentKit,
    withdrawAmount: BN,
    vault: PublicKey,
    strategy: PublicKey
): Promise<string>
```

* Withdraws assets from a specific strategy within a vault
* Returns transaction signature
* Supports both regular SPL tokens and Token-2022 program
* Automatically fetches and handles remaining accounts

### Example Autonomous Mode Implementation

In our example, the autonomous mode operates on a fixed interval (default 10 seconds) and follows a two-phase cycle:

#### Even Iterations (Analysis Phase):

1. Fetches total vault amount and strategy amounts
2. Records strategy IDs and their respective amounts
3. Calculates total strategy amounts
4. Determines excess amount (vault total - sum of strategy amounts)
5. Identifies strategy with lowest amount

#### Odd Iterations (Action Phase):

1. Reviews excess amount from previous analysis
2. If excess amount > 0:
   * Deposits excess into the strategy with lowest amount
3. If excess amount = 0:
   * No action taken
4. Waits for next interval before starting new cycle

```typescript
async function runAutonomousMode(agent: any, config: any, interval = 10) {
  console.log("Starting autonomous mode...");

  let iterations = 0;

  while (true) {
    try {
      const evenThought =
        "1. Get the total amount and amount for each strategy of a Voltr vault: 3ab3KVY9GbDbUUbRnYNSBDQqABTDup7HmdgADHGpB8Bq. " +
        "2. Take note of the strategies id with the their respective amount. " +
        "3. Calculate the sum of all strategy amounts. " +
        "4. Subtract that sum from the vault total to get the excess amount. " +
        "5. Indicate the excess amount and the strategy id with the lowest amount.";

      const oddThought =
        "Using the latest excess amount, if it is 0 then do nothing. " +
        "Else if it is more than 0, deposit the excess amount into the lowest strategy and vault: 3ab3KVY9GbDbUUbRnYNSBDQqABTDup7HmdgADHGpB8Bq.";

      const thought = iterations % 2 === 0 ? evenThought : oddThought;

      const stream = await agent.stream(
        { messages: [new HumanMessage(thought)] },
        config,
      );

      for await (const chunk of stream) {
        if ("agent" in chunk) {
          for (const message of chunk.agent.messages) {
            console.log(message.content);
          }
        } else if ("tools" in chunk) {
          for (const message of chunk.tools.messages) {
            console.log(message.content);
          }
        }
        console.log("-------------------");
      }

      iterations++;

      await new Promise((resolve) => setTimeout(resolve, interval * 1000));
    } catch (error) {
      if (error instanceof Error) {
        console.error("Error:", error.message);
      }
      process.exit(1);
    }
  }
}
```

### Security Considerations

1. Secure storage of private keys
2. Validation of all input parameters
3. Verification of account authorities
4. Transaction amount verification
5. Error handling for failed transactions

### Limitations

* Operations are limited to available vault strategies
* Transactions require appropriate account permissions
* Network latency may affect operation timing
* Rate limits may apply to API calls
