# Public APIs

GorillaPool maintains a public `1sat-server`

## Get Inscriptions

Get inscription data for a given outpoint. An outpoint is a transaction ID and output index with the following formatting: `` `txid_vout` ``

```
https://ordinals.gorillapool.io/api/inscriptions/:outpoint
```

Sample response

```
[
    {
      "id": 165,
      "txid": "e17d7856c375640427943395d2341b6ed75f73afc8b22bb3681987278978a584",
      "vout": 1,
      "file": {
        "hash": "3dbe16ec7625e0d8a02ceaa5b2b03bc412c06186d03fbd69090c162469cf0292",
        "size": 2592,
        "type": "image/png"
      },
      "origin": "e17d7856c375640427943395d2341b6ed75f73afc8b22bb3681987278978a584_1",
      "ordinal": 0,
      "height": 783968,
      "idx": 756,
      "lock": "95EA9JV8O+RWtoU7zrUdcsIRyF2RhhHON/CxiBEpw3Y="
    }
]
```

## Get Ordinal Utxos

Get unspent artifacts for an address.

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
      "acc_sats": 1,
      "lock": "b0a542a4a4707f7b5b48f4c7a45e12bee4f9481a5ad3db250dfd3730f5ff4225",
      "origin": "5af82e9c72688270afc9c70a3f523560600d1aa2c39eab74756d11243f4752ba_0",
      "ordinal": 0
    },
    {
      "txid": "7ecb6b642cf7e44cf757562fe88f8c51f0bb844e41c6c844eca2b13af8c49ca0",
      "vout": 0,
      "satoshis": 1,
      "acc_sats": 1,
      "lock": "b0a542a4a4707f7b5b48f4c7a45e12bee4f9481a5ad3db250dfd3730f5ff4225",
      "origin": "7ecb6b642cf7e44cf757562fe88f8c51f0bb844e41c6c844eca2b13af8c49ca0_0",
      "ordinal": 0
    }
]
```
{% endcode %}
