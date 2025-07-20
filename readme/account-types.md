# Account Types

This document describes all account types (state structures) used in the Actions Prediction Market Protocol.

## GlobalConfig

The protocol-level configuration that governs all platforms.

```rust
pub struct GlobalConfig {
    pub protocol_authority: Pubkey,     // Protocol admin
    pub global_treasury: Pubkey,        // Protocol fee destination
    pub global_fee_percentage: u16,     // Basis points (25 = 0.25%)
    pub initialized: bool,              // Init status
    pub bump: u8,                       // PDA bump seed
}
```

**Size**: 76 bytes\
**PDA Seeds**: `["global_config"]`

### Fields

* `protocol_authority`: The wallet authorized to update global settings
* `global_treasury`: Where global protocol fees are sent
* `global_fee_percentage`: Fee percentage in basis points (max 500 = 5%)
* `initialized`: Whether the global config has been set up
* `bump`: The bump seed used for PDA derivation

***

## PlatformConfig

Platform-specific configuration for individual prediction market platforms.

```rust
pub struct PlatformConfig {
    pub authority: Pubkey,              // Platform admin
    pub fee_percentage: u16,            // Platform fee in basis points
    pub creator_fee_percentage: u16,    // Creator fee in basis points
    pub treasury: Pubkey,               // Platform fee destination
    pub initialized: bool,              // Init status
    pub bump: u8,                       // PDA bump seed
}
```

**Size**: 77 bytes\
**PDA Seeds**: `["platform_config", global_config_key, platform_id]`

### Fields

* `authority`: The wallet authorized to manage this platform
* `fee_percentage`: Platform fee in basis points (max 10000 = 100%)
* `creator_fee_percentage`: Creator fee in basis points (max 10000 = 100%)
* `treasury`: Where platform fees are sent
* `initialized`: Whether the platform has been set up
* `bump`: The bump seed used for PDA derivation

***

## SimpleMarket

The core market structure containing all market information.

```rust
pub struct SimpleMarket {
    pub name: String,                   // Market title (max 100 chars)
    pub description: String,            // Market description (max 500 chars)
    pub metadata_uri: String,           // IPFS metadata URI (max 200 chars)
    pub creator: Pubkey,                // Market creator
    pub expiry_slot: u64,               // When market stops accepting bets
    pub finalization_deadline: u64,     // When market auto-cancels
    pub state: MarketState,             // Current market state
    pub total_market_size: u64,         // Total amount bet (lamports)
    pub yes_amount: u64,                // Amount bet on YES (lamports)
    pub no_amount: u64,                 // Amount bet on NO (lamports)
    pub winning_option: Option<bool>,   // None=canceled, Some(bool)=winner
    pub participants_registry: Pubkey,  // Link to participants account
    pub created_slot: u64,              // When market was created
    pub finalization_slot: Option<u64>, // When market was finalized
    pub bump: u8,                       // PDA bump seed
}
```

**Size**: 816 bytes\
**PDA Seeds**: `["market", creator_key, expiry_slot_bytes]`

### Fields

* `name`: Human-readable market title
* `description`: Detailed market description
* `metadata_uri`: IPFS URI for additional metadata (images, etc.)
* `creator`: The wallet that created this market
* `expiry_slot`: Solana slot when betting closes
* `finalization_deadline`: expiry\_slot + 12 hours for creator to resolve
* `state`: Current market state (Active, Deciding, Finalized, etc.)
* `total_market_size`: Sum of all bets placed (after fees)
* `yes_amount`: Total amount bet on YES option
* `no_amount`: Total amount bet on NO option
* `winning_option`: Market outcome (None for canceled markets)
* `participants_registry`: Address of associated participants account
* `created_slot`: When the market was created
* `finalization_slot`: When the market was resolved (if applicable)
* `bump`: The bump seed used for PDA derivation

***

## MarketVault

Holds the betting pool funds for a market.

```rust
pub struct MarketVault {
    pub market: Pubkey,                 // Associated market
    pub bump: u8,                       // PDA bump seed
}
```

**Size**: 41 bytes\
**PDA Seeds**: `["market_vault", market_key]`

### Fields

* `market`: The market this vault belongs to
* `bump`: The bump seed used for PDA derivation

***

## ParticipantsRegistry

Tracks all participants and their bets for a market.

```rust
pub struct ParticipantsRegistry {
    pub market: Pubkey,                 // Associated market
    pub participants: Vec<Participant>, // List of all participants
    pub total_participants: u32,        // Count of participants
    pub bump: u8,                       // PDA bump seed
}
```

**Size**: Variable (Base: 49 bytes + participants)\
**PDA Seeds**: `["participants", market_key]`

### Fields

* `market`: The market this registry belongs to
* `participants`: Vector containing all participant data
* `total_participants`: Total number of unique participants
* `bump`: The bump seed used for PDA derivation

### Participant Structure

```rust
pub struct Participant {
    pub pubkey: Pubkey,                 // Participant's wallet
    pub amount: u64,                    // Amount bet (lamports, after fees)
    pub option: bool,                   // true = YES, false = NO
    pub timestamp: i64,                 // When bet was placed
}
```

**Size**: 49 bytes per participant

***

## CreatorFeeAccount

Accumulates creator fees for a market.

```rust
pub struct CreatorFeeAccount {
    pub market: Pubkey,                 // Associated market
    pub creator: Pubkey,                // Market creator
    pub accumulated_fees: u64,          // Total fees earned (lamports)
    pub bump: u8,                       // PDA bump seed
}
```

**Size**: 73 bytes\
**PDA Seeds**: `["creator_fees", platform_id, market_key]`

### Fields

* `market`: The market this fee account belongs to
* `creator`: The market creator who can claim these fees
* `accumulated_fees`: Total creator fees accumulated in lamports
* `bump`: The bump seed used for PDA derivation

> **Note**: Platform authorities can reclaim fees from auto-canceled markets using the `reclaim_creator_fees` instruction.

***

## ClaimAccount

Tracks winnings claims to prevent double-claiming.

```rust
pub struct ClaimAccount {
    pub market: Pubkey,                 // Associated market
    pub participant: Pubkey,            // Participant who claimed
    pub amount_to_claim: u64,           // Amount claimed (lamports)
    pub claimed: bool,                  // Whether claim was processed
    pub bump: u8,                       // PDA bump seed
}
```

**Size**: 74 bytes\
**PDA Seeds**: `["claim", market_key, participant_key]`

### Fields

* `market`: The market this claim belongs to
* `participant`: The participant who made the claim
* `amount_to_claim`: The amount that was claimed
* `claimed`: Whether the claim has been processed (always true after creation)
* `bump`: The bump seed used for PDA derivation

***

## MarketState Enum

Represents the current state of a market.

```rust
pub enum MarketState {
    Active,                             // Accepting bets
    Deciding,                           // Expired, awaiting resolution
    Finalized,                          // Resolved with outcome
    FinalizedNoParticipants,            // Resolved but no/invalid participants
    AutoCanceled,                       // Auto-canceled due to timeout
}
```

### State Transitions

```
Active → Deciding (when market expires)
Deciding → Finalized (when creator resolves)
Deciding → AutoCanceled (when deadline passes)
Active → FinalizedNoParticipants (special cases)
```

### State Descriptions

* **Active**: Market is open for betting
* **Deciding**: Market has expired, creator has time to resolve
* **Finalized**: Market resolved with winning option determined
* **FinalizedNoParticipants**: Market ended with no valid participants
* **AutoCanceled**: Creator failed to resolve within deadline

***

## Size Calculations

| Account Type         | Base Size | Variable Part             |
| -------------------- | --------- | ------------------------- |
| GlobalConfig         | 76 bytes  | Fixed                     |
| PlatformConfig       | 77 bytes  | Fixed                     |
| SimpleMarket         | 816 bytes | Fixed                     |
| MarketVault          | 41 bytes  | Fixed                     |
| ParticipantsRegistry | 49 bytes  | +49 bytes per participant |
| CreatorFeeAccount    | 73 bytes  | Fixed                     |
| ClaimAccount         | 74 bytes  | Fixed                     |

All sizes include the 8-byte Anchor discriminator.
