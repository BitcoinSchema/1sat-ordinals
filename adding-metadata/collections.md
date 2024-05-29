---
description: A special subType to describe a collection of ordinals
---

# Collection SubType

The collection subType is an ordinal including an on-chain record describing a collection. Member ordinals reference the collection subType by `txid_vout` using the `collectionId` field.

A collection inscription should be created first before any ordinals that reference it. If you do not record the collection inscription before the member ordinals, they will not be considered valid members of the collection.

The collection record should be signed with AIP using a key that matches the signature on member ordinals.

## subType data

Since a collection is an ordinal, all top level required fields are still required. When defining a collection there will be no `collectionId` present in top level metadata.

<table><thead><tr><th width="178">Name</th><th width="188">Description</th><th width="67">Req.</th><th width="115">Type</th><th>Example</th></tr></thead><tbody><tr><td>description</td><td>A brief description of the collection.</td><td>Y</td><td>string</td><td></td></tr><tr><td>quantity</td><td>Number in collection</td><td>N</td><td>string</td><td></td></tr><tr><td>rarityLabels</td><td>valid rarity labels &#x26; occurrences</td><td>N</td><td>stringified JSON</td><td>see example below</td></tr><tr><td>traits</td><td>list of valid traits objects</td><td>N</td><td>stringified JSON Array</td><td>see example below</td></tr></tbody></table>

## Collection Inscription

The collection subType is also an ordinal whos inscription should be of type `image/*`

While other content types will be considered valid, it it strongly recommended to use an image type to ensure apps can reliably display a collection preview.

## Example Rarity

To be considered valid by the indexer, accumulated rarity occurrences must equal 100.

```json
{
  "common": "3.00",
  "rare": "97.00"
}
```

## Example Traits

To be considered valid by the indexer, each occurancePercentage array must sum to 100.

{% code overflow="wrap" %}
```json
{
  "background": {
    "values": [
      "night time",
      "day time"
    ],
    "occurancePercentages": [
      "95.00",
      "5.00"
    ]
  },
  "lips": {
    "values": [
      "red",
      "blue"
    ],
    "occurancePercentages": [
      "90.00",
      "10.00"
    ]
  }
}
```
{% endcode %}

## Signatures

Signatures are applied the same way as [top level rules](collections.md#signatures). This allows collections to be verified.

## Transaction Structure

This pseudo-script screates a collection called "The Awesome Collection" with only the minimum required fields, and adds a signature via AIP so the issuer can be verified.

Output 1:

{% code overflow="wrap" %}
```
1SAT_P2PKH <INSCRIPTION> OP_RETURN MAP SET app <mint_platform> type ord subType collection name "The Awesome Collection"  description "57 wonderul things" | AIP <address> "BITCOIN_ECDSA" <signature> [-1]
```
{% endcode %}
