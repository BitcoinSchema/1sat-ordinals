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

<table><thead><tr><th width="243">Description</th><th width="124">Required</th><th width="177">Type</th><th>Example</th></tr></thead><tbody><tr><td><code>app</code><br><br>The name of the app that originally produced the ordinal</td><td>Y</td><td>string</td><td>handcash</td></tr><tr><td><code>type</code><br><br>The type of MAP data. For this spec we use <code>ord</code></td><td>Y</td><td>string</td><td>ord</td></tr><tr><td><code>name</code><br><br>Name of the ordinal</td><td>Y</td><td>string</td><td>Joe Racoon</td></tr><tr><td><code>subType</code><br><br>The subType</td><td>N</td><td>string</td><td>collectionItem, collection, website</td></tr><tr><td><code>subTypeData</code><br><br>A stringified version of the data required by the specific subType specified. See subType documentation.</td><td>N<br>SubType: Y<br></td><td>stringified JSON</td><td>{ file: ..., fileType: ... }</td></tr><tr><td><code>royalties</code><br><br>Where creator royalties should be sent</td><td>N</td><td>stringified JSON array of <code>royalty</code></td><td>see definition below</td></tr><tr><td><code>previewUrl</code></td><td>N</td><td>string URL</td><td>http://so.me/prev.png<br>b://&#x3C;txid><em>&#x3C;idx></em><br><em>c://&#x3C;contentH</em>ash></td></tr><tr><td><code>...</code><br><br>You can add additional fields with MAP as needed.</td><td>N</td><td>any</td><td>see <a href="./#other-metadata">metadata examples</a></td></tr></tbody></table>

## Royalties

Royalties should be applied when a sale of an item occurs. The definition of `royalty` within the `royalties` array:

<table><thead><tr><th width="166">Name</th><th width="300">Description</th><th width="104">Required</th><th>Type</th></tr></thead><tbody><tr><td>type</td><td>Currently supports paymail and any valid script/address/paymail</td><td>Y</td><td><code>PaymentType</code></td></tr><tr><td>destination</td><td>The destination of the payment (the receivers address/paymail)</td><td>Y</td><td>string</td></tr><tr><td>percentage</td><td>The royalty percentage (3% would be <code>0.03</code>)</td><td>Y</td><td>string</td></tr></tbody></table>

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
