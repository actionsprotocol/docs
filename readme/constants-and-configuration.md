# Constants & Configuration

This document describes all constants, enums, and configuration values used in the Actions Prediction Market Protocol.

## Global Constants



### Timing Constants

```rust
// Maximum market duration - 24 hours
pub const MAX_MARKET_DURATION_SLOTS: u64 = 216000;

// Finalization timeout - 12 hours after market expires
pub const FINALIZATION_TIMEOUT_SLOTS: u64 = 108000;
```

**Timing Calculations** (assuming \~400ms per slot):

* Maximum market duration: 216,000 slots ≈ 24 hours
* Finalization window: 108,000 slots ≈ 12 hours
* Total maximum lifecycle: 36 hours from creation to auto-cancel

### String Length Limits

```rust
impl SimpleMarket {
    pub const MAX_NAME_LENGTH: usize = 100;
    pub const MAX_DESCRIPTION_LENGTH: usize = 500;
    pub const MAX_METADATA_URI_LENGTH: usize = 200;
}
```

These limits ensure account sizes remain manageable while providing sufficient space for market information.

### Fee Limits

```rust
// Maximum platform/creator fees (in basis points)
const MAX_PLATFORM_FEE: u16 = 10000; // 100%
```

* Global protocol fee capped at 5% for safety
* Platform and creator fees can theoretically be 100% but typically much lower

***

## Enums

### MarketState

Represents the current state of a prediction market.

```rust
#[derive(AnchorSerialize, AnchorDeserialize, Clone, Copy, PartialEq, Eq, Debug)]
pub enum MarketState {
    Active,                  // Market is accepting bets
    Deciding,                // Market has expired, waiting for resolution
    Finalized,               // Market resolved (yes/no/null)
    FinalizedNoParticipants, // Market resolved but had no valid participants
    AutoCanceled,            // Market auto-canceled due to deadline passing
}
```

#### State Descriptions

| State                     | Value | Description                                 | Next Possible States                  |
| ------------------------- | ----- | ------------------------------------------- | ------------------------------------- |
| `Active`                  | 0     | Market is open for betting                  | `Deciding`, `FinalizedNoParticipants` |
| `Deciding`                | 1     | Market expired, awaiting creator resolution | `Finalized`, `AutoCanceled`           |
| `Finalized`               | 2     | Market resolved with outcome                | None (terminal)                       |
| `FinalizedNoParticipants` | 3     | Resolved but no valid participants          | None (terminal)                       |
| `AutoCanceled`            | 4     | Creator failed to resolve in time           | None (terminal)                       |

#### State Transition Rules

```
         Create Market
              ↓
          [Active] ←──── Can accept bets
              ↓
         Market Expires
              ↓
          [Deciding] ←─── Creator has 12 hours to resolve
              ↓
         ┌─────────┴─────────┐
         ↓                   ↓
    [Finalized]        [AutoCanceled]
  Creator resolved    Deadline passed
```

#### Special Cases

* **Direct to FinalizedNoParticipants**: Markets with no participants or only single-sided betting
* **Auto-transition**: Markets automatically transition from Active → Deciding when they expire
* **Lazy Updates**: State transitions happen when markets are accessed, not automatically

***

## Account Size Constants

### Fixed-Size Accounts

```rust
impl GlobalConfig {
    pub const LEN: usize = 76; // 8 + 32 + 32 + 2 + 1 + 1
}

impl PlatformConfig {
    pub const LEN: usize = 77; // 8 + 32 + 2 + 2 + 32 + 1 + 1
}

impl SimpleMarket {
    pub const LEN: usize = 816; // 8 + 4+100 + 4+500 + 4+200 + 32 + 8 + 8 + 1 + 8 + 8 + 8 + 1+1 + 32 + 8 + 9 + 1
}

impl MarketVault {
    pub const LEN: usize = 41; // 8 + 32 + 1
}

impl CreatorFeeAccount {
    pub const LEN: usize = 73; // 8 + 32 + 32 + 8 + 1
}

impl ClaimAccount {
    pub const LEN: usize = 74; // 8 + 32 + 32 + 8 + 1 + 1
}
```

### Variable-Size Accounts

```rust
impl ParticipantsRegistry {
    pub const BASE_LEN: usize = 49; // 8 + 32 + 4 + 4 + 1
    
    pub fn len(&self) -> usize {
        Self::BASE_LEN + (self.participants.len() * Participant::LEN)
    }
}

impl Participant {
    pub const LEN: usize = 49; // 32 + 8 + 1 + 8
}
```

**Dynamic Growth**: ParticipantsRegistry grows as more users bet, requiring account resizing.

***

## Default Configuration Values

### Typical Fee Structure

```typescript
// Common platform configuration
const TYPICAL_PLATFORM_CONFIG = {
  globalFeePercentage: 25,    // 0.25% protocol fee
  platformFeePercentage: 50, // 0.5% platform fee
  creatorFeePercentage: 100,  // 1% creator fee
};

// Total fees: ~1.75% of each bet
```

### Market Duration Examples

```typescript
// Common market durations (in slots)
const MARKET_DURATIONS = {
  ONE_HOUR: 9000,      // ~1 hour
  FOUR_HOURS: 36000,   // ~4 hours
  TWELVE_HOURS: 108000, // ~12 hours
  ONE_DAY: 216000,     // ~24 hours (maximum)
};
```

## PDA Seed Constants

All PDA seeds used in the protocol:

```rust
// Global level
const GLOBAL_CONFIG_SEED: &[u8] = b"global_config";

// Platform level
const PLATFORM_CONFIG_SEED: &[u8] = b"platform_config";

// Market level
const MARKET_SEED: &[u8] = b"market";
const MARKET_VAULT_SEED: &[u8] = b"market_vault";
const PARTICIPANTS_SEED: &[u8] = b"participants";
const CREATOR_FEES_SEED: &[u8] = b"creator_fees";

// Participant level
const CLAIM_SEED: &[u8] = b"claim";
```

***

## Basis Points Reference

The protocol uses basis points for fee calculations:

| Percentage | Basis Points | Example Usage                    |
| ---------- | ------------ | -------------------------------- |
| 0.01%      | 1            | Minimal fee                      |
| 0.25%      | 25           | **Global protocol fee**          |
| 0.5%       | 50           | Low platform fee                 |
| 1%         | 100          | **Typical platform/creator fee** |
| 2%         | 200          | High platform fee                |
| 5%         | 500          | **Maximum global fee**           |
| 10%        | 1000         | Very high fee                    |
| 100%       | 10000        | **Maximum platform fee**         |

**Calculation Formula**:

```
amount_after_fee = bet_amount - (bet_amount * basis_points / 10000)
fee_amount = bet_amount * basis_points / 10000
```

***

## Runtime Constants

### Slot Timing Assumptions

The protocol assumes \~400ms per slot on average:

* 1 minute ≈ 150 slots
* 1 hour ≈ 9,000 slots
* 1 day ≈ 216,000 slots

**Note**: Actual slot times may vary with network conditions.

### Transaction Limits

* Maximum participants per market: Limited by transaction size and account resizing
* Maximum concurrent bets: Limited by compute units
* Account data size: ≤ 10MB per account
