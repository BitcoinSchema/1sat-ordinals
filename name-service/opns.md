---
description: Low-difficulty proof-of-work name service for Bitcoin SV
---

# OpNS

## Overview

OpNS is a name service for Bitcoin SV. Names are claimed permissionlessly by low-difficulty proof of work — enough cost to deter spam, not a serious mining race — with no registrar, no auction, and no renewal. Each name can exist exactly once. A claimed name is an ordinary 1Sat ordinal inscription, so holding, transferring, and selling a name works like any other ordinal.

Uniqueness is enforced on-chain by a stateful covenant. Contract UTXOs form a trie — the *mine tree* — where each live UTXO represents a name prefix. Spending a node extends its prefix by one character, and the spend is only valid with a proof-of-work solution. Every spend emits the newly formed name as an inscription.

## Protocol Constants

| Constant | Value |
|---|---|
| Genesis outpoint | `58b7558ea379f24266c7e2f5fe321992ad9a724fd7a87423ba412677179ccb25_0` |
| Difficulty | 22 bits |
| Character set | `a-z`, `0-9`, `-` |
| Content type | `application/op-ns` |
| Protocol marker | `1opNSUJVbBc2Vf8LFNSoywGGK4jMcGVrC` |

The genesis output (block 806214) holds the root contract instance — the empty prefix from which all names descend. Difficulty and an optional TLD suffix were fixed at deployment; the mainnet deployment uses 22 bits and no TLD. The original contract source is at [github.com/shruggr/opns](https://github.com/shruggr/opns) for reference and for validating the deployed bytecode.

## The Mine Tree

Each node in the tree carries four state variables:

| Field | Size | Description |
|---|---|---|
| `genesis` | 36 bytes | Outpoint of the deployment output (txid + little-endian vout). Identifies the lineage |
| `claimed` | variable | Little-endian bitmask; bit *n* set means the character with ASCII code *n* has been claimed from this node |
| `domain` | variable | The name prefix this node represents |
| `pow` | 32 bytes | The previous proof-of-work solution hash; seed for the next solution |

A freshly spawned node has `claimed = 0x00` and `domain` equal to its full prefix. The root node's domain is the empty string.

## The Mint Operation

Nodes are not owned. The contract has a single public method, `mint`, with no signature check — anyone who produces a valid proof of work may spend a node. The contract enforces everything:

**Character validation.** The character must be in the allowed set (`a-z`, `0-9`, `-`), and its bit must not already be set in `claimed`. Each character can be claimed from a given node exactly once — and since every name is exactly one path through the tree, each name can only ever be minted once.

**Proof of work.** The solution is a nonce such that:

```
hash256(pow ‖ char ‖ nonce)
```

has its top 22 bits zero when the hash is read **byte-reversed** (leading zero bits of that view) — about 4.2 million attempts (2²²) on average. `pow` is the spent node's 32-byte seed, `char` the single character byte, `nonce` miner-chosen. The winning hash becomes the `pow` seed of both resulting nodes, so solutions chain: future work cannot be precomputed.

**Output structure.** The contract verifies `hashOutputs` against exactly this layout (extending prefix `app` with `l`):

```
Inputs:
  0: node "app" (1 sat)
  1+: funding inputs (unconstrained)

Outputs:
  0: node "app" restated — 'l' marked claimed, new pow seed (1 sat)
  1: node "appl" — new child, nothing claimed (1 sat)
  2: name inscription "appl", locked to the miner's chosen script (1 sat)
  3+: change and other outputs (unconstrained)
```

Output 0 keeps the prefix alive so its remaining characters stay minable. Output 1 opens the next level of the tree. Output 2 is the name itself.

The unlocking script pushes five values: the character, the nonce, the owner locking script for output 2, the concatenated raw bytes of any outputs past index 2, and the sighash preimage (`SIGHASH_ALL | ANYONECANPAY | FORKID`), which the contract uses for output introspection.

**Mining races.** Every spend consumes the node and recreates it at a new outpoint with a new seed. Two miners working from the same node — even on different characters — are racing for one UTXO: only one spend confirms, and the loser's solution is worthless because the seed changed. Mining from any given node is inherently serialized.

## Name Inscriptions

Output 2 is a standard 1Sat ordinal inscription, built by the contract:

```
<owner locking script>
OP_FALSE OP_IF
  "ord"
  OP_1 "application/op-ns"
  OP_0 <name>
OP_ENDIF
OP_RETURN
  "1opNSUJVbBc2Vf8LFNSoywGGK4jMcGVrC"
  <36-byte genesis outpoint>
```

The content type is `application/op-ns` and the content is the full name. The `OP_RETURN` carries the protocol marker and the genesis outpoint so indexers can recognize a claimed OpNS name and its lineage without decoding the mine tree — though the claim still requires verification (see [Validation Rules](#validation-rules)).

Every character mint emits an inscription: mining `appl` when only `ap` existed also mints `app` along the way. Each intermediate name goes to whatever lock the miner supplied for that step.

## Ownership and Transfer

The contract's involvement ends when the inscription is created. The name ordinal is an ordinary spendable output: transfers, sales (e.g. [Ordinal Lock](../ordinal-lock.md)), and custom locking scripts all work exactly as for any other 1Sat ordinal. The current owner of a name is the current holder of its ordinal, resolved by standard ordinal tracking from the name's origin. Names never expire.

## Identity Binding

By application convention, a name can be bound to an identity key and used as a paymail handle. This is not part of the contract protocol — it rides on ordinal transfers as MAP metadata. See [Payments](payments.md).

## Validation Rules

**Mints are valid by construction.** The script enforces proof of work, the character set, the claimed bitmask, and the exact output structure. An invalid mint cannot be confirmed, so any observed spend of a genuine node is a valid mint.

**Lineage must be verified.** Anything in a script can be forged — a fake node or an inscription carrying the genesis outpoint in its `OP_RETURN` proves nothing by itself. A name is genuine only if its mint traces back through node spends to the genesis outpoint. Indexers must follow the tree from genesis rather than pattern-match scripts.

**Names are unique.** Each character is claimable once per node, each name is one path through the tree, so a genuine name has exactly one origin.

**Transfers need no OpNS validation.** Once minted, a name ordinal follows ordinary ordinal rules; only the mint itself is protocol-governed.

## Resolution

An OpNS indexer starts at the genesis outpoint and follows spends of node outputs (outputs 0 and 1 of each mint), producing two kinds of records:

- **Prefix nodes** — the live UTXO for each prefix, where mining continues
- **Name origins** — the inscription outpoint where each name was minted

Availability follows from the tree: a name is taken if a node with that exact domain exists (it was spawned when the name was minted). If not, the longest mined prefix is the node to mine from — the remaining characters each take one mint transaction.

The hosted overlay at `api.1sat.app` exposes this index:

| Endpoint | Returns |
|---|---|
| `GET /1sat/opns/origin/:name` | The origin outpoint of a claimed name |
| `GET /1sat/opns/mine/:name` | Nothing if the name is taken; otherwise the longest mined prefix and its live node outpoint |
| `POST /1sat/opns/origins` | For a JSON array of outpoints, which are genuine OpNS origins |

Ownership and metadata lookups from an origin use standard ordinal resolution; content and accumulated MAP are available through [OrdFS](../ordfs.md).

## Mining a Name

Mining `ab` from an empty tree takes two transactions.

**1. Mine `a` from the root**

```
Input:  root node "" (seed S0)

Outputs:
  0: node "" — 'a' claimed, seed S1
  1: node "a" — nothing claimed, seed S1
  2: inscription "a" → miner's lock
```

The miner found a nonce where `hash256(S0 ‖ 'a' ‖ nonce)` meets difficulty (22 leading zero bits of the byte-reversed hash); that hash is `S1`.

**2. Mine `b` from node `a`**

```
Input:  node "a" (seed S1)

Outputs:
  0: node "a" — 'b' claimed, seed S2
  1: node "ab" — nothing claimed, seed S2
  2: inscription "ab" → miner's lock
```

The names `a` and `ab` now exist, each as a 1-sat ordinal. The root can still mint `b` through `z`, node `a` can still mint every character except `b`, and node `ab` is open for a third character. No transaction can ever mint `a` or `ab` again.

## Summary

OpNS names are mined, not registered. A covenant trie makes each name mintable exactly once, proof of work is the only cost of claiming one, and the result is a plain 1Sat ordinal that transfers and trades like any other. The contract enforces the entire mint; indexers only need to follow the tree from genesis to know every name, its origin, and what remains available.

## Reference implementations

- **TypeScript template** — `@1sat/templates` `OpNS` (`lock`, `decode`, `unlock`, `buildInscription`, `claimBit`, `testSolution`)
- **Go template / overlay** — `1sat-stack` `pkg/template/opns` and `pkg/opns` (mine-tree index and HTTP API above)

Identity binding and paymail are application conventions; see [Payments](payments.md).
