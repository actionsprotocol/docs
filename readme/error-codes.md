# Error Codes

This document lists all custom error codes used in the Actions Prediction Market Protocol.

## Error Types

The program defines custom error codes in the `PredictionMarketError` enum to provide clear feedback when operations fail.

## Complete Error List

| Code | Error Name                   | Description                                           | When It Occurs                                       |
| ---- | ---------------------------- | ----------------------------------------------------- | ---------------------------------------------------- |
| 6000 | `PlatformAlreadyInitialized` | Platform is already initialized                       | Trying to initialize an already-initialized platform |
| 6001 | `PlatformNotInitialized`     | Platform is not initialized                           | Using a platform that hasn't been set up             |
| 6002 | `GlobalConfigNotInitialized` | Global configuration is not initialized               | Trying to use protocol before global config is set   |
| 6003 | `Unauthorized`               | Only the platform authority can perform this action   | Access control violation                             |
| 6004 | `MarketNameTooLong`          | Market name is too long                               | Market name exceeds 100 characters                   |
| 6005 | `MarketDescriptionTooLong`   | Market description is too long                        | Market description exceeds 500 characters            |
| 6006 | `MetadataUriTooLong`         | Metadata URI is too long                              | Metadata URI exceeds 200 characters                  |
| 6007 | `MarketExpired`              | Market has expired and cannot accept new bets         | Trying to bet on an expired market                   |
| 6008 | `InvalidMarketState`         | Market is not in the correct state for this operation | State transition violation                           |
| 6009 | `NotMarketCreator`           | Only the market creator can perform this action       | Non-creator trying creator-only action               |
| 6010 | `MarketStillActive`          | Market is still active and cannot be resolved         | Trying to resolve before expiry                      |
| 6011 | `MarketAlreadyFinalized`     | Market has already been finalized                     | Trying to finalize twice                             |
| 6012 | `ParticipantNotFound`        | Participant not found in the market                   | Looking up non-existent participant                  |
| 6013 | `AlreadyClaimed`             | Participant has already been claimed                  | Double-claiming attempt                              |
| 6014 | `NotAWinner`                 | Participant is not a winner                           | Non-winner trying to claim                           |
| 6015 | `InvalidBetAmount`           | Invalid bet amount                                    | Bet amount is 0 or invalid                           |
| 6016 | `InsufficientFunds`          | Insufficient funds                                    | Not enough lamports for operation                    |
| 6017 | `InvalidFeePercentage`       | Fee percentage cannot exceed 100%                     | Fee percentage > 10000 basis points                  |
| 6018 | `CannotCancelResolvedMarket` | Market cannot be cancelled after it has been resolved | Trying to cancel finalized market                    |
| 6019 | `InvalidExpirySlot`          | Expiry slot must be in the future                     | Expiry slot in past or too far future                |
| 6020 | `ParticipantRegistryFull`    | Participant registry is full                          | Too many participants                                |
| 6021 | `ParticipantAlreadyExists`   | Participant already exists in the market              | Duplicate participant entry                          |
| 6022 | `ConflictingBetOption`       | Cannot bet on different option than previous bet      | Trying to switch bet sides                           |
| 6023 | `NoParticipants`             | Market has no participants                            | Operating on empty market                            |
| 6024 | `NoWinners`                  | Market has no winners                                 | No winning participants                              |
| 6025 | `InvalidMarketAccount`       | Invalid market account                                | Wrong market account provided                        |

## Error Categories

### Platform & Configuration Errors (6000-6003)

These errors relate to platform setup and access control:

```rust
PlatformAlreadyInitialized  // 6000
PlatformNotInitialized     // 6001
GlobalConfigNotInitialized // 6002
Unauthorized               // 6003
```

**Common Causes:**

* Trying to initialize platform twice
* Using uninitialized platform/global config
* Wrong wallet trying privileged operations

### Market Creation Errors (6004-6006, 6019)

These errors occur during market creation with invalid parameters:

```rust
MarketNameTooLong        // 6004
MarketDescriptionTooLong // 6005
MetadataUriTooLong       // 6006
InvalidExpirySlot        // 6019
```

**Limits:**

* Market name: ≤ 100 characters
* Description: ≤ 500 characters
* Metadata URI: ≤ 200 characters
* Expiry slot: Must be future, ≤ 24 hours

### Market State Errors (6007-6011, 6018)

These errors relate to market lifecycle and state management:

```rust
MarketExpired              // 6007
InvalidMarketState         // 6008
NotMarketCreator          // 6009
MarketStillActive         // 6010
MarketAlreadyFinalized    // 6011
CannotCancelResolvedMarket // 6018
```

**State Flow Issues:**

* Betting on expired markets
* Wrong state transitions
* Non-creators trying to resolve
* Resolving active markets

### Participant & Betting Errors (6012-6017, 6020-6024)

These errors occur during betting and participant management:

```rust
ParticipantNotFound        // 6012
AlreadyClaimed            // 6013
NotAWinner               // 6014
InvalidBetAmount         // 6015
InsufficientFunds        // 6016
InvalidFeePercentage     // 6017
ParticipantRegistryFull  // 6020
ParticipantAlreadyExists // 6021
ConflictingBetOption     // 6022
NoParticipants           // 6023
NoWinners               // 6024
```

**Betting Issues:**

* Invalid bet amounts (0 or negative)
* Insufficient SOL for bet + fees
* Switching bet sides (YES ↔ NO)
* Registry size limits

### Account Validation Errors (6025)

```rust
InvalidMarketAccount      // 6025
```

**Account Issues:**

* Wrong market account provided
* Account doesn't match expected PDA

## Common Error Scenarios

### Market Creation

```typescript
// ❌ Will fail with MarketNameTooLong (6004)
const longName = "A".repeat(101);

// ❌ Will fail with InvalidExpirySlot (6019)
const pastSlot = currentSlot - 1000;

// ✅ Correct usage
const validName = "Will Bitcoin reach $100k?"; // ≤ 100 chars
const futureSlot = currentSlot + 216000;       // ≤ 24 hours
```

### Betting

```typescript
// ❌ Will fail with InvalidBetAmount (6015)
const zeroBet = 0;

// ❌ Will fail with ConflictingBetOption (6022) if user previously bet NO
const yesOption = true; // But user previously bet false

// ✅ Correct usage
const validBet = 1000000000; // 1 SOL
const sameOption = false;    // Same as previous bet to add more
```

### Market Resolution

```typescript
// ❌ Will fail with MarketStillActive (6010)
const activeMarket = { state: MarketState.Active };

// ❌ Will fail with NotMarketCreator (6009)
const wrongCreator = someOtherWallet;

// ✅ Correct usage
const decidingMarket = { state: MarketState.Deciding };
const correctCreator = marketCreatorWallet;
```

### Claiming

```typescript
// ❌ Will fail with NotAWinner (6014)
const losingParticipant = { option: false, market: { winning_option: true } };

// ❌ Will fail with AlreadyClaimed (6013)
const claimAccount = { claimed: true };

// ✅ Correct usage
const winningParticipant = { option: true, market: { winning_option: true } };
const unclaimedAccount = null; // No claim account exists yet
```
