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
