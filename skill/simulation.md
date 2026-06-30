# Simulation-First Execution Guidelines

## Overview
In the **Safe On-Chain Solana Agent Skill**, every state-changing transaction must be simulated against the live mainnet state before a signature is requested. This `Simulation-First` approach is the primary defense against drained wallets, MEV sandwich attacks, and wasted compute fees.

## Why Simulate?
Standard agentic tools often construct a transaction and broadcast it blindly. On Solana, where state changes rapidly, this can lead to:
- **Failed Transactions:** Wasted lamports due to unexpected state changes or insufficient compute.
- **Slippage Violations:** Executing a swap at a terrible price because the agent hallucinated the parameters.
- **Account Errors:** Attempting to interact with uninitialized Token Accounts.

## The Simulation Workflow

### 1. Construct the Transaction
The AI Agent or underlying tool constructs the `VersionedTransaction` or `Transaction` object as usual. **Do not sign it yet.**

### 2. Run the RPC Simulation
Send the unsigned transaction to the RPC node using the `simulateTransaction` method:
```typescript
const { value: simResult } = await connection.simulateTransaction(transaction, {
    sigVerify: false,
    replaceRecentBlockhash: true,
    commitment: 'confirmed'
});
```

### 3. Pre-Flight Checks on Simulation Results

The middleware must evaluate the `simResult` against the following criteria:

- **Program Errors (`simResult.err`):** 
  If `err` is not null, the transaction will fail. Map the error code (e.g., `0x1` for insufficient funds) using the `error-handling` module and prompt the agent to adjust parameters.
  
- **Compute Unit (CU) Budget:** 
  Check `simResult.unitsConsumed`. Ensure the transaction's requested compute budget is greater than the consumed units (with a safe 10-15% buffer). If it exceeds the limit, instruct the agent to add a `ComputeBudgetProgram.setComputeUnitLimit` instruction.
  
- **Slippage & Token Balances:** 
  Parse the simulation logs (`simResult.logs`) or use the `return_data` (if the program supports it) to verify the exact output amount of a swap. Compare this against the user's maximum slippage tolerance. If it violates the bounds, abort the transaction and recalculate the route.

## Agent Instructions
When an AI agent interacts with this module, it must follow these rules:
1. **Never skip simulation:** If a tool tries to bypass simulation, block the execution.
2. **Read the Logs:** If a simulation fails, do not just say "it failed". Read the `simResult.logs` to determine exactly *which* instruction failed and *why*, then self-correct.
3. **Prompt the User:** If a simulation succeeds but the slippage is dangerously high (e.g., > 5%), pause execution and ask for explicit human confirmation.
