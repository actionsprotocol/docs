# PDA Derivations

This document explains how to derive Program Derived Addresses (PDAs) for all account types in the Actions Prediction Market Protocol.

## Global Level

### GlobalConfig

The protocol-level configuration account.

**Seeds**: `["global_config"]`

```typescript
const [globalConfigPda, globalConfigBump] = PublicKey.findProgramAddressSync(
  [Buffer.from("global_config")],
  programId
);
```

**JavaScript Helper**:

```typescript
export function getGlobalConfigPda(programId: PublicKey): [PublicKey, number] {
  return PublicKey.findProgramAddressSync(
    [Buffer.from("global_config")],
    programId
  );
}
```

***

## Platform Level

### PlatformConfig

Platform-specific configuration.

**Seeds**: `["platform_config", global_config_key, platform_id]`

```typescript
const [platformConfigPda, platformConfigBump] = PublicKey.findProgramAddressSync(
  [
    Buffer.from("platform_config"),
    globalConfigPda.toBuffer(),
    platformId.toBuffer()
  ],
  programId
);
```

**JavaScript Helper**:

```typescript
export function getPlatformConfigPda(
  programId: PublicKey,
  globalConfigPda: PublicKey,
  platformId: PublicKey
): [PublicKey, number] {
  return PublicKey.findProgramAddressSync(
    [
      Buffer.from("platform_config"),
      globalConfigPda.toBuffer(),
      platformId.toBuffer()
    ],
    programId
  );
}
```

***

## Market Level

### SimpleMarket

The main market account.

**Seeds**: `["market", creator_key, expiry_slot_bytes]`

```typescript
const expirySlotBytes = Buffer.alloc(8);
expirySlotBytes.writeBigUInt64LE(BigInt(expirySlot));

const [marketPda, marketBump] = PublicKey.findProgramAddressSync(
  [
    Buffer.from("market"),
    creatorPublicKey.toBuffer(),
    expirySlotBytes
  ],
  programId
);
```

**JavaScript Helper**:

```typescript
export function getMarketPda(
  programId: PublicKey,
  creator: PublicKey,
  expirySlot: number
): [PublicKey, number] {
  const expirySlotBytes = Buffer.alloc(8);
  expirySlotBytes.writeBigUInt64LE(BigInt(expirySlot));
  
  return PublicKey.findProgramAddressSync(
    [
      Buffer.from("market"),
      creator.toBuffer(),
      expirySlotBytes
    ],
    programId
  );
}
```

### MarketVault

Holds the betting pool funds.

**Seeds**: `["market_vault", market_key]`

```typescript
const [marketVaultPda, marketVaultBump] = PublicKey.findProgramAddressSync(
  [
    Buffer.from("market_vault"),
    marketPda.toBuffer()
  ],
  programId
);
```

**JavaScript Helper**:

```typescript
export function getMarketVaultPda(
  programId: PublicKey,
  marketPda: PublicKey
): [PublicKey, number] {
  return PublicKey.findProgramAddressSync(
    [
      Buffer.from("market_vault"),
      marketPda.toBuffer()
    ],
    programId
  );
}
```

### ParticipantsRegistry

Tracks all participants and their bets.

**Seeds**: `["participants", market_key]`

```typescript
const [participantsRegistryPda, participantsRegistryBump] = PublicKey.findProgramAddressSync(
  [
    Buffer.from("participants"),
    marketPda.toBuffer()
  ],
  programId
);
```

**JavaScript Helper**:

```typescript
export function getParticipantsRegistryPda(
  programId: PublicKey,
  marketPda: PublicKey
): [PublicKey, number] {
  return PublicKey.findProgramAddressSync(
    [
      Buffer.from("participants"),
      marketPda.toBuffer()
    ],
    programId
  );
}
```

### CreatorFeeAccount

Accumulates creator fees (now includes platform\_id for governance).

**Seeds**: `["creator_fees", platform_id, market_key]`

```typescript
const [creatorFeeAccountPda, creatorFeeAccountBump] = PublicKey.findProgramAddressSync(
  [
    Buffer.from("creator_fees"),
    platformId.toBuffer(),
    marketPda.toBuffer()
  ],
  programId
);
```

**JavaScript Helper**:

```typescript
export function getCreatorFeeAccountPda(
  programId: PublicKey,
  platformId: PublicKey,
  marketPda: PublicKey
): [PublicKey, number] {
  return PublicKey.findProgramAddressSync(
    [
      Buffer.from("creator_fees"),
      platformId.toBuffer(),
      marketPda.toBuffer()
    ],
    programId
  );
}
```

> **Important**: The creator fee account now includes `platform_id` in its derivation to enable platform authorities to reclaim fees from auto-canceled markets.

***

## Participant Level

### ClaimAccount

Tracks winnings claims to prevent double-claiming.

**Seeds**: `["claim", market_key, participant_key]`

```typescript
const [claimAccountPda, claimAccountBump] = PublicKey.findProgramAddressSync(
  [
    Buffer.from("claim"),
    marketPda.toBuffer(),
    participantPublicKey.toBuffer()
  ],
  programId
);
```

**JavaScript Helper**:

```typescript
export function getClaimAccountPda(
  programId: PublicKey,
  marketPda: PublicKey,
  participant: PublicKey
): [PublicKey, number] {
  return PublicKey.findProgramAddressSync(
    [
      Buffer.from("claim"),
      marketPda.toBuffer(),
      participant.toBuffer()
    ],
    programId
  );
}
```

***

## Complete Market Creation Example

Here's how to derive all PDAs needed to create a market:

```typescript
import { PublicKey } from "@solana/web3.js";

export function deriveAllMarketPdas(
  programId: PublicKey,
  platformId: PublicKey,
  creator: PublicKey,
  expirySlot: number
) {
  // Global config (singleton)
  const [globalConfigPda] = getGlobalConfigPda(programId);
  
  // Platform config
  const [platformConfigPda] = getPlatformConfigPda(
    programId,
    globalConfigPda,
    platformId
  );
  
  // Market accounts
  const [marketPda] = getMarketPda(programId, creator, expirySlot);
  const [marketVaultPda] = getMarketVaultPda(programId, marketPda);
  const [participantsRegistryPda] = getParticipantsRegistryPda(programId, marketPda);
  const [creatorFeeAccountPda] = getCreatorFeeAccountPda(
    programId,
    platformId,
    marketPda
  );
  
  return {
    globalConfigPda,
    platformConfigPda,
    marketPda,
    marketVaultPda,
    participantsRegistryPda,
    creatorFeeAccountPda,
  };
}
```

## Complete Betting Example

PDAs needed for placing a bet:

```typescript
export function deriveBettingPdas(
  programId: PublicKey,
  platformId: PublicKey,
  marketPda: PublicKey
) {
  // Global config
  const [globalConfigPda] = getGlobalConfigPda(programId);
  
  // Platform config
  const [platformConfigPda] = getPlatformConfigPda(
    programId,
    globalConfigPda,
    platformId
  );
  
  // Market accounts
  const [marketVaultPda] = getMarketVaultPda(programId, marketPda);
  const [participantsRegistryPda] = getParticipantsRegistryPda(programId, marketPda);
  const [creatorFeeAccountPda] = getCreatorFeeAccountPda(
    programId,
    platformId,
    marketPda
  );
  
  return {
    globalConfigPda,
    platformConfigPda,
    marketVaultPda,
    participantsRegistryPda,
    creatorFeeAccountPda,
  };
}
```

## Complete Claiming Example

PDAs needed for claiming winnings:

```typescript
export function deriveClaimingPdas(
  programId: PublicKey,
  platformId: PublicKey,
  marketPda: PublicKey,
  participant: PublicKey
) {
  // Global config
  const [globalConfigPda] = getGlobalConfigPda(programId);
  
  // Platform config
  const [platformConfigPda] = getPlatformConfigPda(
    programId,
    globalConfigPda,
    platformId
  );
  
  // Market accounts
  const [marketVaultPda] = getMarketVaultPda(programId, marketPda);
  const [participantsRegistryPda] = getParticipantsRegistryPda(programId, marketPda);
  const [claimAccountPda] = getClaimAccountPda(programId, marketPda, participant);
  
  return {
    globalConfigPda,
    platformConfigPda,
    marketVaultPda,
    participantsRegistryPda,
    claimAccountPda,
  };
}
```

***

## PDA Summary Table

| Account Type         | Seeds                                             | Scope       | Uniqueness             |
| -------------------- | ------------------------------------------------- | ----------- | ---------------------- |
| GlobalConfig         | `["global_config"]`                               | Protocol    | Singleton              |
| PlatformConfig       | `["platform_config", global_config, platform_id]` | Platform    | Per platform           |
| SimpleMarket         | `["market", creator, expiry_slot]`                | Market      | Per creator+time       |
| MarketVault          | `["market_vault", market]`                        | Market      | Per market             |
| ParticipantsRegistry | `["participants", market]`                        | Market      | Per market             |
| CreatorFeeAccount    | `["creator_fees", platform_id, market]`           | Market      | Per platform+market    |
| ClaimAccount         | `["claim", market, participant]`                  | Participant | Per market+participant |

## Key Changes

### CreatorFeeAccount PDA Update

The most important change is in the `CreatorFeeAccount` PDA derivation:

**Old** (pre-governance):

```typescript
["creator_fees", market_key]
```

**New** (with governance):

```typescript
["creator_fees", platform_id, market_key]
```

This change enables platform authorities to reclaim creator fees from auto-canceled markets while maintaining separation between platforms.
