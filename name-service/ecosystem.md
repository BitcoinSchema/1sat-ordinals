---
description: Names on Bitcoin — what OpNS is and how the pieces around it fit together
---

# Name Service

An OpNS name — `alice`, `pizza`, `node-7` — is a single satoshi on the Bitcoin SV blockchain. You claim one by doing a bit of computational work, and it's then yours the way a coin is yours: hold it, send it, sell it. Attach an identity to it and people can pay you at that name like an email address.

Nobody registers it for you, and there's nothing to renew.

## What it is

Names are mined one character at a time. Each character is a transaction carrying a proof of work — a few million hash attempts, seconds on an ordinary computer. Mining `alice` along an untouched path takes five transactions, and you keep `a`, `al`, `ali`, and `alic` as well.

The rules live in the transaction, not in a service checking submissions. An invalid mint isn't rejected by a gatekeeper; it can't confirm. Each character can be claimed only once from a given prefix, so a name can be minted once and never again.

After the mint, the mining rules are done with it. The name is an ordinary 1Sat ordinal that transfers and sells like any other, and it never expires.

## Why it works

Nothing about a name is stored off-chain. The proof of work, the character claimed, and the name itself are all in the transaction that mints it — so anyone can rebuild the entire namespace by starting at the genesis output in block 806214 and following the spends.

That costs one transaction per character, which for a namespace in real use is millions of small transactions. OpNS treats that as an ordinary thing to ask of Bitcoin, and that's what keeps the state on-chain rather than in somebody's database.

Nobody rebuilds from genesis per lookup, though. An **overlay** is a node that tracks just the OpNS slice of the chain and syncs it with other overlays. It's a cache, not an authority: if one disappears its peers hold the same set, and if one answers wrongly the chain says so.

## The network

| Piece | What it does |
|---|---|
| **Overlay** | Which names are taken, where each was minted, which prefixes are still minable |
| **Paymail host** | Where to send money for `alice` — reads the identity bound to the name |
| **Delivery** | Tells the recipient's wallet that a payment arrived |
| **Wallets & apps** | Search, mine, hold, transfer, sell |

None of them holds a name; that sits in the owner's wallet. Any of them can be swapped for another, because the name and its identity are both on-chain.

## What this is for

A handle people can pay, with each payment going to a fresh output so nothing is reused. An asset you can sell without anyone's approval. An identity key anchored to a name, so the name can verify a signature and not just receive money.

## Protocol

How names are mined and proven unique → [OpNS](opns.md)

How a name becomes payable → [Payments](payments.md)
