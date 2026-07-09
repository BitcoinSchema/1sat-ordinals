# Shrug Parity Plan

Status: **Decisions resolved 2026-07-09 — ready to implement (Phases 1–2 first)**

Goal: bring the shrug token (`¯\_(ツ)_/¯`) to functional parity with BSV-21: a working parser, balance/auth validation, overlay topics, a lookup service, and a published spec. Dynamic overlay topic infrastructure is out of scope.

## Background

Shrug is a binary encoding of the BSV-21 token rules. Token data is pushed directly in the locking script — no inscription envelope, no JSON:

```
<push "¯\_(ツ)_/¯"> <push 36-byte token id | OP_0> OP_2DROP <push amount (script number) | OP_0> OP_DROP <owner locking script>
```

Operations are implicit rather than named:

| Encoding | BSV-21 equivalent |
|---|---|
| No id (`OP_0`), amount > 0 | `deploy+mint` |
| No id, amount 0 | `deploy+auth` |
| id present, amount 0 | `auth` |
| id present, amount > 0 | `transfer` / `mint` (see D2) |

The token id is the deploy outpoint (36 bytes binary: txid + LE vout), identical in meaning to BSV-21's `<txid>_<vout>`.

## Current state (2026-07-09)

| Component | State |
|---|---|
| `go-templates/template/shrug` | Encoder works; decoder can never match (tag check bug); round-trip test skipped for lack of a valid vector |
| `1sat-stack/pkg/parse/shrug.go` | Parse-only wrapper over the broken decoder; registered but not in DefaultTags; no validation, topic, or lookup |
| `1sat-engine` Go / proto / Zig parsers | Parse-only; proto and Zig drop the amount entirely; event names diverge from stack |
| `1sat-indexer/mod/shrug` | Only implementation with validation (deploy auto-valid, zero-amount = mint authority, balance conservation); dormant — never registered; latent nil `big.Int` bug |
| TS SDK / ts-stack | No shrug support |
| Spec | None; the code is the de facto spec and is self-contradictory |

## Decisions (resolved 2026-07-09)

**D1 — Tag check.** Canonical form is the full 13-byte UTF-8 tag push, matching the encoder and the Zig parser. The Go decoders' 9-byte/8-byte-prefix check is a bug to fix.

**D2 — Mint vs transfer: contextual validation.** The binary format has no op field and doesn't need one. If the transaction spends a valid auth input for the token, value outputs are admitted without balance coverage (minting); otherwise input/output conservation applies. Same outcomes as BSV-21. The only difference: individual outputs can't be labeled "mint" vs "transfer", so new supply is computed as the value delta (outputs − inputs) of auth-bearing transactions rather than by summing mint outputs.

**D3 — Burn: deferred.** Implicit burning (outputs < inputs) only; an explicit encoding is left as a future extension.

**D4 — Amount domain.** Amounts capped at uint64; larger script numbers fail to decode. Matches BSV-21 exactly.

**D5 — Metadata: separate inscription layer.** The prefix carries no metadata. Display metadata (`sym`/`icon`/`dec`, BSV-21 field names) is a JSON inscription on the deploy output with content type `application/shrug+json`. Shared data structures keep the fields nullable. Composition: an inscription envelope may sit between the prefix and the owner script — also the basis for non-fungible ordinals (supply 1 + inscription, origin identified in the locking script). No version field in the prefix; evolution happens via new tags or composed prefixes.

**D6 — Event naming.** Follow the BSV-21 event conventions in each repo.

**D7 — Data structures.** Reuse the BSV-21 shapes (id, op, amt, nullable sym/dec/icon). Shrug parsing populates `op` from the implicit encoding per the table above. Validation implementation is copied, not shared (see Phase 2).

## Work items

### Phase 1 — Spec + parser
1. Resolve D1–D7; write `fungible-tokens/shrug.md` in this repo with the same structure as `bsv-21.md`
2. `go-templates/template/shrug`: fix `Decode` (tag check; id branch currently skips amount parsing and misparses the opcode sequence), enforce the amount domain, add real test vectors, un-skip the round-trip test

### Phase 2 — Validation + topics (1sat-stack)
3. Topic managers in a new `pkg/shrug` mirroring `pkg/bsv21`: discovery (deploys) + validated (auth-gated minting, conservation for value outputs). Copy the pattern; leave the deployed BSV-21 path untouched. Unify into a shared core later if warranted
4. Lookup service: the BSV-21 `token_outputs` schema fits as-is; register the shrug topics in server config. Owner extraction must peel an optional inscription envelope from the script suffix before decoding the lock. Read `application/shrug+json` metadata from deploy outputs into sym/dec/icon
5. Emit events per D6

### Phase 3 — Engine + SDK
6. `1sat-engine`: add amount to the proto and Zig parsers, align event names, and port validation (note: engine currently lacks BSV-21 auth validation too — its `topic_validated.go` is the old balance-only version)
7. TS: shrug template in `@1sat/templates` (lock/decode), wallet indexer support, actions — scope as needed

### Out of scope
- Dynamic overlay topic infrastructure
- `1sat-indexer/mod/shrug`: leave dormant as a reference implementation of the validation rules

## Resolved questions
- Topic manager: copy the pattern per protocol (`pkg/shrug`); revisit unification later.
- On-chain state: clean slate — no existing shrug outputs constrain the spec. Defined fresh from the encoder format.
- First branch covers Phases 1 and 2.
