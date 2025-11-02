# DRT-to-SOL Batch Conversion Escrow

A decentralized escrow system that allows users to pool their DRT-to-SOL conversions to minimize gas fees and slippage by executing batch conversions when a collective target is reached.

## User Story

**As a** DRT token holder with a small amount to convert  
**I want to** lock my DRT tokens in a vault until enough users join to reach a conversion threshold  
**So that** I can benefit from lower gas fees and better exchange rates through a single large batch conversion

### Scenario

Sarah has 150 DRT tokens she wants to convert to SOL, but converting such a small amount individually would cost her 0.4 SOL in gas fees and result in 3-5% slippage due to low liquidity. Instead, she deposits her 150 DRT into the batch conversion vault with a target of 1000 DRT.

Over the next 3 days, other users (Mike with 200 DRT, Emma with 300 DRT, and others) also deposit their tokens into the vault. Once the 1000 DRT threshold is reached, the escrow automatically executes a single conversion, getting a much better rate with only 0.8% slippage and 0.6 SOL total gas fees split among all participants. Sarah receives her proportional share of SOL (15% of the total) and saves approximately 85% on gas fees while getting a 2% better conversion rate.

## Architecture

```
┌──────────────┐
│   User A     │──┐
│  (150 DRT)   │  │
└──────────────┘  │
                  │
┌──────────────┐  │    ┌─────────────────────────────────┐
│   User B     │──┼───▶│         Vault Contract          │
│  (200 DRT)   │  │    │                                 │
└──────────────┘  │    │  • Target: 1000 DRT             │
                  │    │  • Current: 1000 DRT ✓          │
┌──────────────┐  │    │  • Deadline: 7 days             │
│   User C     │──┤    │  • Participants: 5              │
│  (300 DRT)   │  │    │  • DRT Token Balance: 1000      │
└──────────────┘  │    └──────────┬──────────────────────┘
                  │               │
┌──────────────┐  │               │ Threshold Reached
│   User N     │──┘               │ Transfer to Escrow
│  (350 DRT)   │                  ▼
└──────────────┘       ┌─────────────────────────────────┐
                       │   Escrow Smart Contract         │
                       │                                 │
                       │  • Holds 1000 DRT               │
                       │  • Executes conversion          │
                       │  • Manages distribution         │
                       └──────────┬──────────────────────┘
                                  │
                                  │ Execute Conversion
                                  ▼
                       ┌─────────────────────────────────┐
                       │   DEX Aggregator / AMM          │
                       │                                 │
                       │  • Execute: 1000 DRT → 85 SOL   │
                       │  • Optimized routing            │
                       │  • Minimal slippage             │
                       └──────────┬──────────────────────┘
                                  │
                                  │ Conversion Complete
                                  ▼
                       ┌─────────────────────────────────┐
                       │   Distribution Contract         │
                       │                                 │
                       │  Calculate & Distribute:        │
                       │  • User A: 12.75 SOL (15%)     │
                       │  • User B: 17.00 SOL (20%)     │
                       │  • User C: 25.50 SOL (30%)     │
                       │  • User N: 29.75 SOL (35%)     │
                       └─────────────────────────────────┘
```

## Flow Diagram

```
1. DEPOSIT PHASE
   User → [Approve DRT] → [Deposit to Vault] → [Record Position]
                                    ↓
                           [Update Pool Total]
                                    ↓
                        [Check Threshold Met?]
                         ├─ No → Wait
                         └─ Yes → Continue

2. TRANSFER TO ESCROW
   [Threshold Met] → [Transfer DRT from Vault to Escrow]
                                    ↓
                          [Lock Funds in Escrow]

3. CONVERSION PHASE
   [Escrow Locked] → [Query DEX for Best Rate]
                              ↓
                    [Execute Batch Swap]
                    (1000 DRT → 85 SOL)
                              ↓
                    [Transfer SOL to Escrow]

4. DISTRIBUTION PHASE
   [Calculate Proportions] → For each user:
                            (user_DRT / total_DRT) × total_SOL
                                    ↓
                          [Transfer SOL to Users]
                                    ↓
                            [Emit Completion Event]

5. TIMEOUT SCENARIO (if threshold not met)
   [Deadline Passed] → [Vault Allows Withdrawals] → [Users Reclaim DRT from Vault]
```

## Key Features

- **Vault-based deposits**: All user deposits accumulate in the vault
- **Threshold-based execution**: Conversion only happens when target amount is reached
- **Escrow security**: Funds transferred to escrow for conversion execution
- **Proportional distribution**: Each user receives SOL based on their contribution percentage
- **Time-bound**: Automatic refund from vault if threshold not met within deadline
- **Gas optimization**: Single transaction replaces multiple individual conversions
- **Better rates**: Larger swap amounts achieve better liquidity and lower slippage

## Technical Components

**Vault Contract**

- Accepts DRT deposits from users
- Tracks participant contributions
- Monitors threshold and deadline
- Transfers funds to escrow when threshold met
- Handles refunds if deadline expires

**Escrow Contract**

- Receives DRT from vault
- Executes conversion through DEX
- Holds SOL temporarily
- Coordinates distribution

**Conversion Module**

- Integrates with DEX aggregator
- Executes optimized swap
- Handles slippage protection

**Distribution Module**

- Calculates proportional shares
- Distributes SOL to participants
- Emits events for transparency

## Example Comparison

| Metric          | Individual Conversion | Batch Escrow      |
| --------------- | --------------------- | ----------------- |
| Amount          | 150 DRT               | 150 DRT (of 1000) |
| Gas Fee         | 0.4 SOL               | 0.06 SOL (shared) |
| Slippage        | 4%                    | 0.8%              |
| SOL Received    | 11.5 SOL              | 12.75 SOL         |
| **Net Benefit** | -                     | **+10.9%**        |

---

Built with security patterns from Aegis, accounting principles from Moneta, and governance from Themis.
