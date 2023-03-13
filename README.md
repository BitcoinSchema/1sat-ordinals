# 1 Sat Ordinals

![1Sat Ordinals](https://raw.githubusercontent.com/BitcoinSchema/1sat-ordinals/blob/master/ordinals.png?raw=true)

An ordinals protocol implementation on Bitcoin SV.

## Motivation

There are several major differences between chains like BTC where ordinals originated, and a classic Bitcoin system like BSV. Most notably, BSV does not have taproot or segwit, so does not need a workaround to store large data files.

Ordinals target a specific satoshi but in practice a range is typically used to satisfy dust limits on other chains. Since BSV has no dust limit, we can dramatically simplify things by truly inscribing a single satoshi.

This satoshi is the ordinal and can be sent to any wallet that recognizes it.

## Important note for wallet implementors

It is very importent for wallets to keep single satoshi unspent outputs seperate from the rest of your UTXOs. To help isolate ordinals and help prevent accidental loss, we separate funding addresses from destinations meant for holding 1sat ordinals.

Those familliar with the Run token protocol will recall a "purse" was used to fund jigs. We use the same termonology here. A purse funds the network fees for inscriptions and transfers, while a seperat wallet holds the 1sat ordinals as to not comingle them with funding sats.

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

### All Together

Here's a plain text "Hello world" inscription bringing it all together. This output script creates an ordinal and sends it to a recipient. The output value must be exactly 1 satoshi.

Output:

```bash
OP_DUP OP_HASH160 <PUBKEY> OP_EQUALVERIFY OP_CHECKSIG OP_FALSE OP_IF 6f7264 OP_1 <content-type> OP_0 <INSCRIPTION_DATA> OP_ENDIF
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
i1 - 1 sat
i2 - funding utxo
o1 - 1 sat + new inscription OP_RETURN AIP <sig...> -1
o3 - change
```

## More Information

- [Suplimental Readme](https://github.com/bitcoinschema/1sat-ordinals/blob/main/SUPPLIMENTAL.md)
- [Common Questions](https://github.com/bitcoinschema/1sat-ordinals/blob/main/FAQ.md)

## Resources

- [Ordinals Indexer](https://github.com/shruggr/bsv-ord-indexer)
