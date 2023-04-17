# CollectionItem SubType

When the top level type is `collectionItem` the `subTypeData` is defined here.

### subTypeData data

The following properties define the `subTypeData` object and should be used if your `collectionItem` ordinal has additional information that should be associated with it. A `collectionId` at the top level is required for `collectionItem` ordinals. Since a collection is an ordinal, all top level required fields are still required as well.&#x20;

| Name & Description                                                                                     | Required | Type                                          | Example                                                                                                      |
| ------------------------------------------------------------------------------------------------------ | -------- | --------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| <p><code>collectionId</code><br><br>A unique identifier, txid_vout of the collection subType.</p>      | Y        | `txid_vout`                                   | TODO: use a good example aaff22a9568dacfa6b90d64e31218b89bb5ef1ab3995e17540870fbf46bb990b\_0                 |
| <p><code>file</code><br><br>The offchain location of the file</p>                                      | N        | string                                        | https://pyxis.nymag.com/v1/imgs/bc9/5bb/95f88f06973066c75f07b98ed8af7f634a-18-pepe-the-frog.rsquare.w330.jpg |
| <p><code>fileType</code><br><br>The media type of the external file</p>                                | N        | string                                        | `image/png`                                                                                                  |
| <p><code>mintNumber</code><br><br>An integer, position the ordinal exists at within the collection</p> | N        | int                                           | 3                                                                                                            |
| <p><code>rank</code><br><br>A integer starting at 1 where 1 is the most 'rare'</p>                     | N        | int                                           | 10                                                                                                           |
| <p><code>rarityLabel</code><br><br>The overall rarity label for this ordinal</p>                       | N        | string enum based on `subTypeData`            | "legendary"                                                                                                  |
| <p><code>traits</code><br><br>Array of traits that describe the ordinal</p>                            | N        | traits as defined by collection `subTypeData` | see examples below                                                                                           |

## Traits

The definition of `trait` within the `traits` array:

| Name              | Description                                                   | Required | Type        |
| ----------------- | ------------------------------------------------------------- | -------- | ----------- |
| name              | The name of the trait                                         | Y        | string      |
| value             | The value of the trait                                        | Y        | string      |
| rarityLabel       | A rarity label to associate with the trait                    | N        | RarityLabel |
| occurrencePercent | The percentage which this trait occurs within this collection | N        | string      |

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
 "royalties": [
    {"type": "paymail", "destination": "jdoe@handcash.io", "percentage": "0.03"}, 
    {"type": "address", "destination": "1MvYhFajARJ82sbgxuAXziq1FmgSY1XQwD", "percentage": "0.025"}
  ]
}
```
