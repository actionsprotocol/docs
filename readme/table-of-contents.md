# Table of Contents

## Introduction

* Overview

## Core Documentation

* Account Types
  * GlobalConfig
  * PlatformConfig
  * SimpleMarket
  * MarketVault
  * ParticipantsRegistry
  * CreatorFeeAccount
  * ClaimAccount
  * MarketState Enum
* Instructions
  * Platform Management
    * init\_platform
    * update\_platform
  * Market Lifecycle
    * create\_market
    * make\_prediction
    * finish\_market
    * update\_market\_state
  * Claiming
    * claim\_winnings
    * claim\_creator\_fees
* PDA Derivations
  * Global Level
  * Platform Level
  * Market Level
  * Participant Level
* Error Codes
  * Error Categories
  * Complete Error List
  * Debugging Tips
* Constants & Configuration
  * Global Constants
  * Program ID
  * Enums
  * Account Sizes
  * Default Values
  * Validation Rules

## Quick Reference

### Key Concepts

* **Multi-Platform Architecture**: Protocol supports multiple platforms with individual configurations
* **Creator Fee Governance**: Platform authorities can reclaim fees from irresponsible creators
* **Three-Tier Fees**: Global (0.25%) + Platform (1%) + Creator (1%) = \~2.25% total
* **Market Lifecycle**: Active → Deciding → Finalized/AutoCanceled
* **PDA-Based Accounts**: All accounts use deterministic Program Derived Addresses

### Program ID

```
ACTUdJVh7H389kKpgxKjhR6o2JhRrTPdB9dS6cy41XzX
```

### Key Constants

* **Maximum Market Duration**: 24 hours (216,000 slots)
* **Finalization Window**: 12 hours (108,000 slots)
* **Maximum Global Fee**: 5% (500 basis points)
* **Market Name Limit**: 100 characters
* **Market Description Limit**: 500 characters

### State Flow

```
[Active] → [Deciding] → [Finalized]
            ↓
        [AutoCanceled]
```

### Fee Distribution

```
Each Bet (100%)
├── Global Fee (0.25%) → Protocol Treasury
├── Platform Fee (0.75%) → Platform Treasury
├── Creator Fee (1%) → Creator Fee Account
└── Market Pool (98%) → Market Vault
```
