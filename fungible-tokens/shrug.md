---
description: Script-native fungible token protocol for Bitcoin SV
---

# ¯\\_(ツ)\_/¯ (Shrug)

{% hint style="warning" %}
¯\\_(ツ)\_/¯ is experimental and the specification may still change. For production tokens, use [BSV-21](bsv-21.md).
{% endhint %}

## Overview

Shrug is a fungible token protocol for Bitcoin SV where the token data lives directly in the locking script as plain data pushes. Every token output starts with a short, fixed prefix — the shrug tag, a token id, and an amount — followed by an ordinary locking script.

Shrug is an evolution of [BSV-21](bsv-21.md) and follows the same general rules. If you know BSV-21, the mapping is summarized in the comparison below; if you don't, this page stands on its own.

## Why not BSV-21?

BSV-21 has wide adoption and works well when wallets and indexers are the only software handling tokens. Its weak spot is Bitcoin script. Token data is a JSON document inside an inscription envelope, and script cannot work with that easily:

- A contract that creates a token output must build JSON in script: assemble the inscription envelope, quote the fields, and convert amounts from numbers into ASCII decimal strings.
- A contract that checks a token output does the same in reverse — hunting for fields inside a text document instead of reading bytes at known positions.
- Token ids are hex text in the opposite byte order from the outpoints script sees in sighash preimages, so even comparing an id means converting and reversing first.

Shrug puts the same data where script can use it. The amount is a script number, so arithmetic opcodes use the pushed value as-is. The token id is the same 36 bytes as a preimage outpoint, so comparing them is one equality check. Building a new token output is concatenating a few pushes. Everything else about the token model stays as BSV-21 defined it.

## Wire Format

```
<push "¯\_(ツ)_/¯"> <push token id | OP_0> OP_2DROP <push amount | OP_0> OP_DROP <owner locking script>
```

| Element | Encoding |
|---|---|
| Tag | Push of the 13-byte UTF-8 string `¯\_(ツ)_/¯` (hex `c2af5c5f28e38384295f2fc2af`) |
| Token id | Push of a 36-byte outpoint (32-byte txid + 4-byte little-endian vout), or `OP_0` on deploys |
| `OP_2DROP` | Drops the tag and id from the stack |
| Amount | Push of the amount as a script number, or `OP_0` for zero |
| `OP_DROP` | Drops the amount |
| Owner script | Any locking script — P2PKH, multisig, a custom contract |

The prefix pushes three values and drops all three, so the owner script runs exactly as it would on its own.

## Token Identity

A token is identified by the outpoint of the output that created it — its deploy output — written as 36 bytes: the txid followed by the output index. A deploy output leaves the id field empty (`OP_0`); its own outpoint becomes the token id. Every later output for that token carries the id.

This is the same 36-byte outpoint encoding that appears inside sighash preimages, so a covenant can compare a token id against a spent outpoint byte for byte.

## Operations

The two prefix fields say everything about what an output does:

| Token id | Amount | Meaning |
|---|---|---|
| Empty | > 0 | Deploy a token; the fixed supply is held in this output |
| Empty | 0 | Deploy a token; this output is the initial minting authority |
| Present | 0 | Minting authority |
| Present | > 0 | Token value |

Tokens are burned by spending them without creating matching outputs.

## Amounts

Amounts are script numbers — the same little-endian format Bitcoin's arithmetic opcodes work with, so `OP_BIN2NUM` or `OP_ADD` can use the pushed value as-is. They must be minimally encoded and non-negative, and there is no upper limit: script numbers have no fixed width. An amount of zero marks the output as a minting authority rather than a token value.

Since amounts have no width limit, software that adds them up must use arithmetic that cannot overflow.

## Satoshi Value

By convention, token outputs hold exactly 1 satoshi, and that is the recommended default. Wallets and indexers across the 1Sat ecosystem are built around single-satoshi outputs, and a deploy output carrying an inscription needs one identifiable satoshi for the inscription to bind to.

This is a convention, not a strict protocol rule. Validation reads only the script, so a token output may hold any number of satoshis, and the token and the satoshis travel together when the output is spent. Carrying additional value is an advanced option: it can have legal and regulatory implications depending on how a token is structured. Policy may dictate validation be limited to single satoshi outputs only.

## Metadata

Display information — symbol, icon, and decimal precision — lives in its own document: a CBOR inscription on the deploy output with content type `application/shrug+cbor`.

The document is an open key/value map. The deployer may include any data they like; these keys have defined meanings:

| Key | CBOR type | Description |
|---|---|---|
| `sym` | text string | Token symbol. Uniqueness is not enforced |
| `icon` | byte string (36 bytes) | Outpoint of an inscription or B protocol file — same encoding as the token id |
| `dec` | unsigned integer | Decimal precision 0-18, default 0 |

Diagnostic notation example:

```
{"sym": "GOLD", "icon": h'11…01000000', "dec": 8}
```

The document is encoded deterministically (RFC 8949 §4.2) — the same fields always produce the same bytes, so it can be hashed or signed reliably. Keys are text strings; readers use the keys they understand. The spec may define more keys over time. All fields are optional, and so is the document itself. Indexers read the metadata once from the deploy output and apply it to the whole token.

## Composition

The prefix makes no claims about the rest of the script, so it stacks with other script-level protocols by simple concatenation. In particular, a standard 1Sat inscription envelope can sit between the prefix and the owner script:

```
<shrug prefix> <inscription envelope> <owner locking script>
```

A shrug decoder reads the prefix and hands the rest to the inscription decoder. By convention, content and metadata go on the deploy output; transfer outputs carry just the prefix.

### Non-Fungible Ordinals

A deploy with a supply of 1, carrying an inscription, is a non-fungible token — and still a completely normal 1Sat ordinal. What the prefix adds is the token's origin, right in the locking script:

- Normally, finding an ordinal's origin means walking the spend chain backwards to its genesis. With the prefix, every output states its origin, and shrug validation proves the claim one transaction at a time — each output is only valid if it spends a valid input of the same token — so no walk is ever needed.
- An indexer can choose to track only shrug-prefixed ordinals and skip origin crawling entirely.
- Indexers that only understand inscriptions see a normal ordinal and ignore the prefix. Nothing about the output is invalidated for them.

The same origin data serves three audiences: shrug indexers verify it, anyone reading the raw script can use it as a hint, and inscription-only indexers never see it.

## Validation Rules

**Deploys** (empty id) are always valid. The output's own outpoint becomes the token id.

**Authority outputs** (id present, amount 0) are valid only when the transaction spends a valid authority output of the same token. A deploy with amount 0 is the token's first authority. Authority can be:

- Split — one authority input, many authority outputs
- Combined — many in, one out
- Passed to a new owner
- Ended — spend it without creating a new one, and minting stops for good

Spending an authority adds nothing to token balance.

**Value outputs** (id present, amount > 0):

- If the transaction spends a valid authority for the token, its value outputs are valid without needing input balance. This is how new tokens are minted.
- Otherwise, the transaction's value outputs must be covered by its valid value inputs — all of them or none of them, per token.
- If outputs exceed inputs with no authority present, the outputs are invalid and the input tokens are burned.
- If inputs exceed outputs, the difference is burned.

Because minting and transferring look the same on-chain, individual outputs are not labeled one or the other. Circulating supply is the net value created in authority-backed transactions, minus everything burned.

## Comparison with BSV-21

Shrug follows the BSV-21 token model — outpoint identity, UTXO balances, authority-gated minting, balance-checked transfers — re-encoded for script:

| | BSV-21 | Shrug |
|---|---|---|
| Encoding | JSON inscription (`application/bsv-20`) | Binary script prefix |
| Token id | `<txid>_<vout>` string | 36-byte binary outpoint |
| Operations | Explicit `op` field (6 ops) | Implied by field presence |
| Explicit burn | Yes | No (implicit only) |
| Metadata (`sym`, `icon`, `dec`) | Optional at deploy | Inscription on deploy output (`application/shrug+cbor`) |
| Amount | String uint64 in JSON | Script number, no width limit |
| Script access to token data | Requires envelope/JSON parsing | Fixed-position pushes |
| Validation model | Auth-gated minting + balance checks | Same |

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

Transfer 400 of 1,000 held tokens (no authority input — balance rules apply):

```
Inputs: value outpoint holding 1,000 tokens

Output 0 (value): "¯\_(ツ)_/¯" <36-byte token id> OP_2DROP 400 OP_DROP <recipient script>
Output 1 (value): "¯\_(ツ)_/¯" <36-byte token id> OP_2DROP 600 OP_DROP <change script>
```
