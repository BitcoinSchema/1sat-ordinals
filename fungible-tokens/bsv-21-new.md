---
description: Extensible fungible token specification for Bitcoin SV
---

# BSV-21 Fungible Token Standard

## Overview

BSV-21 is a fungible token standard for Bitcoin SV that uses ordinal inscriptions to create, mint, and transfer tokens. Tokens are identified by their genesis transaction output (`<txid>_<vout>`) and exist as UTXOs on the Bitcoin SV blockchain.

The protocol supports two token models:
- **Fixed Supply**: Entire token supply created in a single deployment transaction
- **Auth Tokens**: Auth-based system allowing ongoing token creation

## Core Concepts

### Token Identification

Tokens are identified by the outpoint of their deployment transaction in the format `<txid>_<vout>`. This unique identifier remains constant throughout the token's lifecycle.

### UTXO Model

BSV-21 tokens exist in UTXOs, identical to native Bitcoin. This enables:
- Parallel transaction processing
- Natural splitting and combining of token amounts
- Standard Bitcoin script locking mechanisms
- Direct integration with Bitcoin's security model

### Content Type

All BSV-21 operations use the content type `application/bsv-20` for ordinal inscriptions.

### JSON Field Handling

BSV-21 inscriptions may contain additional JSON fields beyond those specified in this standard. Implementations must ignore unrecognized fields - only the fields defined in this specification affect token behavior.

## Fixed Supply Tokens

### Deploy+Mint Operation

Creates a token with a fixed, immutable supply. The entire token supply is minted in a single transaction output.

**Fields:**

| Field | Required | Description |
|-------|----------|-------------|
| `p` | Yes | Protocol identifier: `bsv-20` |
| `op` | Yes | Operation: `deploy+mint` |
| `amt` | Yes | Total token supply (max: 2^64-1) |
| `sym` | No | Token symbol (not enforced unique) |
| `icon` | No | Icon reference (inscription origin or B protocol file outpoint) |
| `dec` | No | Decimal precision (default: 0, max: 18) |

**Example:**

```json
{
  "p": "bsv-20",
  "op": "deploy+mint",
  "amt": "21000000",
  "sym": "GOLD",
  "dec": "8"
}
```

The token ID will be set to the outpoint where this inscription is created (e.g., `3b31...e000_0`).

## Auth Tokens

Auth tokens enable controlled, ongoing minting through auth UTXOs that grant minting capability.

### Deploy+Auth Operation

Creates a token with no initial supply and generates an authority UTXO that can be spent to mint new tokens.

**Fields:**

| Field | Required | Description |
|-------|----------|-------------|
| `p` | Yes | Protocol identifier: `bsv-20` |
| `op` | Yes | Operation: `deploy+auth` |
| `sym` | No | Token symbol (not enforced unique) |
| `icon` | No | Icon reference (inscription origin or B protocol file outpoint) |
| `dec` | No | Decimal precision (default: 0, max: 18) |
| `amt` | No | **Must not be present** - auth outputs carry authority, not value |

**Example:**

```json
{
  "p": "bsv-20",
  "op": "deploy+auth",
  "sym": "STABLE",
  "dec": "2"
}
```

### Mint Operation

Creates new token supply by spending an auth UTXO. Any number of mint outputs can be created from a single auth input.

**Fields:**

| Field | Required | Description |
|-------|----------|-------------|
| `p` | Yes | Protocol identifier: `bsv-20` |
| `op` | Yes | Operation: `mint` |
| `id` | Yes | Token ID (`<txid>_<vout>` of deploy+auth output) |
| `amt` | Yes | Amount of tokens to mint |

**Example:**

```json
{
  "p": "bsv-20",
  "op": "mint",
  "id": "3b31...e000_0",
  "amt": "1000000"
}
```

**Transaction Structure:**
```
Inputs:
  - Auth UTXO (from deploy+auth or previous auth output)

Outputs:
  - Mint inscription (creates 1,000,000 new tokens)
  - Auth inscription (continues minting capability)
```

### Auth Operation

Manages auth UTXOs by creating new auth outputs. Auth can be split, combined, or transferred to delegate minting authority.

**Fields:**

| Field | Required | Description |
|-------|----------|-------------|
| `p` | Yes | Protocol identifier: `bsv-20` |
| `op` | Yes | Operation: `auth` |
| `id` | Yes | Token ID (`<txid>_<vout>` of deploy+auth output) |
| `amt` | No | **Must not be present** - auth outputs carry authority, not value |

**Example:**

```json
{
  "p": "bsv-20",
  "op": "auth",
  "id": "3b31...e000_0"
}
```

**Auth Capabilities:**
- **Split**: One auth input → multiple auth outputs (delegate authority)
- **Combine**: Multiple auth inputs → one auth output (consolidate authority)
- **Transfer**: Spend auth to new locking script (transfer authority)
- **Burn**: Spend auth without creating new auth output (permanently end minting)

## Token Transfers

Tokens are transferred by spending token UTXOs and creating new token outputs, identical to spending native Bitcoin.

### Transfer Operation

**Fields:**

| Field | Required | Description |
|-------|----------|-------------|
| `p` | Yes | Protocol identifier: `bsv-20` |
| `op` | Yes | Operation: `transfer` |
| `id` | Yes | Token ID (`<txid>_<vout>` of deployment output) |
| `amt` | Yes | Amount of tokens in this output |

**Example:**

```json
{
  "p": "bsv-20",
  "op": "transfer",
  "id": "3b31...e000_0",
  "amt": "5000"
}
```

### Transfer Validation Rules

**Token Conservation:**
- Total output tokens ≤ Total input tokens
- If outputs exceed inputs: transaction invalid, tokens burned
- If outputs < inputs: excess tokens burned

**Example Transaction:**

```
Inputs:
  - Transfer UTXO: 1,000 tokens
  - Transfer UTXO: 500 tokens

Outputs:
  - Transfer: 800 tokens (to recipient A)
  - Transfer: 600 tokens (to recipient B)
  - Transfer: 100 tokens (change to sender)

Total In: 1,500 tokens
Total Out: 1,500 tokens
Status: Valid
```

**Invalid Example:**

```
Inputs:
  - Transfer UTXO: 500 tokens

Outputs:
  - Transfer: 300 tokens
  - Transfer: 400 tokens

Total In: 500 tokens
Total Out: 700 tokens
Status: Invalid - All tokens burned
```

## Validation Rules

### Operation-Specific Rules

**Deploy Operations (deploy+mint, deploy+auth):**
- Automatically valid - no input validation required
- Token ID is set to the deployment output's outpoint
- Creates token genesis

**Mint Operations:**
- Requires at least one auth input to be valid
- Minted tokens are created, not transferred from inputs
- Any number of mint outputs can be created from a single auth input

**Auth Operations:**
- Requires valid auth input spending
- Auth inputs do not contribute to token balance
- Can create multiple auth outputs from single auth input

**Transfer Operations:**
- Requires token conservation: `total_input_tokens ≥ total_output_tokens`
- Auth inputs do not affect transfer validation
- Presence of auth does not bypass balance requirements

### Field Validation

**Amount Field (`amt`):**
- Required: `deploy+mint`, `mint`, `transfer`
- Prohibited: `deploy+auth`, `auth`
- Format: String representation of uint64 (max: 18,446,744,073,709,551,615)

**Token ID Field (`id`):**
- Required: `mint`, `auth`, `transfer`
- Format: `<txid>_<vout>` where txid is 64 hex characters
- Must reference valid deployment output
- Auto-set for deploy operations

**Decimals Field (`dec`):**
- Optional: `deploy+mint`, `deploy+auth`
- Range: 0-18
- Default: 0
- Determines token divisibility

## Locking Scripts

BSV-21 tokens support any valid Bitcoin locking script, including P2PKH, multisig, custom smart contracts, and complex spending conditions. Tokens can be locked using the same mechanisms available to native Bitcoin satoshis.

## Token Metadata

Token metadata (`sym`, `icon`, `dec`) is set during deployment and inherited by all subsequent operations.

The `icon` field references an image by its outpoint in `<txid>_<vout>` format, pointing to either an inscription containing an image or a B protocol file upload.

**Deployment:** Sets metadata
```json
{
  "p": "bsv-20",
  "op": "deploy+mint",
  "amt": "1000000",
  "sym": "GOLD",
  "dec": "8",
  "icon": "abc123...def456_0"
}
```

**Transfer:** Metadata auto-populated from deployment
```json
{
  "p": "bsv-20",
  "op": "transfer",
  "id": "3b31...e000_0",
  "amt": "100"
}
```
The transfer inherits `sym: "GOLD"`, `dec: 8`, `icon: "abc..."` from the deployment.

## Protocol Identifier

All BSV-21 operations use `"p": "bsv-20"` as the protocol identifier for backward compatibility with existing infrastructure.

## Transaction Examples

### Complete Fixed Supply Token Lifecycle

**1. Deploy Token**
```json
{
  "p": "bsv-20",
  "op": "deploy+mint",
  "amt": "10000",
  "sym": "FIXED",
  "dec": "2"
}
```
Creates token `abc...123_0` with 10,000 tokens (100.00 with 2 decimals)

**2. Split Tokens**
```
Input: Deploy output (10,000 tokens)
Outputs:
  - Transfer: 5,000 tokens
  - Transfer: 5,000 tokens
```

**3. Transfer to User**
```
Input: Transfer UTXO (5,000 tokens)
Outputs:
  - Transfer: 4,900 tokens (to user)
  - Transfer: 100 tokens (change)
```

### Complete Auth Token Lifecycle

**1. Deploy with Auth**
```json
{
  "p": "bsv-20",
  "op": "deploy+auth",
  "sym": "STABLE",
  "dec": "6"
}
```
Creates token `def...456_0` with auth capability

**2. Initial Mint**
```
Input: Auth UTXO (from deploy+auth)
Outputs:
  - Mint: 1,000,000 tokens
  - Auth: Continue minting capability
```

**3. Distribute Minted Tokens**
```
Input: Mint output (1,000,000 tokens)
Outputs:
  - Transfer: 500,000 tokens (to user A)
  - Transfer: 500,000 tokens (to user B)
```

**4. Additional Mint**
```
Input: Auth UTXO (from previous auth output)
Outputs:
  - Mint: 500,000 tokens
  - Auth: Continue minting capability
```

**5. Delegate Minting Authority**
```
Input: Auth UTXO
Outputs:
  - Auth: Locked to admin A
  - Auth: Locked to admin B
```

**6. Burn Auth (End Minting)**
```
Input: Auth UTXO
Outputs:
  - (No auth outputs created)
```
Minting permanently disabled for this token.

## Summary

BSV-21 provides two complementary token models on Bitcoin SV:

**Fixed Supply Tokens** offer simplicity and immutability - the entire supply is created at deployment and cannot be changed.

**Auth Tokens** offer flexibility - auth UTXOs enable controlled, ongoing minting with delegatable authority.

Both models leverage Bitcoin's UTXO architecture for parallel processing, standard script locking, and native security guarantees.
