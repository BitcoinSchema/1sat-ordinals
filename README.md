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
This inscription script represents an inscription on an ordinal. The output value is 1 satoshi.

```bash
OP_FALSE OP_IF 6f7264 OP_1 <content-type> OP_0 <data> OP_ENDIF
```

A locking script (typically P2PKH) is then appended to the inscription script seperated by `OP_CODE_SEPERATOR`.

```bash
<inscription script> OP_CODE_SEPERATOR <Locking Script>
```

See the [Ordinals Docs](https://docs.ordinals.com/) for more information on ordinal theory.

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
To transfer ownership, simply send a 1sat output to the intended recipient as you normally would with any utxo.

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
1SatOrdinals is an implementation of the Ordinals Protocol for use on the BSV blockchain. It uses the same [inscription rules](https://docs.ordinals.com/inscriptions.html) as the founding implementation on BTC, with the following caveats/clarifications:

### Inscribing in Outputs
Due to the use of Tap Root in BTC, inscriptions are exposed in the input scripts. On BSV, they are written in outputs. 
Due to this difference, Inscription IDs in 1SatOrdinals are stated in relation to the output of a transaction, and take the form of `<txid hex>_<vout>`.

### PUSH DATA
On BSV, push data are NOT limited to 520 bytes and values should NOT be concatenated across multiple data pushes.

### Valid Inscriptions
An inscription is considered valid if an `ord` envelope is present inline in an output script, regardless of how many satoshis are present on the output.

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

### Field Aliases
Due to ambiguity of documentation and implementations of Bitcoin ASM, `OP_1`-`OP_16` are treated as alieses of the corisponding push values of `OP_DATA_1` followed by the value 1-16

### Repeated Fields
Parsing considerations for handling if a field is repeated is not defined in the Ordinals Spec. 1SatOrdinals treats repeated fields such that later values will overwrite previous values. 

### Why 1Sat?
If inscriptions are valid on any output of any size, why is it named
1SatOrdinals?

1SatOrdinals is a superset of the Ordinals Protocol and is 100% backward compatible.

We take a different approach to indexing due to the expanded capacity of the BSV blockchain. `origin` indexing is built on the idea that it ultimately doesn't matter WHICH specific ordinal is being transferred across the blockchain, as long as it can be easily determined that multiple transactions are referencing the SAME ordinal. 

`origin`s are tracked within the 1SatOrdinals indexer only as 1-satoshi outputs. This does not mean that if you inscribe on more than 1 satoshi, that the inscription is not a valid Ordinal. It simply means that the inscription will not have a 1sat `origin` and the 1SatOridinals indexer will not follow the transfers of your Ordinal.

### Origin vs Ordinal indexing



### Resources

* [Discord](https://discord.gg/XUfss6StD8)
* [BTC Ordinals Specification](https://docs.ordinals.com/)
