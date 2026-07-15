# Resolving Ordinals

To resolve ordinal numbers for unspent outputs an indexer is required. This diagram shows the process of resolving an ordinal number.

![Ordinals Indexing](https://github.com/BitcoinSchema/1sat-ordinals/blob/main/Ordinals_Indexer.jpg?raw=true)

## Current implementation

Production indexing and HTTP resolution for this ecosystem are provided by **[1sat-stack](https://github.com/b-open-io/1sat-stack)** (see [Libraries](../Libraries.md)). Content and transfer-chain resolution over HTTP are described in **[OrdFS](../ordfs.md)**.

## Historical reference implementations

{% hint style="info" %}
**Deprecated for new work.** The following repositories are early indexers/servers. Prefer **1sat-stack**.
{% endhint %}

{% embed url="https://github.com/shruggr/1sat-server" %}

{% embed url="https://github.com/shruggr/1sat-indexer" %}
