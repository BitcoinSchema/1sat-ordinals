---
description: Names on Bitcoin — the network around OpNS
---

# Name Service

A name on Bitcoin.

Claimed once. Held like any other asset. Paid to like an email address. No registrar. No renewal. No company in the middle of the truth.

---

## What it is

**OpNS** is a name system written directly on Bitcoin SV.

You mine a name. It becomes yours — a single satoshi you can keep, send, or sell.  
You can attach an identity so people pay you at that name.  
You can leave. The name stays on the chain.

---

## Why it works

Bitcoin here is big enough to hold the data.

Names settle on-chain. What sits around them is an **overlay** — a peer network that tracks only what it cares about and syncs that state with other peers.

You do not need a full-chain indexer or a hosted subscription to know the namespace.  
An overlay node learns the name tree from peers, serves lookups, and accepts new mints.  
If one node goes away, others still have the graph.

Payment hosts and wallets sit on that overlay the same way.  
They are not the source of truth. The chain is. The overlay is how applications stay current without central infrastructure.

---

## The network

| | |
|---|---|
| **Overlay** | Peer-synced name state: taken or open, origins, new mints. |
| **Payment host** | Where do I send money for `alice@…`? |
| **Delivery** | Tell the wallet a payment arrived. |
| **Apps & wallets** | Search, claim, hold, list, transfer. |

Run your own overlay. Peer with others. Point apps at any honest node.

Hosts and apps are interchangeable because the overlay is shared state — not a company's database.

---

## What this is for

A human-readable handle that settles on-chain.

A payment destination that does not depend on one company's servers.  
A namespace kept alive by overlay peers, not by a single indexer.

Not a whitepaper stack. The pieces run: mint, own, resolve, pay.

---

## Protocol

How names are mined and proven unique → [OpNS](opns.md)  
How a name becomes payable → [Payments](payments.md)
