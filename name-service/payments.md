---
description: Sending payments to OpNS names
---

# Payments

OpNS names double as payment handles. A name is addressed as standard paymail — `<name>@<domain>` — where the alias is the name itself and the domain is any host serving the paymail capabilities described here. Because ownership and the identity binding both live on-chain, every resolver reads the same state: `alice@1sat.name` and `alice@example.com` pay the same owner.

To receive payments at a name, the holder sets MAP field `opns.idKey` on the name ordinal. Claiming the name (see [OpNS](opns.md)) and setting that binding are separate steps.

## Binding (`opns.idKey`)

A name is bound by self-transferring its ordinal with MAP metadata. The value of `opns.idKey` is one of:

| Form | Value | Paymail behavior |
|---|---|---|
| **Identity (preferred)** | BRC-100 identity public key (hex) | One-shot BRC-29 destinations; messagebox delivery to that identity |
| **Address (legacy)** | Base58Check P2PKH address | Fixed P2PKH script to that address; no derivation; no messagebox |
| **Clear** | empty string | Binding removed |

Resolvers try **hex public key first**, then a valid Base58Check P2PKH address. There is a single MAP field — not two keys with priority rules.

The on-chain state of the name *is* the registration — there are no server-side accounts. Resolvers read the name's accumulated metadata, so the binding in effect is whatever the most recent write says.

Metadata persists across transfers until overwritten. A new owner who wants payments should set their own binding after acquiring a name.

### Wallet actions (`@1sat/actions`)

| Action | Role |
|---|---|
| `opnsRegister` | Set `opns.idKey` — wallet identity key by default, or optional `address` for legacy Base58 binding |
| `opnsDeregister` | Clear the binding (empty string) |
| `opnsTransfer` | Transfer the name ordinal |
| `opnsList` | List the name for sale (ordlock) |
| `getOpnsNames` | List OpNS names in the wallet |

## Resolution

A paymail server answers "where do I pay `alice`?" as follows:

1. **Origin** — look up the name's origin outpoint in the OpNS overlay.
2. **Current state** — from that origin, resolve the ordinal's tip and **merged MAP** via [OrdFS](../ordfs.md) (latest sequence).
3. **Binding** — read `opns.idKey` as identity hex or legacy address. Payment requires a usable binding.

## How the paymail server works

1. **Discovery** — `.well-known/bsvalias` advertises the capabilities below.
2. **PKI** — `GET /id/:paymail` returns the registered **identity public key** when the binding is hex identity. Legacy address bindings have no pubkey for this endpoint.
3. **Payment destination** — `POST /p2p-payment-destination/:paymail` with the amount:
   - Resolve the name and `opns.idKey`.
   - **Identity:** derive a one-shot P2PKH with BRC-29 (server **anyone** key vs recipient identity key, random derivation prefix and suffix); store pending with script, amount, prefix, suffix, and identity under a `reference`.
   - **Address (legacy):** lock to that P2PKH address; store pending with script and amount under a `reference` (no derivation fields, no identity for inbox).
   - Return `reference` and the output script to the payer.
4. **Receive** — payer posts the signed payment to `receive-beef` (or `receive-transaction`) with that `reference`:
   - Server verifies the transaction against the pending destination.
   - Broadcasts the transaction.
   - **Identity:** posts a remittance to the recipient's messagebox (`payment_inbox`, addressed by identity public key) so the wallet can internalize: BEEF, output index, derivation prefix/suffix, sender identity key, satoshis, and alias.
   - **Address (legacy):** no messagebox step; funds sit at the registered address on-chain.
5. **Wallet (identity path)** — the owner's wallet lists `payment_inbox`, internalizes with the derivation remittance, and acknowledges the message.

## Paymail Capabilities

Servers expose the standard bsvalias surface, discovered via `.well-known/bsvalias`:

| Capability | Endpoint | Purpose |
|---|---|---|
| `pki` | `/id/:paymail` | The identity key bound to the name (identity binding only) |
| `2a40af698840` | `/p2p-payment-destination/:paymail` | Payment outputs for a requested amount |
| `5c55a7fdb7bb` | `/receive-beef/:paymail` | Deliver the signed payment as BEEF |
| `5f1323cddf31` | `/receive-transaction/:paymail` | Deliver the signed payment as a raw transaction |

## Notes

- The binding follows the name, not the person: whoever holds the ordinal can rewrite it, and it stays in effect across transfers until rewritten.
- Prefer identity hex for private one-shot destinations and wallet inbox delivery.
- Address binding is legacy: static destination, address reuse, no messagebox.
