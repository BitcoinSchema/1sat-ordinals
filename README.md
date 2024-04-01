---
description: Ordinals-compatible, built on Bitcoin SV
---

# Protocol Specification

```
Protocol:          1Sat Ordinals
Status:            DRAFT
Documentation:     https://docs.1satordinals.com
```

### Overview
1SatOrdinals in an implementation of Ordinals running on the BSV blockchain.

See the [Ordinals Docs](https://docs.ordinals.com/) for more information on Ordinals and ordinal theory.

This inscription script represents an inscription on an ordinal. The output value is 1 satoshi.

```bash
OP_FALSE OP_IF 6f7264 OP_1 <content-type> OP_0 <data> OP_ENDIF
```

A locking script (typically P2PKH) is then prepended/appended to the inscription script, optionally seperated with OP_CODESEPERATOR.

```bash
// Recommended
<inscription script> OP_CODESEPERATOR <locking script>

// Note: OP_CODESEPERATOR is not strictly required for an inscription to be valid, but should be used with P2PKH for future compatiblity across wallets.
<locking script> <inscription script>
```

### Creating an Inscription
Creating an inscription requires a single transaction. To summarize the transaction template:

```bash
Input  #1 - Any valid utxo
Output #1 - Inscription w/ Locking Script (1 Satoshi)
Output #2 - Change
```

The output with this script should lock exactly 1 Sat.

#### `ord` Envelope
Inscribe a data file by filling in the two inscription fields, `data` and `content-type`. 

```bash
OP_FALSE OP_IF "ord" OP_1 <content-type> OP_0 <data> OP_ENDIF 
```

#### Locking Script
Typically, a P2PKH script is used to lock the ordinal. Simply send the 1 sat output to a new destination to transfer it. From here on, `1SAT_P2PKH` refers to a standard p2pkh output with a single sat value, but keep in mind, any locking script can be used.

```bash
OP_DUP OP_HASH160 <pubkeyhash> OP_EQUALVERIFY OP_CHECKSIG
```

### Transfers
To transfer ownership, simply send the 1sat output to the intended recipient as you normally would with any utxo while maintaining ordinality of the satoshi being transferred. (i.e. the `n`th satoshi input to the transaction is transferred to the `n`th satoshi output of the transaction)

```
i1 - 1sat_p2pkh
i2 - funding utxo
o1 - 1sat_p2pkh
o2 - change
```

You can also append to the inscriptions on an ordinal by inscribing the same sat again.

```
i1 - previously inscribed ordinal
i2 - funding utxo
o1 - second inscription w/ 1sat_p2pkh
o2 - change
```

### Examples
In this example, we inscribe a 3d model (GLTF binary) and tag it with a geolocation:

Mint & Inscribe: (1SAT\_P2PKH + inscription)

```
https://whatsonchain.com/tx/10f4465cd18c39fbc7aa4089268e57fc719bf19c8c24f2e09156f4a89a2809d6
```

Transfer:

```
https://whatsonchain.com/tx/61fd6e240610a9e9e071c34fc87569ef871760ea1492fe1225d668de4d76407e
```


## Ordinals vs 1SatOrdinals
The BSV blockchain is unique among blockchains which support ordinals, in that BSV supports single satoshi outputs. This allows us to take some short-cuts in indexing efficiently. We call this `origin`-based indexing.

Since ordinals are a unique serial number for each satoshi, an `origin` can be defined as the first outpoint where a satoshi exists alone, in a one satoshi output. Each subsequent spend of that satoshi can be crawled back to the first ancestor where an output contains more than one satoshi.

We define an 1SatOrdinal as a chain of single satoshi output spends. Each owner transfers 1sat by creating transaction that has single satoshi output in a position determined by ordinals theory.  Payee can verify these transfers math to verify the chain of
ownership.

If a satoshi is subsequently packaged up in an output of more than one satoshi, the origin is no longer carried forward and the token can be considered burned. If the satoshi is later spent into another one satoshi output, a new origin will be created. Both of these origins would be the same ordinal, but are distinct tokens in 1SatOrdinals.

1SatOrdinals uses the same [inscription rules](https://docs.ordinals.com/inscriptions.html) as the founding implementation on BTC, with the following caveats/clarifications:

### Inscribing in Outputs
Due to the use of Tap Root in BTC, inscriptions are exposed in the input scripts. On BSV, they are written in outputs. 
Due to this difference, Inscription IDs in 1SatOrdinals are stated in relation to the output of a transaction, and take the form of `<txid hex>_<vout>`.

Only the first valid inscription envelope produces a 1SatOrdinal. Any subsequent inscriptions MUST be ignored.


### 1 Satoshi Outputs
1SatOrinals requires inscrptions to be made on a single satoshi output.

### PUSH DATA
On BSV, push data are NOT limited to 520 bytes and values should NOT be concatenated across multiple data pushes.

### Valid Inscriptions
An inscription is considered valid if an `ord` envelope is present inline in an output script on a 1 satoshi output.

### ord Envelope Format
```
OP_FALSE OP_IF 6f7264
    <field1> <value1>
    ...
    <fieldN> <valueN>
    OP_0 <content> 
OP_ENDIF
```
- `field` and `value` MUST alway appear as pairs
- `field` MUST be a single PUSH_DATA or `OP_1`-`OP_16`
- `value` MUST be a single PUSH_DATA, `OP_0`, or `OP_1`-`OP_16`

See the [Ordinals Docs](https://docs.ordinals.com/inscriptions.html) for more information on Ordinals fields.

### Field Aliases
Due to ambiguity of documentation and implementations of Bitcoin ASM, `OP_1`-`OP_16` are treated as alieses of the corisponding push values of `OP_DATA_1` followed by the value 1-16

### Repeated Fields
Parsing considerations for handling if a field is repeated is not defined in the Ordinals Spec. 1SatOrdinals treats repeated fields such that later values will overwrite previous values. 

### Origin
1SatOrdinals is a superset of the Ordinals Protocol and is 100% backward compatible.

We take a different approach to indexing due to the expanded capacity of the BSV blockchain. `origin` indexing is built on the idea that it ultimately doesn't matter WHICH specific ordinal is being transferred across the blockchain, as long as it can be easily determined that multiple transactions are referencing the SAME ordinal. 

`origin`s are tracked within the 1SatOrdinals indexer only as 1-satoshi outputs. If you inscribe on more than 1 satoshi, that the inscription is not a valid 1SatOrdinal.


### Resources

* [Discord](https://discord.gg/XUfss6StD8)
* [BTC Ordinals Specification](https://docs.ordinals.com/)
