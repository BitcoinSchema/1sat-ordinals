# CollectionItem SubType

When the top level type is `collectionItem` the `subTypeData` is defined here.

### subTypeData data

The following properties define the `subTypeData` object and should be used if your `collectionItem` ordinal has additional information that should be associated with it. A `collectionId` at the top level is required for `collectionItem` ordinals. Since a collection is an ordinal, all top level required fields are still required as well.

<table><thead><tr><th>Name &#x26; Description</th><th width="118">Required</th><th>Type</th><th>Example</th></tr></thead><tbody><tr><td><code>collectionId</code><br><br>A unique identifier, txid_vout of the collection subType.</td><td>Y</td><td><code>txid_vout</code></td><td>TODO: use a good example <code>aaff22a9568dacfa6b90d64e31218b89bb5ef1ab3995e17540870fbf46bb990b_0</code><br><br>or for self: <code>_0</code></td></tr><tr><td><code>mintNumber</code><br><br>An integer, position the ordinal exists at within the collection</td><td>N</td><td>int</td><td>3</td></tr><tr><td><code>rank</code><br><br>A integer starting at 1 where 1 is the most 'rare'</td><td>N</td><td>int</td><td>10</td></tr><tr><td><code>rarityLabel</code><br><br>The overall rarity label for this ordinal</td><td>N</td><td>string enum based on <code>subTypeData</code></td><td>"legendary"</td></tr><tr><td><code>traits</code><br><br>Array of traits that describe the ordinal</td><td>N</td><td>traits as defined by collection <code>subTypeData</code></td><td>see examples below</td></tr><tr><td><code>attachments</code></td><td>N</td><td><code>Attachment[]</code></td><td>https://...<br>b://...<br>c://...</td></tr></tbody></table>



## Trait

The definition of `trait` within the `traits` array:

<table><thead><tr><th width="197">Name</th><th width="317">Description</th><th width="106">Required</th><th>Type</th></tr></thead><tbody><tr><td>name</td><td>The name of the trait</td><td>Y</td><td>string</td></tr><tr><td>value</td><td>The value of the trait</td><td>Y</td><td>string</td></tr><tr><td>rarityLabel</td><td>A rarity label to associate with the trait</td><td>N</td><td>RarityLabel</td></tr><tr><td>occurrencePercent</td><td>The percentage which this trait occurs within this collection</td><td>N</td><td>string</td></tr></tbody></table>

## Attachment

| Name         | Description                        | Required | Type   |
| ------------ | ---------------------------------- | -------- | ------ |
| name         | The name of the attachment         | Y        | string |
| description  | The description of the attachment  | N        | string |
| content-type | The content-type of the attachment | Y        | string |
| url          | The url of the attachment          | Y        | string |

## Transaction Structure

This pseudo-script creates an ordinal with metadata called "The Awesome Ordinal" with only the minimum required fields, and adds a signature via AIP so the issuer can be verified.

Output 1:

{% code overflow="wrap" %}
```
1SAT_P2PKH <INSCRIPTION> OP_RETURN MAP SET app <mint_platform> type ord name "The Awesome Ordinal" | AIP <address> "BITCOIN_ECDSA" <signature> [-1]
```
{% endcode %}

### Example `ord` type data

```json
{
 "name": "Pepe with Fire",
 "previewUrl": "https://somepreview.com/image.png",
 "royalties": [
    {"type": "paymail", "destination": "jdoe@handcash.io", "percentage": "0.03"}, 
    {"type": "address", "destination": "1MvYhFajARJ82sbgxuAXziq1FmgSY1XQwD", "percentage": "0.025"}
  ]
}
```

## Fungible (BSV-21) collection members

A `collectionItem` is not required to be an image inscription. A BSV-21
`deploy+mint` or `deploy+auth` output (content type `application/bsv-20`, see
[BSV-21](../fungible-tokens/bsv-21.md)) may itself carry `collectionItem`
metadata as a MAP suffix appended after its `bsv-20` JSON payload, exactly as
any other ordinal does. The collection membership lives entirely in MAP — the
`bsv-20` JSON is unchanged and carries no collection fields.

In this case:

- The **member is the token's deploy/genesis outpoint** (`tokenId`, i.e.
  `<txid>_0`), not any individual held or transferred unit. Metadata inherits to
  holders by resolving `tokenId` back to its deploy outpoint — the same way
  `sym`/`dec`/`icon` already inherit per the BSV-21 spec. One token type is one
  collection member. An application may treat a nonzero balance as ownership
  or access; BSV-21 itself proves the balance but does not enforce product rights.
- `mintNumber` and `rank` do not apply to a fungible member and should be
  omitted.
- `traits` / `rarityLabel`, if present, describe the token type as a whole
  rather than an edition or serial position.
- The token's own `icon` field (an inscription-origin or B-protocol-file
  outpoint) may serve as the item's preview image in place of a separate
  `previewUrl` / attachment.
- Canonical membership is `subTypeData.collectionId` plus an authorship
  signature whose signer matches the collection root. Use **SIGMA** — its prefix
  commits to a transaction input, so the signature cannot be replayed onto
  another item (legacy AIP-signed members remain valid). When `collectionId` is
  an absolute outpoint, the ordinal `parent` field (field 3) may duplicate it as
  a navigation hint; it is not required, and a same-transaction `_N` reference
  cannot use it.

### Supply model

The BSV-21 supply model the member uses is a product decision, not a collection
concern:

- **Fixed supply** (`deploy+mint`) — the entire, immutable supply is created at
  deploy. Fits capped editions: e.g. an album collection where each track is a
  member token with a set number of tradeable units, one unit granting the right
  to play that track.
- **Auth tokens** (`deploy+auth` then `mint`) — no initial supply; an auth UTXO
  grants ongoing minting. Fits access passes and any mint-over-time model. Only
  the holder of the auth UTXO can mint more, so a distributor keeps that key on
  the server and mints one unit per sale (mint-on-buy). The collection creator
  and the auth holder are typically the same key.
