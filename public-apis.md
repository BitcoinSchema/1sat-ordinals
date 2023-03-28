# Public APIs

GorillaPool maintains a public `1sat-server`

See the auto-generated [swagger documentation](https://ordinals.gorillapool.io/api/docs/) for a complete API reference.

The API is also [available on Github](https://github.com/shruggr/1sat-server).

## Get Files, Inscriptions

```
METHOD: GET
```

Get inscription file for a given origin.&#x20;

```
https://ordinals.gorillapool.io/api/files/inscriptions/:origin
```

Sample response

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

## Get Inscriptions

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

## Get Ordinal Utxos

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

{% code overflow="wrap" %}
```
https://ordinals.gorillapool.io/api/subscribe?address=:address1&address=:address2&lock=:lock1&...
```
{% endcode %}
