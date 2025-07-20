# Instructions

This document describes all instructions available in the Actions Prediction Market Protocol.

## Platform Management Instructions

### init\_platform

Initializes a new platform with its own configuration.

#### Parameters

* `platform_id: Pubkey` - Unique identifier for this platform
* `fee_percentage: u16` - Platform fee in basis points (max 10000 = 100%)
* `creator_fee_percentage: u16` - Creator fee in basis points (max 10000 = 100%)
* `treasury: Pubkey` - Where platform fees are sent

#### Accounts

* `global_config` - Global configuration (must be initialized)
* `platform_config` - Platform configuration account
* `authority` (signer) - Platform admin wallet
* `treasury` - Platform treasury wallet
* `system_program` - Solana System Program

#### Example

```typescript
const instructions = await generateInitPlatformInstructions({
  platformId: "ACTYY7k4vRAhzHw5gazNtEDdYEk1hC8751enx5K7Rwc",
  feePercentage: 100, // 1%
  creatorFeePercentage: 100, // 1%
  treasuryAddress: "ACT5vtRG11zd8trtR7ybNujkmbn1sUN5oHWsoTV2iszr",
  authorityAddress: platformAuthority.toString(),
});
```

***

### update\_platform

Updates platform configuration settings.

#### Parameters

* `platform_id: Pubkey` - Platform identifier
* `fee_percentage: Option<u16>` - New platform fee (optional)
* `creator_fee_percentage: Option<u16>` - New creator fee (optional)
* `treasury: Option<Pubkey>` - New treasury address (optional)

#### Accounts

* `global_config` - Global configuration
* `platform_config`  - Platform configuration account
* `authority` (signer) - Platform admin wallet

#### Access Control

* Only the platform authority can call this instruction
* At least one parameter must be provided

#### Example

```typescript
const instructions = await generateUpdatePlatformInstructions({
  platformId: "ACTYY7k4vRAhzHw5gazNtEDdYEk1hC8751enx5K7Rwc",
  feePercentage: 150, // Update to 1.5%
  authorityAddress: platformAuthority.toString(),
});
```

***

## Market Lifecycle Instructions

### create\_market

Creates a new prediction market.

#### Parameters

* `platform_id: Pubkey` - Platform this market belongs to
* `name: String` - Market title (max 100 characters)
* `description: String` - Market description (max 500 characters)
* `metadata_uri: String` - IPFS URI for metadata (max 200 characters)
* `expiry_slot: u64` - Solana slot when betting stops

#### Accounts

* `global_config` - Global configuration
* `platform_config` - Platform configuration
* `market` - Market account
* `market_vault`- Market vault for funds
* `participants_registry` - Participants tracking
* `creator_fee_account`  - Creator fee accumulator
* `creator` (signer) - Market creator wallet
* `system_program` - Solana System Program

#### Constraints

* Market name ≤ 100 characters
* Market description ≤ 500 characters
* Metadata URI ≤ 200 characters
* Expiry slot must be in the future
* Maximum duration: 24 hours (216,000 slots)

#### Example

```typescript
const instructions = await generateCreateMarketInstructions({
  platformId: "ACTYY7k4vRAhzHw5gazNtEDdYEk1hC8751enx5K7Rwc",
  marketName: "Will Bitcoin reach $100k by EOY?",
  marketDescription: "Prediction on whether Bitcoin will reach $100,000 by December 31st",
  metadataUri: "ipfs://QmHash123...",
  expirySlot: currentSlot + 216000, // 24 hours
  creatorAddress: creator.publicKey.toString(),
});
```

***

### make\_prediction

Places a bet on a prediction market.

#### Parameters

* `platform_id: Pubkey` - Platform identifier
* `option: bool` - Prediction choice (true = YES, false = NO)
* `amount: u64` - Bet amount in lamports

#### Accounts

* `global_config` - Global configuration
* `platform_config` - Platform configuration
* `market` - Market account
* `market_vault` - Market vault
* `creator_fee_account` - Creator fee accumulator
* `participants_registry` - Participants tracking
* `participant` (signer) - Bettor's wallet
* `treasury` - Platform treasury
* `global_treasury` - Protocol treasury
* `system_program` - Solana System Program

#### Fee Distribution

From each bet amount:

1. Global protocol fee → global\_treasury
2. Platform fee → treasury
3. Creator fee → creator\_fee\_account
4. Remaining amount → market\_vault

#### Constraints

* Amount must be > 0
* Market must be in Active state
* Cannot change bet option (must bet same option to add more)

#### Example

```typescript
const instructions = await generateMakePredictionInstructions({
  platformId: "ACTYY7k4vRAhzHw5gazNtEDdYEk1hC8751enx5K7Rwc",
  marketAddress: "BDq9PJhkgC3Zp2Y7xE1kN5oJ8mR6vT3wL9sA4fH2cG7uI",
  option: true, // YES
  amountLamports: "1000000000", // 1 SOL
  participantAddress: participant.publicKey.toString(),
});
```

***

### finish\_market

Resolves a market with the final outcome.

#### Parameters

* `platform_id: Pubkey` - Platform identifier
* `winning_option: Option<bool>` - Market outcome (None = canceled, Some(bool) = winner)

#### Accounts

* `global_config` - Global configuration
* `platform_config` - Platform configuration
* `market` - Market account
* `participants_registry`- Participants tracking
* `creator` (signer) - Market creator wallet
* `system_program` - Solana System Program

#### Access Control

* Only the market creator can call this instruction
* Market must be in Deciding state
* Must be called within 12 hours of market expiry

#### Market States After Resolution

* `Finalized` - Normal resolution with winners
* `FinalizedNoParticipants` - No participants or single-sided market
* `AutoCanceled` - If called after deadline

#### Example

```typescript
const instructions = await generateFinishMarketInstructions({
  platformId: "ACTYY7k4vRAhzHw5gazNtEDdYEk1hC8751enx5K7Rwc",
  marketAddress: "BDq9PJhkgC3Zp2Y7xE1kN5oJ8mR6vT3wL9sA4fH2cG7uI",
  winningOption: true, // YES wins
  creatorAddress: creator.publicKey.toString(),
});
```

***

### update\_market\_state

Automatically updates market state based on timing.

#### Parameters

* `platform_id: Pubkey` - Platform identifier

#### Accounts

* `global_config` - Global configuration
* `platform_config` - Platform configuration
* `market` - Market account
* `participants_registry` - Participants tracking
* `caller` (signer) - Platform authority wallet
* `system_program` - Solana System Program

#### Access Control

* Only platform authority can call this instruction

#### State Transitions

* Active → Deciding (when market expires)
* Deciding → AutoCanceled (when finalization deadline passes)

#### Example

```typescript
const instructions = await generateUpdateMarketStateInstructions({
  platformId: "ACTYY7k4vRAhzHw5gazNtEDdYEk1hC8751enx5K7Rwc",
  marketAddress: "BDq9PJhkgC3Zp2Y7xE1kN5oJ8mR6vT3wL9sA4fH2cG7uI",
  callerAddress: platformAuthority.toString(),
});
```

***

## Claiming Instructions

### claim\_winnings

Claims winnings from a finalized market.

#### Parameters

* `platform_id: Pubkey` - Platform identifier

#### Accounts

* `global_config` - Global configuration
* `platform_config` - Platform configuration
* `market`- Market account
* `market_vault` - Market vault
* `participants_registry` - Participants tracking
* `claim_account` - Claim tracking account
* `participant` (signer) - Winner's wallet
* `system_program` - Solana System Program

#### Constraints

* Market must be finalized
* Participant must be a winner
* Cannot claim twice (claim account prevents double-claiming)

#### Payout Calculation

Winner's payout = (participant\_bet / winning\_side\_total) × total\_market\_size

#### Example

```typescript
const instructions = await generateClaimWinningsInstructions({
  platformId: "ACTYY7k4vRAhzHw5gazNtEDdYEk1hC8751enx5K7Rwc",
  marketAddress: "BDq9PJhkgC3Zp2Y7xE1kN5oJ8mR6vT3wL9sA4fH2cG7uI",
  participantAddress: winner.publicKey.toString(),
});
```

***

### claim\_creator\_fees

Claims accumulated creator fees from a market.

#### Parameters

* `platform_id: Pubkey` - Platform identifier

#### Accounts

* `global_config` - Global configuration
* `platform_config` - Platform configuration
* `market` - Market account
* `creator_fee_account` - Creator fee accumulator
* `creator` (signer) - Market creator wallet
* `system_program` - Solana System Program

#### Access Control

* Only the market creator can call this instruction
* Fees must be available (accumulated\_fees > 0)

#### Example

```typescript
const instructions = await generateClaimCreatorFeesInstructions({
  platformId: "ACTYY7k4vRAhzHw5gazNtEDdYEk1hC8751enx5K7Rwc",
  marketAddress: "BDq9PJhkgC3Zp2Y7xE1kN5oJ8mR6vT3wL9sA4fH2cG7uI",
  creatorAddress: creator.publicKey.toString(),
});
```

***

###

***

## Instruction Summary

| Instruction           | Who Can Call       | Purpose                   |
| --------------------- | ------------------ | ------------------------- |
| `init_platform`       | Anyone             | Create new platform       |
| `update_platform`     | Platform Authority | Update platform settings  |
| `create_market`       | Anyone             | Create prediction market  |
| `make_prediction`     | Anyone             | Place bets                |
| `finish_market`       | Market Creator     | Resolve market outcome    |
| `update_market_state` | Platform Authority | Auto-update market states |
| `claim_winnings`      | Winners            | Claim prize money         |
| `claim_creator_fees`  | Market Creator     | Claim creator fees        |

## Fee Flow Summary

Each bet goes through this flow:

1. **Global Fee** (0.25%) → Protocol Treasury
2. **Platform Fee** (0.75%) → Platform Treasury
3. **Creator Fee** (1%) → Creator Fee Account
4. **Remaining** (98%) → Market Vault

When markets auto-cancel, platform authorities can reclaim creator fees that would otherwise be lost.
