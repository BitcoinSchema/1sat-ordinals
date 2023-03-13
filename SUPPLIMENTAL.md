## Handling Large Files - BCAT

If you were to inscribe files larger than 10MB (at the time of writing this) the ransaction would generally not be accepted by miners. To work around this, a large data file can be spread across multiple transactions. We can achieve this using the BCAT protocol. BCAT lists, in order, the transaction ids required to assemble the data file. It is a Bitcom protocol that uses the prefix "15DHFxWZJT58f9nhyGnsRBqrgwK4W6h4Up". You can find more information about BCAT protocol [here](https://bcat.bico.media/).

While you cannot add the bcat data directly into the inscription fields (since ordinals protocol does not support concatenating external transactions), you can inscribe a thumbnail as the ordinal image, and attach the video using BCAT in OP_RETURN as follows.

```bash
1SAT_P2PKH <INSCRIPTION> OP_RETURN <BCAT fields...> tx2 tx3 tx4...
```

## Adding Metadata

You can use metadata to add context to ordinals. For example, you can tag an ordinal with a geohash to place it on a physical location. Metadata is written using "Magic Attribute Protocol" which has a prefix of "1PuQa7K62MiKCtssSLKy1kh56WWU7MtUR5".

```bash
1SAT_P2PKH <INSCRIPTION> OP_RETURN <B fields...> | MAP SET app <platform_name> type "post" context "geohash" geohash "dhmgdqvr7"
```

Similarly, you can attach an ordinal to a particular url:

```bash
1SAT_P2PKH <INSCRIPTION> OP_RETURN <B fields...> | MAP SET app <platform_name> type "post" context "url" url "https://google.com"
```

## Ordinal Lock

You can lock an ordinal, creating an on-chain public listing for sale. If the listing value is transferred to the specified address, the ordinal is released to the destination given by the buyer. The listing can be canceled.

Below is the sCrypt contract. Provide a payment output, and a sellec pubkey and compile the contract.

```js
export class OrderLock extends SmartContract {
    @prop()
    seller: PubKey

    @prop()
    payOutput: ByteString

    constructor(seller: PubKey, payOutput: ByteString) {
        super(...arguments)

        this.seller = seller;
        this.payOutput = payOutput;
    }

    @method()
    public purchase(selfOutput: ByteString, trailingOutputs: ByteString) {
        assert(hash256(selfOutput + this.payOutput + trailingOutputs) == this.ctx.hashOutputs)
    }

    @method()
    public cancel(sig: Sig) {
        assert(this.checkSig(sig, this.seller), 'signature check failed')
    }
}
```

Once the script is compiled, use it to mint and list an ordinal in a single 1 sat output:

```bash
1Sat_ORDINAL_LOCK <INSCRIPTION>
```

## Signing

It is possible to sign ordinals to link an identity to the creation of the token. Separating identity signatures from funding signatures is more flexible. This is useful for verified minting. Signatures use "Author Identity Protocol" which has a Bitcom prefix of "15PciHG22SNLQJXMoSUaWVi7WSqc7hCfva".

Since AIP was designed to sign OP_RETURN based protocols, we need to use a special indices selector to sign the entire transaction, including the ordinal data. To do this, pass `[-1]` in the indices field to indicate the entire output script is being signed.

### User Signatures

To sign when inscribing:

```bash
1SAT_P2PKH <INSCRIPTION> OP_RETURN <AIP> <signing_address> "BITCOIN_ECDSA" <sig> -1
```

### Platform Signatures

You can not only use AIP signatures to prove a particular identity created an inscription, but you can sign once more to prove that a particular platform was used to create the transaction as well.

```bash
1SAT_P2PKH <INSCRIPTION> OP_RETURN AIP <user_address> <user_sig> -1 | AIP <platform_address> <platform_sig> -1
```

### Inscription Collection

You can also mint lots of ordinals at once, and link them to a specific collection.

```
o1 - 1SAT_P2PKH <INSCRIPTION> OP_RETURN MAP SET app <mint_platform> type ord collection <unique-collection-name>
o2 - 1SAT_P2PKH <INSCRIPTION> OP_RETURN MAP SET app <mint_platform> type ord collection <unique-collection-name>
```

## ATTACHING TO REAL WORLD LOCATION

Create with initial location

```
1SAT_P2PKH <INSCRIPTION> OP_RETURN MAP app <appname> type "ordinal" context geohash geohash <geohash> | AIP <sig...>
```

Spend the ordinal with MAP geohash - updates are checked by indexer

```
i1 - 1 sat o1 from inscription tx
o1 - 1SAT_P2PKH OP_RETURN MAP geohash <geohash> | AIP <sig...>
```

### Web Pages

To inscribe a webpage, simply:

```bash
1SAT_P2PKH OP_IF "ord" OP_1 "text/html;charset=utf8" OP_0 <html_data> OP_ENDIF
```

### o:// - Ordinal URL References

You can embed references to other ordinals inside the on-chain HTML or markdown.

An ordinalID is the precise satoshi identifier.

```html
<img src="o://ordinalID" />
```

Or in markdown

```markdown
[My Ordinal!]("o://ordinalID")
```

You can also address the specific inscription by txid and output index.

```html
<img src="b://txid{64}.o2" />
```

Complex example: To inscribe a video that exists across multiple transactions and place it
on a specific geohash with identity signature

```
tx1 - o1 - 1SAT_P2PKH OP_RETURN BCAT tx2 tx3 tx4... | MAP SET context geohash geohash <geohash> | AIP <sig...> -1

tx2 - o1 - BCAT_PART <tx2_data>
```

## Needs

- [ ] API Endpoint - Get ordinals by address (similar to endpoint for p2pkh utxos)
- [ ] Paymail - Need a capability for 1sat Ordinals
- [ ] BUX Wallet Support
