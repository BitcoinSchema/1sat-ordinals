# Metadata

Metadata is located after the `OP_RETURN` opcode in the output script of an inscription, and does not interefere with the [inscription protocol](../).

```
1SAT_P2PKH INSCRIPTION OP_RETURN METADATA
```

You can use metadata to add context to ordinals. For example, you can tag an ordinal with collection data, geohashes, external file references, etc. Metadata is written using "Magic Attribute Protocol" which has a prefix of `1PuQa7K62MiKCtssSLKy1kh56WWU7MtUR5`.

(TODO: Find better example using more typical case)

{% code overflow="wrap" %}
```
OP_DUP OP_HASH160 <PUBKEY> OP_EQUALVERIFY OP_CHECKSIG OP_FALSE OP_IF 6f7264 OP_1 <content-type> OP_0 <data> OP_ENDIF OP_RETURN 3150755161374b36324d694b43747373534c4b79316b683536575755374d74555235 534554 617070 6f72642d64656d6f 74797065 706f7374 636f6e74657874 67656f68617368 67656f68617368 6468786e643170776e
```
{% endcode %}

{% hint style="info" %}
Notice we do not use OP\_FALSE OP\_RETURN. This is important as omitting OP\_FALSE allows us to spend the output.
{% endhint %}

In the above example, we geotag an ordinal with a location using Magic Attribute Protocol.

## Schema Types

To help standardize consumption of this data, we define schema types to establish common field names to be shared cross platforms. While schema type cover a broad range of use cases, this document refers the Ordinals schema type `ord` .

## Other Metadata

BSV has several protocols for tagging on-chain data that pre-date Ordinals. 1Sat Ordinals can be extended using many existing protocols during inscription, or even to tag the sat as it is spent. This enables things like minting files > 10MB, attaching metadata to inscriptions, creating collections, and signing with identity keys.

```
NOTE: Enriching an ordinal with OP_RETURN is completely optional.
```

You can still add metadata from other schema types besides "ord" if you prefer. You can find a list of different schema typesa at https://bitcoinschema.org.

{% code overflow="wrap" %}
```bash
1SAT_P2PKH <INSCRIPTION> OP_RETURN <B fields...> | MAP SET app <platform_name> type "post" context "geohash" geohash "dhmgdqvr7"
```
{% endcode %}

Similarly, you can attach an ordinal to a particular url:

{% code overflow="wrap" %}
```bash
1SAT_P2PKH <INSCRIPTION> OP_RETURN <B fields...> | MAP SET app <platform_name> type "post" context "url" url "https://google.com"
```
{% endcode %}
