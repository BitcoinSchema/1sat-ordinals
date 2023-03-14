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

This output script represents an inscription on an ordinal. The output value is 1 satoshi.

```bash
OP_FALSE OP_IF 6f7264 OP_1 <content-type> OP_0 <data> OP_ENDIF
```

This script is then appended to the payment script (typically P2PKH).

```bash
<Locking Script> <Inscription Script>
```

The rest of the Ordinals protocol is identical as it relates to ordinal numbers. See the [Ordinals Docs](https://docs.ordinals.com/) for more information on ordinal theory.

### Creating an Inscription

Creating an inscription requires a single transaction. To summarize the transaction template:

```bash
Input  #1 - Any valid utxo
Output #1 - Locking Script w/ Inscription (1 Satoshi)
Output #2 - Change
```

The output with this script must lock exactly 1 Sat.

#### Locking Script

Typically, a P2PKH script is used to lock the ordinal. Simply send the 1 sat output to a new destination to transfer it. From here on, `1SAT_P2PKH` refers to a standard p2pkh output with a single sat value, but keep in mind, any locking script can be used.

```bash
OP_DUP OP_HASH160 <pubkeyhash> OP_EQUALVERIFY OP_CHECKSIG
```

#### Inscription Script

Next, inscribe a data file by filling in the two inscription fields, `data` and `content-type`. Append the inscription script to the locking script.

```bash
OP_FALSE OP_IF "ord" OP_1 <content-type> OP_0 <data> OP_ENDIF
```

### Transfers

To transfer ownership, simply send a 1sat output to the intended recipient as you normally would with any utxo.&#x20;

```
ii - funding utxo
o1 - 1sat_p2pkh
o2 - change
```

You can also append to the inscriptions on an ordinal by inscribing the same sat again.

```
i1 - previously inscribed ordinal
i2 - funding utxo
o1 - 1sat_p2pkh w/ second inscription
o3 - change
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

### Resources

* [Discord](https://discord.gg/XUfss6StD8)
* [BTC Ordinals Specification](https://docs.ordinals.com/)

### Maintainers

| [![Jad Wahab](https://github.com/jadwahab.png)](https://github.com/jadwahab) | [![Satchmo](https://github.com/rohenaz.png)](https://github.com/rohenaz) | [![Shruggr](https://github.com/shruggr.png)](https://github.com/shruggr) | [![Siggi](https://github.com/icellan.png)](https://github.com/icellan) |
| :--------------------------------------------------------------------------: | :----------------------------------------------------------------------: | :----------------------------------------------------------------------: | :--------------------------------------------------------------------: |
|                      [Jad](https://github.com/jadwahab)                      |                   [Satchmo](https://github.com/rohenaz)                  |                   [Shruggr](https://github.com/shruggr)                  |                   [Siggi](https://github.com/icellan)                  |
