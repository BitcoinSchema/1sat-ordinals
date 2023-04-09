# Public APIs

GorillaPool maintains a public `1sat-server` Basic usage is documented below. See the auto-generated [swagger documentation](https://ordinals.gorillapool.io/api/docs/) for a complete API reference.

The API is also [available on Github](https://github.com/shruggr/1sat-server).

### Other APIs

* There is also basic support from [#whats-on-chain](public-apis.md#whats-on-chain "mention")
* The [#bmap-api](public-apis.md#bmap-api "mention") can be used to resolve inscriptions with support for several other data protocols.&#x20;

## 1SAT Server Endpoints

### Get Files, Inscriptions

```
METHOD: GET
```

Get inscription file for a given origin.&#x20;

```
https://ordinals.gorillapool.io/api/files/inscriptions/:origin
```

Sample response

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

### Get Inscriptions

```
Method: GET
```

Get inscription data for a given origin.

```
https://ordinals.gorillapool.io/api/inscriptions/origin/:origin
```

Get inscriptions for a given txid.

```
https://ordinals.gorillapool.io/api/inscriptions/txid/:txid
```

Sample response

```json
[
    {
      // inscription number
      "id": 165, 
      // transaction id
      "txid": "e17d7856c375640427943395d2341b6ed75f73afc8b22bb3681987278978a584",
      // output index
      "vout": 1,
      // file info
      "file": {
        "hash": "3dbe16ec7625e0d8a02ceaa5b2b03bc412c06186d03fbd69090c162469cf0292",
        "size": 2592,
        "type": "image/png"
      },
      // 1sat output origin
      "origin": "e17d7856c375640427943395d2341b6ed75f73afc8b22bb3681987278978a584_1",
      // ordinal number
      "ordinal": 0,
      // block height
      "height": 783968,
      // block index
      "idx": 756,
      // hash of locking script preceding inscription
      "lock": "5d5c07f532ae28f90263209aeba5417366569c60947b74b363ce55c5d57d253d"
    }
]
```

### Get Ordinal Utxos

```
Method: GET
```

Because ordinals exist in Bitcoin Script, . Get unspent utxos with ordinal locks matching a given address.

```
https://ordinals.gorillapool.io/api/utxos/address/:address
```

Sample response

{% code overflow="wrap" %}
```json
[
    {
      "txid": "5af82e9c72688270afc9c70a3f523560600d1aa2c39eab74756d11243f4752ba",
      "vout": 0,
      "satoshis": 1,
      "lock": "b0a542a4a4707f7b5b48f4c7a45e12bee4f9481a5ad3db250dfd3730f5ff4225",
      "origin": "5af82e9c72688270afc9c70a3f523560600d1aa2c39eab74756d11243f4752ba_0",
      "ordinal": 0
    },
    {
      "txid": "7ecb6b642cf7e44cf757562fe88f8c51f0bb844e41c6c844eca2b13af8c49ca0",
      "vout": 0,
      "satoshis": 1,
      "lock": "b0a542a4a4707f7b5b48f4c7a45e12bee4f9481a5ad3db250dfd3730f5ff4225",
      "origin": "7ecb6b642cf7e44cf757562fe88f8c51f0bb844e41c6c844eca2b13af8c49ca0_0",
      "ordinal": 0
    }
]
```
{% endcode %}

You can also get inscriptions for UTXOs in a single request

```
https://ordinals.gorillapool.io/api/utxos/address/:address/inscriptions
```

## Get Lock Utxos

```
Method: GET
```

Get unspent outputs for a given "lock", which is the scripthash of the locking script up to the point of an inscription.

```
https://ordinals.gorillapool.io/api/utxos/lock/:lock
```

Returns UTXOs:

```json
[{
    "txid": string,
    "vout": number,
    "satoshis": number,
    "lock": string,
    "spend": string,
    "origin": string,
    "ordinal": number,
}]
```

## Address Events - SSE

SSE endpoints are available for real-time mempool tx notifications. Watch multiple addresses or locking script hashes simultaneously, and get notified when a transaction occurs matching. Add query param `address` for each address you want to monitor, and `lock` for each script hash.

You will receive both new Inscription UTXOs AND spends from the same messages. If the `spend` field is not `null`, that means you have received a spend, and you should remove the UTXO identified by `txid`:`vout` from your UTXO set.

If the spend field is `null`, then this is a new Inscription in your wallet.

{% code overflow="wrap" %}
```javascript
https://ordinals.gorillapool.io/api/subscribe?address=:address1&address=:address2&lock=:lock1&...
```
{% endcode %}

Listening for ordinal address activity:

```tsx
const API_HOST = "https://ordinals.gorillapool.io/api/"
const s = new EventSource(`${API_HOST}subscribe?address=${ordAddress}`);

s.onmessage = (e) => {
    console.log({ message: e })
}
// s.onopen
// s.onerror
```

## What's On Chain

WhatsOnChain.com also provides 1Sat Ordinals support by tagging the inscriptions and providing a plugin for rending ordinals by txid and outpout index as follows:

{% code overflow="wrap" %}
```
https://plugins.whatsonchain.com/api/plugin/main/:txid/:vout
```
{% endcode %}

## BMAP API

Get a BMAP Transaction object as a formatted JSON string

```
https://b.map.sv/tx/:txid
```

or non-formatted bmap

```
https://b.map.sv/tx/:txid/bmap
```

raw transaction hex

```
https://b.map.sv/tx/:txid/raw
```

or BOB format

```
https://b.map.sv/tx/:txid/bob
```

