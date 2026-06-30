---
name: safe-onchain-agent-skill
description: Provides simulation-first safety middleware, intelligent error recovery, and scoped permissions for executing reliable transactions on Solana.
---

# Safe On-Chain Solana Agent Skill

## Overview
The Safe On-Chain Solana Agent Skill acts as a rigorous safety middleware layer for autonomous agents interacting with the Solana blockchain. It ensures that every agent-proposed action is mathematically verified via local pre-flight checks and mainnet simulation prior to broadcast, mitigating risks such as drained wallets, MEV attacks from incorrect slippage, and stalled workflows.

## When to Use
Activate this skill whenever the user or agent needs to:
- Propose, construct, or execute any state-changing transaction on Solana (e.g., token swaps, NFT mints, staking).
- Implement autonomous, long-running monitoring loops that require self-healing or error recovery.
- Validate token balances, slippage bounds, compute unit budgets, or rent exemptions before executing on-chain logic.
- Execute actions using session keys or scoped allowances instead of god-mode private keys.

## Core Principles
1. **Simulation-First Execution:** Treat every transaction as untrusted. Always simulate against live mainnet state before signing to validate exact balances and parameters.
2. **Scoped Permissions & Wallet Hygiene:** Avoid exposing root private keys. Utilize session keys with granular, time-bound, or amount-bounded allowances.
3. **Intelligent Error Recovery:** Do not panic on cryptic Solana program errors. Automatically translate and recover from transient RPC issues, blockhash expirations, and minor slippage deviations.
4. **Pre-Flight Validation:** Verify required state, token accounts, and fee lamports before initiating complex on-chain interactions.

## Available Knowledge Modules
When diving deeper into specific safety mechanics, reference the following supporting modules (if progressively loaded):
- `modules/simulation.md`: Guidelines for structuring and interpreting Solana transaction simulations.
- `modules/permissions.md`: Implementing session keys, zero-trust policies, and granular allowances.
- `modules/error-handling.md`: Semantic mapping of common Solana errors (e.g., `0x1`, `0x1771`) to self-healing retry strategies.

## Integration Notes & Progressive Loading
To maintain a token-efficient context window, do not load all safety modules simultaneously.
- **Default State:** Load this `SKILL.md` entry point to understand basic safety principles.
- **Action State:** Progressively load specific modules (e.g., `modules/simulation.md` or specific tool schemas) only when an active on-chain intent is detected.
- **Deep Composability:** This skill is designed to wrap existing tools in the Solana AI Kit (like Jupiter or Meteora plugins) without modifying their underlying execution logic. Always prioritize using the safe executor wrappers over direct tool invocation.

## Usage Guidelines for the AI
1. **Never Blindly Sign:** Before proposing a final transaction, explicitly state: "I will now simulate this transaction to ensure safety."
2. **Handle Slippage Gracefully:** If a simulation fails due to slippage, recalculate based on the feedback and propose a corrected transaction.
3. **Explain Adjustments:** If the safety middleware alters a parameter (like compute units or slippage), clearly explain to the user why the adjustment was made.
