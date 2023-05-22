# Ordinal Lock

{% embed url="https://github.com/shruggr/ordinal-lock" %}

{% code overflow="wrap" %}
```
WARNING: This is currently experimental, and not sufficiently tested. 
Use at your own risk.
```
{% endcode %}

You can lock an ordinal, creating an on-chain public listing for sale. If the listing value is transferred to the specified address, the ordinal is released to the destination given by the buyer. The listing can be canceled.

Below is the sCrypt contract. Provide a payment output, and a sellec pubkey and compile the contract.

```js
import {
    assert,
    ByteString,
    hash160,
    hash256,
    method,
    prop,
    PubKey,
    PubKeyHash,
    SmartContract,
    Sig,
    SigHash,
} from 'scrypt-ts'

export class OrdinalLock extends SmartContract {
    @prop()
    seller: PubKeyHash

    @prop()
    payOutput: ByteString

    constructor(seller: PubKeyHash, payOutput: ByteString) {
        super(...arguments)

        this.seller = seller;
        this.payOutput = payOutput;
    }

    @method(SigHash.ANYONECANPAY_ALL)
    public purchase(destOutput: ByteString, trailingOutputs: ByteString) {
        assert(hash256(destOutput + this.payOutput + trailingOutputs) == this.ctx.hashOutputs)
    }

    @method()
    public cancel(sig: Sig, pubkey: PubKey) {
        assert(this.seller == hash160(pubkey), 'bad seller')
        assert(this.checkSig(sig, pubkey), 'signature check failed')
    }
}
```

`destOutput` is the destination output for the utxo which is locked.

Once the script is compiled, use it to mint and list an ordinal in a single 1 sat output:

```bash
1Sat_ORDINAL_LOCK <INSCRIPTION>
```
