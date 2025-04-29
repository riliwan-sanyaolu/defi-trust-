# DefiTrust: Decentralized Reputation-Based Lending Protocol

**Built for the Stacks blockchain with native Bitcoin integration.**

## Overview

**DefiTrust** is a decentralized lending protocol enabling trust-minimized borrowing and lending based on a dynamic, on-chain credit scoring system. Unlike traditional DeFi platforms that rely solely on collateralization, DefiTrust uses historical borrower behavior to reduce required collateral and adjust interest rates, rewarding users who demonstrate financial reliability over time.

The protocol is implemented in **Clarity**, the smart contract language for the **Stacks blockchain**, which brings smart contracts to Bitcoin without compromising its security or finality.

## Features

- **Dynamic Credit Scoring**: Users are scored based on repayment behavior. Higher scores reduce collateral requirements and interest rates.
- **Reputation-Based Lending**: Encourages responsible borrowing via tiered benefits for repeat, successful repayments.
- **Permissionless Access**: No intermediaries or centralized approval required.
- **Collateralized Loans**: Uses STX as collateral with safe over-collateralization based on credit risk.
- **Admin Controls**: Contract owner can mark loans as defaulted after due height.
- **Bitcoin Compatibility**: Designed for Stacks, inheriting Bitcoin's security guarantees.

## Contract Structure

### Constants

- `MIN-SCORE`: `u50` – Minimum credit score.
- `MAX-SCORE`: `u100` – Maximum achievable credit score.
- `MIN-LOAN-SCORE`: `u70` – Minimum score to be eligible for a loan.

### Error Codes

Error handling uses explicit `err` constants (e.g., `ERR-UNAUTHORIZED`, `ERR-INSUFFICIENT-SCORE`).

## Core Data Models

### `UserScores` (map)

Tracks user credit profile:

```clojure
{
  score: uint,
  total-borrowed: uint,
  total-repaid: uint,
  loans-taken: uint,
  loans-repaid: uint,
  last-update: uint
}
```

### `Loans` (map)

Stores individual loan records:

```clojure
{
  borrower: principal,
  amount: uint,
  collateral: uint,
  due-height: uint,
  interest-rate: uint,
  is-active: bool,
  is-defaulted: bool,
  repaid-amount: uint
}
```

### `UserLoans` (map)

Holds a list of up to 20 active loan IDs per user:

```clojure
{ active-loans: (list 20 uint) }
```

## Public Functions

### `initialize-score`

Initializes the user's credit score to the minimum. Must be called before requesting a loan.

### `request-loan (amount, collateral, duration)`

- Creates a new loan.
- Validates score and collateral requirements.
- Transfers collateral and disburses STX loan.

Returns: `loan-id`

### `repay-loan (loan-id, amount)`

- Enables partial or full repayment.
- Repayment is tracked.
- Collateral returned and credit score improved upon full repayment.

## 🔒 Admin Functions

### `mark-loan-defaulted (loan-id)`

- Callable only by contract owner.
- Flags a loan as defaulted if overdue.
- Reduces user credit score and disables the loan.

## Private Utility Functions

### `calculate-required-collateral`

Dynamically calculates required collateral based on credit score.

### `calculate-interest-rate`

Assigns interest rates inversely proportional to the credit score.

### `calculate-total-due`

Computes total due amount including interest.

### `update-credit-score`

Adjusts credit score based on loan performance (success/failure).

### `update-user-loans`

Maintains user's active loan list with size limit of 20.

## Read-Only Functions

- `get-user-score (user)`: Retrieves the credit profile.
- `get-loan (loan-id)`: Returns loan details.
- `get-user-active-loans (user)`: Lists current loans of a user.

## Security & Validations

- All loan operations ensure:
  - The user has an initialized score.
  - The credit score is sufficient.
  - Loan duration is within bounds (max ~1 year).
  - Collateral covers risk as determined by creditworthiness.
- Admin operations are restricted to the contract owner.
- Safe overflows are avoided using Clarity’s numeric safety.

## Credit Score Mechanics

| Behavior             | Score Change |
|----------------------|--------------|
| On-time Repayment    | +2 (max 100) |
| Loan Default         | -10 (min 50) |

Score adjustments affect:

- Required collateral
- Interest rates
- Eligibility for new loans

## Economic Model

- **Interest Rate Formula**:  

  ```clojure

  interest-rate = base-rate - (score * 5 / 100)
  ```

  Base rate is 10%, scaling down for better scores.

- **Collateral Requirement**:  

  ```clojure
  required = amount * (100 - (score * 50 / 100)) / 100
  ```

This creates a **trust curve** where users are rewarded for responsible borrowing with cheaper and easier credit access.

## Example Flow

1. User calls `initialize-score`.
2. Requests loan via `request-loan` with STX as collateral.
3. Repays via `repay-loan`, can be partial or full.
4. Admin can call `mark-loan-defaulted` after due height if unpaid.
5. Scores and access conditions are updated dynamically.

## Requirements

- **Stacks blockchain**
- **Clarity smart contract language**
- **STX for loans and collateral**

## Future Considerations

- Multi-asset support for collateral (e.g., SIP-010 tokens)
- Loan refinancing
- Off-chain credit bridge via Oracles
- Credit delegation & syndication
