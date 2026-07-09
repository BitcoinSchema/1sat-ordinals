---
description: Binary fungible token encoding following the BSV-21 rules
---

# ¯\\_(ツ)\_/¯ (Shrug)

## Overview

Shrug is a binary encoding of the [BSV-21](bsv-21.md) fungible token rules. It is not a competing standard — the token model, identification scheme, and validation rules are the same as BSV-21. Where BSV-21 encodes token data as a JSON inscription, shrug encodes it as raw data pushes at the front of the locking script.

This makes shrug outputs significantly easier to work with from Bitcoin script. The token id and amount sit at fixed positions in a compact binary prefix, so contracts and covenants can construct and inspect token outputs directly — no JSON parsing, no inscription envelope.

## Wire Format

A shrug output is an ordinary locking script with a five-element prefix:

```
<push "¯\_(ツ)_/¯"> <push token id | OP_0> OP_2DROP <push amount | OP_0> OP_DROP <owner locking script>
```

| Element | Encoding |
|---|---|
| Tag | Push of the 13-byte UTF-8 string `¯\_(ツ)_/¯` (hex `c2af5c5f28e38384295f2fc2af`) |
| Token id | Push of the 36-byte outpoint (32-byte txid + 4-byte little-endian vout), or `OP_0` for deploys |
| `OP_2DROP` | Drops tag and id from the stack |
| Amount | Push of a minimally-encoded script number, or `OP_0` for zero |
| `OP_DROP` | Drops the amount from the stack |
| Owner script | Any valid Bitcoin locking script (P2PKH, multisig, custom contracts) |

The prefix is stack-neutral: it pushes three values and drops all three, so the owner script that follows executes exactly as it would alone.

## Operations

Shrug has no operation field. The operation is implied by which fields are present:

| Token id | Amount | BSV-21 equivalent | Meaning |
|---|---|---|---|
| Absent (`OP_0`) | > 0 | `deploy+mint` | Genesis with fixed supply in this output |
| Absent (`OP_0`) | 0 | `deploy+auth` | Genesis; this output is the initial minting authority |
| Present | 0 | `auth` | Minting authority |
| Present | > 0 | `transfer` / `mint` | Token value (see Validation Rules) |

As in BSV-21, a deploy output self-identifies: the token id is the outpoint of the deploy output itself. All subsequent outputs reference it in binary form.

There is no explicit `burn` operation. Tokens are burned implicitly by spending value inputs without creating matching value outputs.

## Amounts

Amounts are Bitcoin script numbers, minimally encoded, in the range 0 to 2^64-1 — the same domain as BSV-21. An output whose amount push is non-minimal, negative, or wider than 64 bits is not a shrug output. An amount of 0 is not a token value; it marks the output as a minting authority.

## Metadata

Shrug carries no metadata — no symbol, icon, or decimals. Indexers may present shrug amounts with 0 decimal precision.

## Validation Rules

Validation follows the BSV-21 model:

**Deploy outputs** (no id):
- Automatically valid — no input validation required
- Token id is set to the deployment output's outpoint

**Authority outputs** (id, amount 0):
- Admitted only when the transaction spends a valid authority output of the same token
- A deploy output with amount 0 is the token's genesis authority
- Authority can be split (one authority input → many authority outputs), combined, transferred, or burned (spent without creating a new authority output)
- Authority inputs contribute nothing to token balance

**Value outputs** (id, amount > 0):
- If the transaction spends a valid authority input for the token: value outputs are admitted without balance coverage. This is minting — authority holders create new supply.
- Otherwise: token conservation applies. Value outputs are admitted only when valid value inputs cover the total output amount, all-or-nothing per token.
- If outputs exceed inputs without authority present, the outputs are invalid and the input tokens are burned
- If inputs exceed outputs, the excess is burned

Because minting and transferring share one encoding, per-output labels do not exist. Circulating supply is computed as the net value delta of authority-bearing transactions, minus implicit burns.

## Comparison with BSV-21

| | BSV-21 | Shrug |
|---|---|---|
| Encoding | JSON inscription (`application/bsv-20`) | Binary script prefix |
| Token id | `<txid>_<vout>` string | 36-byte binary outpoint |
| Operations | Explicit `op` field (6 ops) | Implicit from field presence |
| Explicit burn | Yes | No (implicit only) |
| Metadata (`sym`, `icon`, `dec`) | Optional at deploy | None |
| Amount | String uint64 in JSON | Script number, uint64 range |
| Script access to token data | Requires envelope/JSON parsing | Fixed-position pushes |
| Validation model | Auth-gated minting + conservation | Same |

## Examples

Deploy a token with a fixed supply of 21,000,000, owned by a P2PKH address:

```
"¯\_(ツ)_/¯" OP_0 OP_2DROP 21000000 OP_DROP
OP_DUP OP_HASH160 <pubkeyhash> OP_EQUALVERIFY OP_CHECKSIG
```

Deploy an authority-based token (no initial supply):

```
"¯\_(ツ)_/¯" OP_0 OP_2DROP OP_0 OP_DROP
OP_DUP OP_HASH160 <pubkeyhash> OP_EQUALVERIFY OP_CHECKSIG
```

Mint 1,000,000 tokens (transaction spends the authority output above and recreates it):

```
Input:  genesis authority outpoint

Output 0 (value):     "¯\_(ツ)_/¯" <36-byte token id> OP_2DROP 1000000 OP_DROP <owner script>
Output 1 (authority): "¯\_(ツ)_/¯" <36-byte token id> OP_2DROP OP_0 OP_DROP <owner script>
```

Transfer 400 of 1,000 held tokens (no authority input — conservation applies):

```
Inputs: value outpoint holding 1,000 tokens

Output 0 (value): "¯\_(ツ)_/¯" <36-byte token id> OP_2DROP 400 OP_DROP <recipient script>
Output 1 (value): "¯\_(ツ)_/¯" <36-byte token id> OP_2DROP 600 OP_DROP <change script>
```
