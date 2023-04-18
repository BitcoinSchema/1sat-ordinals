# Ord Schema Type

## The "ord" schema type

The base schema type for ordinals metadata is `ord`. This base type can be extended with the `subType` property so that wallets and markets will understand how to interpret and display metadata.

\
While subtypes can be created at will, a list of common subTypes is maintained at [https://bitcoinschema.org](https://bitcoinschema.org).

## Required Fields:

`app` - The name of the app that originally produced the ordinal

`type` - This should always be "ord". This helps indexers find the metadata based on type.

`name` - Name description of the ordinal.

## 1Sat Ordinals - Metadata Proposal

The following outlines a standard set of metadata properties that should be utilized by creators and implemented by developers when adding metadata to 1Sat Ordinals. Adhering to these specifications when creating and displaying content across applications will make for a better and more cohesive cross-platform user experience.

### Top-Level Metadata

Added to the inscription output using MAP protocol in OP\_RETURN.

Please note that the MAP protocol expects all data types for the value in the key value pair to be of type `string`.

| Description                                                                                                                                     | Required                   | Type                                | Example                                                                                               |
| ----------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------- | ----------------------------------- | ----------------------------------------------------------------------------------------------------- |
| <p><code>app</code><br><br>The name of the app that originally produced the ordinal</p>                                                         | Y                          | string                              | handcash                                                                                              |
| <p><code>type</code><br><br>The type of MAP data. For this spec we use <code>ord</code></p>                                                     | Y                          | string                              | ord                                                                                                   |
| <p><code>name</code><br><br>Name of the ordinal</p>                                                                                             | Y                          | string                              | Joe Racoon                                                                                            |
| <p><code>subType</code><br><br>The subType</p>                                                                                                  | N                          | string                              | collectionItem, collection, website                                                                   |
| <p><code>subTypeData</code><br><br>A stringified version of the data required by the specific subType specified. See subType documentation.</p> | <p>N<br>SubType: Y<br></p> | stringified JSON                    | { file: ..., fileType: ... }                                                                          |
| <p><code>royalties</code><br><br>Where creator royalties should be sent</p>                                                                     | N                          | stringified JSON array of `royalty` | see definition below                                                                                  |
| `previewUrl`                                                                                                                                    | N                          | string URL                          | <p>http://so.me/prev.png<br>b://&#x3C;txid><em>&#x3C;idx></em><br><em>c://&#x3C;contentH</em>ash></p> |
| <p><code>...</code><br><br>You can add additional fields with MAP as needed.</p>                                                                | N                          | any                                 | see [metadata examples](./#other-metadata)                                                            |

## Royalties

Royalties should be applied when a sale of an item occurs. The definition of `royalty` within the `royalties` array:

| Name        | Description                                                     | Required | Type          |
| ----------- | --------------------------------------------------------------- | -------- | ------------- |
| type        | Currently supports paymail and any valid script/address/paymail | Y        | `PaymentType` |
| destination | The destination of the payment (the receivers address/paymail)  | Y        | string        |
| percentage  | The royalty percentage (3% would be `0.03`)                     | Y        | string        |

## Payment Type

The type definition for `PaymentType`:

```
type PaymentType = 'paymail' | 'address' | 'script';
```

## Transaction Structure

This pseudo-script creates an ordinal with metadata called "The Awesome Ordinal" with only the minimum required fields, and adds a signature via AIP so the issuer can be verified.

Output 1:

{% code overflow="wrap" %}
```
1SAT_P2PKH <INSCRIPTION> OP_RETURN MAP SET app <mint_platform> type ord name "The Awesome Ordinal" | AIP <address> "BITCOIN_ECDSA" <signature> [-1]
```
{% endcode %}

### Example `ord`type

<pre class="language-json"><code class="lang-json">{
<strong>    "app": "take_it",
</strong>    "type": "ord",
    "name": "Awesome ordinal",
    "royalties": [
        {"type": "paymail", "destination": "jdoe@handcash.io", "percentage": "0.03"}, 
        {"type": "address", "destination": "1MvYhFajARJ82sbgxuAXziq1FmgSY1XQwD", "percentage": "0.025"}
    ]
}
</code></pre>
