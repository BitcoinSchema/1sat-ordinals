---
description: Using 1Sat Ordinals with existing data protocols
---

# Enriching Ordinals

### OP_RETURN

```
NOTE: Enriching an ordinal with OP_RETURN is completely optional.
```

BSV has several protocols for tagging on-chain data that pre-date Ordinals. 1Sat Ordinals can be extended using many existing protocols during inscription, or even to tag the sat as it is spent. This enables things like minting files > 10MB, attaching metadata to inscriptions, creating collections, and signing with identity keys.

```
1SAT_P2PKH INSCRIPTION OP_RETURN
```

Notice we do not use `OP_FALSE OP_RETURN` as is typical in BSV. This is important as omitting `OP_FALSE` allows us to spend the output.

In this example, we geotag an ordinal with a location using Magic Attribute Protocol.

{% code overflow="wrap" %}

```
OP_DUP OP_HASH160 <PUBKEY> OP_EQUALVERIFY OP_CHECKSIG OP_FALSE OP_IF 6f7264 OP_1 <content-type> OP_0 <data> OP_ENDIF OP_RETURN 3150755161374b36324d694b43747373534c4b79316b683536575755374d74555235 534554 617070 6f72642d64656d6f 74797065 706f7374 636f6e74657874 67656f68617368 67656f68617368 6468786e643170776e
```

{% endcode %}
