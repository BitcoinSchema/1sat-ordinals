# Ordinal Lock

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
