# Actions Prediction Market Protocol

A decentralized prediction market protocol built on Solana, featuring multi-platform governance and creator fee management.

## Overview

The Actions Prediction Market Protocol enables the creation and management of prediction markets with a sophisticated fee structure and governance model. The protocol supports multiple platforms, each with their own configuration, while maintaining global protocol standards.

## Key Features

* **Multi-Platform Architecture**: Support for multiple platforms with individual fee structures
* **Creator Fee Governance**: Platform authorities can reclaim fees from irresponsible creators
* **Automated Market Lifecycle**: Markets automatically transition between states based on timing
* **Flexible Betting System**: Support for YES/NO predictions with anti-manipulation features
* **Fee Distribution**: Three-tier fee structure (Protocol, Platform, Creator)

## Program ID

**Devnet**: `ACTUdJVh7H389kKpgxKjhR6o2JhRrTPdB9dS6cy41XzX`\
**Mainnet**: `ACTUdJVh7H389kKpgxKjhR6o2JhRrTPdB9dS6cy41XzX`

## Architecture

The protocol uses a hierarchical structure:

```
Global Config (Protocol Level)
├── Platform Config A
│   ├── Market 1
│   ├── Market 2
│   └── ...
├── Platform Config B
│   ├── Market 3
│   ├── Market 4
│   └── ...
└── ...
```

## Fee Structure

Each bet is subject to three fees:

* **Global Protocol Fee**: 0.25% (goes to protocol treasury)
* **Platform Fee**: Configurable per platform (typically 0.75%)
* **Creator Fee**: Configurable per platform (typically 1%)

## Market Lifecycle

1. **Active**: Market accepts bets
2. **Deciding**: Market expired, waiting for creator resolution
3. **Finalized**: Market resolved with winning option
4. **AutoCanceled**: Creator failed to resolve within deadline

## Quick Start

```typescript
import { generateCreateMarketInstructions } from "./tx";

// Create a new prediction market
const instructions = await generateCreateMarketInstructions({
  platformId: "ACTYY7k4vRAhzHw5gazNtEDdYEk1hC8751enx5K7Rwc",
  marketName: "Will Bitcoin reach $100k by EOY?",
  marketDescription: "Prediction on Bitcoin price target",
  metadataUri: "ipfs://...",
  expirySlot: currentSlot + 216000, // 24 hours
  creatorAddress: creatorPublicKey.toString(),
});
```

## Documentation Structure

* Account Types - All program state structures
* Instructions - All program instructions
* PDA Derivations - Account address generation
* Error Codes - Custom error types
* Constants - Program constants and enums
