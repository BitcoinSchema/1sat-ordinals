<h1 align="center">1Sat Ordinals<h1>
<p align="center">
<img src="https://github.com/BitcoinSchema/1sat-ordinals/blob/main/ordinals.png?raw=true" alt="1Sat Ordinals" />
</p>
<h5 align="center">An ordinals protocol implementation on Bitcoin SV.<h5>

```
DRAFT SPECIFICATION
```

### Protocol

This output script creates an inscription on an ordinal. The output value should be 1 satoshi.

```bash
OP_FALSE OP_IF 0x6f7264 OP_1 <content-type> OP_0 <data> OP_ENDIF
```

This script is then appended to the payment script (typically P2PKH).

```bash
<P2PKH or Script> <Inscription Script>
```

## Ordinals on BSV

There are several major differences between chains like BTC where ordinals originated, and a classic Bitcoin system like BSV. Most notably, BSV does not have taproot or segwit, so it does not need a workaround to store data on-chain. We can simply put the ordinals script in an output, creating an inscribed ordinal in a single step.

Ordinals target a specific satoshi but in practice a range is typically used to satisfy dust limits on other chains. Since BSV has no dust limit, we can dramatically simplify things by truly inscribing a single satoshi. This has the additional benefit of improved scalability, since only 1sat utxos need to be considered.

# Inscriptions

## Creating an Inscription

Creating an inscription is much easier than on other blockchains. To summarize the transaction template:

```bash
Input - Any valid utxo
Output #1 - P2PKH + INSCRIPTION (1 Satoshi)
```

The output with this script must lock exactly 1 Sat.

### 1SAT P2PKH

Spending a 1sat ordinal is as simple as sending the 1 sat output to another address. From here on, `1SAT_P2PKH` refers to a standard p2pkh output with a single sat value.

```bash
OP_DUP OP_HASH160 <pubkeyhash> OP_EQUALVERIFY OP_CHECKSIG
```

### Inscription

Next, inscribe a data file by filling in the two inscription fields, `data` and `content-type`.

```bash
1SAT_P2PKH OP_FALSE OP_IF "ord" OP_1 <content-type> OP_0 <data> OP_ENDIF
```

## Examples

In this example, we inscribe a 3d model (GLTF binary) and tag it with a geolocation:

Mint & Inscribe: (1 sat P2PKH + inscription)

```
https://whatsonchain.com/tx/10f4465cd18c39fbc7aa4089268e57fc719bf19c8c24f2e09156f4a89a2809d6
```

Transfer:

```
https://whatsonchain.com/tx/61fd6e240610a9e9e071c34fc87569ef871760ea1492fe1225d668de4d76407e
```

## TRANSFERS

You can either just send the sat or append to the inscriptions on an ordinal

```
i1 - ordinal
i2 - funding utxo
o1 - 1Sat_p2pkh + additional inscription
o3 - change
```

## More Information

- [Suplimental Readme](https://github.com/bitcoinschema/1sat-ordinals/blob/main/SUPPLIMENTAL.md)
- [Common Questions](https://github.com/bitcoinschema/1sat-ordinals/blob/main/FAQ.md)

## Resources

- [Ordinals Indexer](https://github.com/shruggr/bsv-ord-indexer)
- [Go Library - go-1sat-ord](https://github.com/bitcoinschema/go-1sat-ord)
- [Tx Parser - go-bmap](https://github.com/bitcoinschema/go-bmap)
- [BTC Ordinals Specification](https://docs.ordinals.com/)
