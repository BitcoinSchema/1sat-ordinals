---
description: Sending payments to OpNS names
---

# Payments

OpNS names double as payment handles. A name is addressed as standard paymail — `<name>@<domain>` — where the alias is the name itself and the domain is any host serving the paymail capabilities described here. Because ownership and the identity binding both live on-chain, every resolver reads the same state: `alice@1sat.name` and `alice@example.com` pay the same owner.

To receive payments at a name, the holder publishes an identity binding on the name itself. Claiming the name (see [OpNS](opns.md)) and publishing the binding are separate steps.

## Binding (signed PushDrop)

A name is bound by spending its ordinal into a **signed PushDrop lock** that carries the identity key. The binding is the locking script of the current name UTXO — not metadata riding alongside it — so the live UTXO always states its own binding.

| PushDrop parameter | Value |
|---|---|
| Protocol | `[0, 'p 1sat']` |
| Counterparty | `anyone` |
| Key ID | `opns:{txid}_{vout}` of the input spent to create the output |
| Fields | `[identity public key]` (33-byte compressed) |
| Field signature | yes, same derivation |

The bind is self-certifying: the lock key is derived from the identity key itself (`anyone`-side derivation over the protocol and key ID), and the field is signed under the same derivation. Only the holder of the identity key can produce a valid binding for it, and anyone can verify one offline from the script alone — no server, no metadata history.

**The binding lives and dies with the UTXO.** Transferring, listing, or burning the name spends the PushDrop output and re-locks the ordinal as a plain output — the binding is gone. A new owner who wants payments publishes their own binding after acquiring the name. Deregistering spends the name back to an ordinary P2PKH, removing the binding without moving ownership.

### Wallet actions (`@1sat/actions`)

| Action | Role |
|---|---|
| `opnsRegister` | Publish the binding — re-lock the name with a signed PushDrop carrying the wallet identity key |
| `opnsDeregister` | Remove the binding — spend back to plain P2PKH |
| `opnsTransfer` | Transfer the name ordinal (unlocks a bound name; binding does not carry) |
| `opnsList` | List the name for sale (ordlock) |
| `getOpnsNames` | List OpNS names in the wallet |

## Resolution

A paymail server answers "where do I pay `alice`?" as follows:

1. **Origin** — look up the name's origin outpoint in the OpNS overlay.
2. **Current state** — from that origin, resolve the ordinal's latest outpoint and load its locking script.
3. **Binding** — decode the PushDrop, take the identity key from the first field, re-derive the lock key from it, and verify both the lock and the field signature. Payment requires a valid binding; an unbound name (plain lock, or failed verification) is not payable.

## How the paymail server works

1. **Discovery** — `.well-known/bsvalias` advertises the capabilities below.
2. **PKI** — `GET /id/:paymail` returns the bound **identity public key**.
3. **Payment destination** — `POST /p2p-payment-destination/:paymail` with the amount:
   - Resolve and verify the binding.
   - Derive a one-shot P2PKH with BRC-29 (server **anyone** key vs recipient identity key, random derivation prefix and suffix); store pending with script, amount, prefix, suffix, and identity under a `reference`.
   - Return `reference` and the output script to the payer.
4. **Receive** — payer posts the signed payment to `receive-beef` (or `receive-transaction`) with that `reference`:
   - Server verifies the transaction against the pending destination.
   - Broadcasts the transaction.
   - Posts a remittance to the recipient's messagebox (`payment_inbox`, addressed by identity public key) so the wallet can internalize: BEEF, output index, derivation prefix/suffix, sender identity key, satoshis, and alias.
5. **Wallet** — the owner's wallet lists `payment_inbox`, internalizes with the derivation remittance, and acknowledges the message.

## Paymail Capabilities

Servers expose the standard bsvalias surface, discovered via `.well-known/bsvalias`:

| Capability | Endpoint | Purpose |
|---|---|---|
| `pki` | `/id/:paymail` | The identity key bound to the name |
| `f12f968c92d6` | `/public-profile/:paymail` | Display name and avatar |
| `2a40af698840` | `/p2p-payment-destination/:paymail` | Payment outputs for a requested amount |
| `5c55a7fdb7bb` | `/receive-beef/:paymail` | Deliver the signed payment as BEEF |
| `5f1323cddf31` | `/receive-transaction/:paymail` | Deliver the signed payment as a raw transaction |

## Notes

- The binding is the current UTXO's locking script: spending the name removes it, so a bound name is always bound by its current holder — stale bindings from prior owners cannot linger.
- Every destination is a one-shot BRC-29 derivation: no address reuse, and payments are internalized through the wallet inbox rather than sitting at a static address.
- Bindings verify offline from the script alone; resolvers need the OpNS index only to find the name's latest outpoint.
