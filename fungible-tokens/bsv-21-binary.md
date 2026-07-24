---
description: Script-native binary encoding for BSV-21 tokens
---

# BSV-21 Binary Encoding

{% hint style="warning" %}
**Status: Draft.** BSV-21 Binary is not final and the wire format may change. For production tokens today, use the [JSON encoding](bsv-21.md).
{% endhint %}

## Overview

BSV-21 Binary is a second wire encoding for the [BSV-21](bsv-21.md) token model. Instead of a JSON document inside an inscription envelope, token data is written as a fixed prefix of data pushes at the start of the locking script.

The token model is unchanged: tokens are identified by their deploy outpoint, balances live in UTXOs, minting is gated by authority outputs, and transfers are balance-checked. A token keeps the same identity in either encoding, and indexers should treat JSON and binary outputs for the same token id as interchangeable.

## Why a binary encoding?

JSON BSV-21 works well when wallets and indexers are the only software handling tokens. Its weak spot is Bitcoin script. Token data is a JSON document inside an inscription envelope, and script cannot work with that easily:

- A contract that creates a token output must build JSON in script: assemble the inscription envelope, quote the fields, and convert amounts from numbers into ASCII decimal strings.
- A contract that checks a token output does the same in reverse, searching for fields inside a text document instead of reading bytes at known positions.
- Token ids are hex text in the opposite byte order from the outpoints script sees in sighash preimages, so even comparing an id means converting and reversing first.

The binary encoding puts the same data where script can use it. The amount is a script number, so arithmetic opcodes can use the pushed value directly. The token id uses the same 36-byte layout as a preimage outpoint, so comparing them is a single equality check. Building a token output is a matter of concatenating a few pushes. Everything else about the token model stays as BSV-21 defined it.

## Wire Format

Every token output begins with the same prefix:

```
<push "BSV21"> <push token id | OP_0> OP_2DROP <push amount | OP_0> <push payload | OP_0> OP_2DROP <rest of script>
```

| Element | Encoding |
|---|---|
| Tag | Push of the UTF-8 string `BSV21` (hex `4253563231`) |
| Token id | Push of a 36-byte outpoint (32-byte txid + 4-byte little-endian vout), or `OP_0` on deploys |
| `OP_2DROP` | Drops the tag and id |
| Amount | Push of the amount as a minimally encoded script number, or `OP_0` for zero |
| Payload | Push of payload bytes (a CBOR map by convention), or `OP_0` if empty |
| `OP_2DROP` | Drops the amount and payload |
| Rest | Owner locking script, optionally preceded by an inscription envelope |

All four pushes are always present; empty values are written as `OP_0`. The prefix pushes four values and drops all four, so the owner script runs exactly as it would on its own.

A script is a BSV-21 Binary output when its prefix matches this layout. Recognition reads the tag, id, and amount; the payload push is opaque bytes and never affects validity.

A decoder reads the prefix and hands the rest of the script to the inscription and lock decoders; an inscription envelope, when present, sits between the prefix and the owner script. Token fields for the binary encoding do **not** use `application/bsv-20` or any content type — they live in the payload push.

## Token Identity

A token is identified by the outpoint of the output that created it — its deploy output — written as 36 bytes: the 32-byte txid followed by the 4-byte little-endian output index. A deploy output leaves the id field empty (`OP_0`); its own outpoint becomes the token id. Every later output for that token carries the id.

This is the same 36-byte outpoint encoding that appears inside sighash preimages, so a covenant can compare a token id against a spent outpoint byte for byte. The txid bytes are in that natural order — the reverse of the hex displayed by explorers and used in the JSON encoding's `<txid>_<vout>` id — so converting between the two forms means reversing the txid bytes.

The two forms name the same token. The encoding does not create a new one.

## Roles

Binary has no `op` field. An output's role is implied by its id and amount:

| Token id | Amount | Role |
|---|---|---|
| Empty | > 0 | Deploy with the fixed supply held in this output |
| Empty | 0 | Deploy; this output is the initial minting authority |
| Present | 0 | Minting authority |
| Present | > 0 | Token value |

There is no explicit burn output type. Tokens are burned by spending value outputs without creating covering value outputs (see [Validation Rules](#validation-rules)).

## Amounts

Amounts are Bitcoin script numbers: minimally encoded, non-negative, little-endian. The maximum value is \(2^{64}-1\), the same domain as the JSON encoding. Note that the maximum value takes nine bytes to encode — eight value bytes plus a leading zero sign byte. Amounts above the maximum, negative values, and non-minimal encodings are invalid.

An amount of zero marks the output as a minting authority rather than a token value.

## Payload

The fourth push is opaque to validation: any bytes are admissible, and recognition and transfer validation depend only on tag, id, and amount — never on payload contents.

Encoders should write either `OP_0` or a CBOR map (RFC 8949), using deterministic encoding (RFC 8949 §4.2) so the same fields always produce the same bytes. Readers parse best-effort:

- A payload that does not decode as a CBOR map is ignored.
- None of the defined keys are required.
- Unknown keys in a decoded map are ignored; they do not invalidate the output.

The payload carries information for indexers, wallets, and applications.

### Fungible token fields

For fungible tokens, the following keys have defined meanings on the deploy output. Deploys should carry these fields so wallets and indexers can display the token. Later outputs normally carry `OP_0` and inherit these values from the deploy; the keys have no protocol meaning on non-deploy outputs.

| Key | CBOR type | Description |
|---|---|---|
| `sym` | text string | Token symbol. Uniqueness is not enforced |
| `icon` | byte string (36 bytes) | Outpoint of an inscription or B protocol file — same encoding as the token id |
| `dec` | unsigned integer | Decimal precision 0–18, default 0 |

The spec may define more keys over time.

## Satoshi Value

By convention, token outputs hold exactly 1 satoshi, matching the rest of the 1Sat ecosystem. This is a convention rather than a protocol rule — validation reads only the script — but policy may limit admission to single-satoshi outputs.

## Cross-encoding

JSON and Binary are two encodings of one token model.

- Each output validates under its own encoding's rules (a JSON `transfer` output requires input coverage even when authority is spent; a binary value output does not). Authority inputs of either encoding gate minting for the token.
- A token deployed as JSON may later have binary value or authority outputs, and the reverse.
- The token id is always the deploy outpoint, whichever encoding wrote it.
- Display fields for a JSON deploy come from the deploy inscription; for a binary deploy, from the deploy payload.
- Overlays and wallets should decode both encodings and aggregate balances by token id.

The JSON protocol identifier (`"p": "bsv-20"`) and content type (`application/bsv-20`) are not used by the binary encoding. The tag push is the only protocol marker.

## Validation Rules

**Deploys** (empty id) are always valid. The output's own outpoint becomes the token id.

**Authority outputs** (id present, amount 0) are valid only when the transaction spends a valid authority of the same token. A deploy with amount 0 is the token's first authority. Authority can be:

- Split — one authority input, many authority outputs
- Combined — many in, one out
- Passed to a new owner
- Ended — spend it without creating a replacement; that authority is destroyed. Minting for the token as a whole ends only when its last authority is spent this way

Spending an authority contributes nothing to token balance.

**Value outputs** (id present, amount > 0):

- If the transaction spends a valid authority for the token, its value outputs are valid without needing input balance. This is how new tokens are minted.
- Otherwise, the transaction's value outputs must be covered by its valid value inputs — all of them or none of them, per token.
- If outputs exceed inputs with no authority present, the outputs are invalid and the input tokens are burned.
- If inputs exceed outputs, the difference is burned.

Because minting and transferring look the same on-chain, individual outputs are not labeled one or the other. Circulating supply is the sum of admitted value UTXOs, not a ledger of mint and burn operations.

## Comparison with JSON Encoding

| | JSON ([bsv-21.md](bsv-21.md)) | Binary (this doc) |
|---|---|---|
| Encoding | JSON inscription (`application/bsv-20`) | Script prefix, tag `BSV21` |
| Token id | `<txid>_<vout>` string | 36-byte outpoint |
| Operations | Explicit `op` field | Implied by id + amount |
| Explicit burn | Yes | No (implicit only) |
| Metadata / extras | JSON fields on deploy | CBOR payload push (open map) |
| Amount | String uint64 | Script number, max \(2^{64}-1\) |
| Protocol id | `"p": "bsv-20"` | Tag push `BSV21` only |
| Script access | Envelope + JSON parsing | Fixed-position pushes |
| Validation model | Auth-gated minting + balance checks | Same, without explicit burn outputs |

## Examples

### Output Scripts

Deploy a token with a fixed supply of 21,000,000, owned by a P2PKH address. The payload carries the token's metadata as a CBOR map:

```
"BSV21" OP_0 OP_2DROP 21000000 <CBOR {"sym": "GOLD", "dec": 8}> OP_2DROP
OP_DUP OP_HASH160 <pubkeyhash> OP_EQUALVERIFY OP_CHECKSIG
```

Deploy an authority-based token (no initial supply):

```
"BSV21" OP_0 OP_2DROP OP_0 <CBOR {"sym": "STABLE", "dec": 2}> OP_2DROP
OP_DUP OP_HASH160 <pubkeyhash> OP_EQUALVERIFY OP_CHECKSIG
```

A value output for an existing token:

```
"BSV21" <36-byte token id> OP_2DROP 5000 OP_0 OP_2DROP <owner script>
```

An authority output for an existing token:

```
"BSV21" <36-byte token id> OP_2DROP OP_0 OP_0 OP_2DROP <owner script>
```

### Fixed Supply Lifecycle

**1. Deploy**

```
Outputs:
  - Deploy: 10,000 tokens (the token id is this output's outpoint)
```

**2. Split the supply**

```
Input:  deploy output (10,000 tokens)

Outputs:
  - Value: 5,000 tokens
  - Value: 5,000 tokens
```

**3. Transfer to a user**

```
Input:  value output (5,000 tokens)

Outputs:
  - Value: 4,900 tokens (recipient)
  - Value: 100 tokens (change)
```

### Authority Lifecycle

**1. Deploy with authority**

```
Outputs:
  - Authority: amount 0 (the token id is this output's outpoint)
```

**2. Mint the first supply**

```
Input:  genesis authority

Outputs:
  - Value: 1,000,000 tokens (newly created — an authority input is present)
  - Authority: amount 0 (keeps minting open)
```

**3. Distribute**

```
Input:  value output (1,000,000 tokens)

Outputs:
  - Value: 500,000 tokens (user A)
  - Value: 500,000 tokens (user B)
```

**4. Delegate authority**

```
Input:  authority

Outputs:
  - Authority: amount 0 (admin A)
  - Authority: amount 0 (admin B)
```

**5. End an authority**

```
Input:  authority

Outputs:
  - (no authority output)
```

This authority is destroyed. Any other authorities for the token keep working; minting is closed for the whole token only when its last authority is spent without a replacement.

### Balance Validation

A valid transfer — outputs covered by inputs:

```
Inputs:
  - Value: 1,000 tokens
  - Value: 500 tokens

Outputs:
  - Value: 800 tokens (recipient A)
  - Value: 600 tokens (recipient B)
  - Value: 100 tokens (change)

Total in: 1,500. Total out: 1,500. Valid.
```

An invalid transfer — outputs exceed inputs with no authority present:

```
Inputs:
  - Value: 500 tokens

Outputs:
  - Value: 300 tokens
  - Value: 400 tokens

Total in: 500. Total out: 700. All outputs invalid; the 500 input tokens are burned.
```

An implicit burn — inputs exceed outputs:

```
Inputs:
  - Value: 1,000 tokens

Outputs:
  - Value: 250 tokens

750 tokens are burned.
```
