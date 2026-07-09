---
description: Binary fungible token encoding following the BSV-21 rules
---

# ¯\\_(ツ)\_/¯ (Shrug)

## Overview

Shrug is a binary evolution of the [BSV-21](bsv-21.md) fungible token standard. It carries forward the BSV-21 token model — outpoint token ids, UTXO-held balances, authority-gated minting, conservation-validated transfers — while replacing the JSON inscription with raw data pushes at the front of the locking script. The capabilities are not identical: shrug trades in-protocol metadata and explicit operation labels for a minimal binary format that Bitcoin script can work with directly (see the comparison table below).

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

As in BSV-21, a deploy output self-identifies: the token id is the outpoint of the deploy output itself. All subsequent outputs reference it in binary form — the same 36-byte outpoint encoding that appears in sighash preimages, so covenants can compare a token id against a spent outpoint without conversion.

There is no explicit `burn` operation. Tokens are burned implicitly by spending value inputs without creating matching value outputs.

## Amounts

Amounts are Bitcoin script numbers: minimally encoded, non-negative, little-endian sign-magnitude — the exact byte format BSV script arithmetic consumes, so `OP_BIN2NUM`, `OP_ADD`, and comparison opcodes operate on the pushed value directly. There is no width limit: BSV script numbers are unbounded after Genesis, and shrug adds no artificial cap. An output whose amount push is non-minimal or negative is not a shrug output. An amount of 0 is not a token value; it marks the output as a minting authority.

The unbounded domain is a capability difference from BSV-21, which caps each output's amount at 2^64-1. Neither protocol bounds total supply — a cap only limits a single output — so shrug drops the constraint rather than enforce a rule script does not have. Implementations must accumulate amounts with arbitrary-precision arithmetic; fixed-width accumulators can overflow even under BSV-21's per-output cap.

## Metadata

The token protocol itself carries no metadata — no symbol, icon, or decimals in the prefix. Display metadata is a separate layer: a CBOR inscription on the deploy output with content type `application/shrug+cbor`, reusing the BSV-21 field semantics with native binary encoding.

The document is a CBOR map, deterministically encoded (RFC 8949 §4.2: definite lengths, sorted keys), with text-string keys. Unknown keys are ignored.

| Key | CBOR type | Description |
|---|---|---|
| `sym` | text string | Token symbol. Uniqueness is not enforced |
| `icon` | byte string (36 bytes) | Outpoint of an inscription or B protocol file — 32-byte txid + 4-byte little-endian vout, same encoding as the prefix token id |
| `dec` | unsigned integer | Decimal precision 0-18, default 0 |

Diagnostic notation example:

```
{"sym": "GOLD", "icon": h'11…01000000', "dec": 8}
```

All fields are optional, and the document may be omitted entirely. Indexers read metadata from the deploy output and apply it to the token, the same way BSV-21 deploy fields are inherited. Deterministic encoding means two encoders always produce identical bytes, so the document can be hashed, signed, or deduplicated reliably. The `+cbor` structured suffix identifies the serialization; a future encoding of the same document is a new suffix, not a new standard.

## Composition

The prefix is stack-neutral and makes no claim about the rest of the script, so shrug composes with other script-level protocols by concatenation. In particular, a standard 1Sat inscription envelope can sit between the prefix and the owner script:

```
<shrug prefix> <inscription envelope> <owner locking script>
```

Decoders layer cleanly: the shrug parser peels the prefix and hands the remainder to the inscription parser. By convention, content and metadata belong on the deploy output; transfer outputs carry the bare prefix.

### Non-Fungible Ordinals

A deploy with supply 1 carrying an inscription is a non-fungible token — and remains a completely standard 1Sat ordinal. What the prefix adds is origin identification in the locking script:

- Ordinal origin resolution normally requires crawling the spend chain back to an unknown genesis. With the prefix, every output carries its origin as a stable 36-byte token id, and shrug validation verifies it incrementally — each amount-1 output is admitted only if it spends the amount-1 input of the same token id — so the origin is proven without any crawl.
- Indexers can limit their scope to shrug-prefixed ordinals for cheap subset indexing.
- Inscription-only indexers parse the envelope as usual and ignore the prefix entirely; the output is not invalidated for them in any way.

The origin data has three consumption tiers on the same output: verified (shrug-validating indexers), untrusted hint (reading the script without validation), or invisible (legacy 1Sat indexers).

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
| Metadata (`sym`, `icon`, `dec`) | Optional at deploy | Inscription on deploy output (`application/shrug+cbor`) |
| Amount | String uint64 in JSON | Script number, unbounded |
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
