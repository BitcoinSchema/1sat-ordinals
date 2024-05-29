---
description: extensible fungible token specification
---

# BSV-21

{% hint style="info" %}
BSV-21 was originally called BSV-20 v2 but was renamed for clarity. This is why the "p" field is the same for both token types.
{% endhint %}

This iteration to the BSV-20 protocol introduces a new `tickerless` mode functionality. `Tickerless` mode forgoes the `first is first` nature of BRC20-BTC, and allows the capabilities to have a smart contract, or an administrator, control distribution. Additionally, every transaction of a `tickerless` mode token, forms part of a single on-chain DAG (Directed Acyclic Graph), such that the transaction can easily be tracked back to that token's genesis `mint`.

In `first is first` mode, tokens are deployed and minted in separate transactions, with the only correlation between them being the token's ticker. Additionally, the ticker may be deployed multiple times, and may have more tokens minted than are defined in the token deployment. The only way to determine if any token UTXO is valid is to scan the entire blockchain and and perform complex analysis. `Tickerless` mode takes a different approach, where a token is deployed, and it's entire supply is minted in a single transaction output. This output can be locked with any logic which can be implemented in Bitcoin script. It can be as simple as a standard adminstrator address (P2PKH script), Proof-of-Work distribution, or even an entire Proof-of-Stake blockchain running on top of Bitcoin SV.

In order to deploy/mint a token, you create a JSON ordinal inscription output with the data fields below and content type of `application/bsv-20`.

### Deploy+Mint

| Key  | Required? | Description                                                                                                           |
| ---- | --------- | --------------------------------------------------------------------------------------------------------------------- |
| p    | Yes       | Protocol: `bsv-20`                                                                                                    |
| op   | Yes       | Operation: `deploy+mint`                                                                                              |
| sym  | No        | Token symbol. This should be unique, but uniqueness is not enforced.                                                  |
| icon | No        | Token icon. This value should contain the origin of another inscription OR the outpoint of a `B` protocol file upload |
| amt  | Yes       | Supply of token. Max 2^64-1                                                                                           |
| dec  | No        | Decimals: set decimal precision, defaults to 0. This is different from BRC20 which defaults to 18                     |

_Example_: To deploy and mint a token, you would create an inscription with the following json (ContentType: application/bsv-20):

```
{ 
  "p": "bsv-20",
  "op": "deploy+mint",
  "amt": "21000000",
  "dec": "10"
}
```

Unlike `first is first` mode of v1, no `tick` field is defined. A token is identified by an `id` field, which is the transaction id and output index where the token was minted, in the form of `<txid>_<vout>`.

### Transfer

Tokens in BSV-20 are held in UTXOs, similar to native bitcoins. This is different from BRC20, which holds balance in an account model. In order to transfer tokens, you spend that specific UTXO and create new outputs the same way you spend regular Satoshis, but the output(s) must contain `transfer` inscriptions.

If more tokens are transferred in the output(s) than are available in the input(s) then the transaction is considered invalid and the tokens are burned. If less tokens are created in the outputs than are available in the intput(s), the unallocated tokens are burned.

Using the same procedure as regular Satoshi transfers allows us to benefit from the parallelisation Bitcoin benefits from, where you can split a specific UTXO with a large amount into smaller UTXOs and spend those in parallel (the same way you could exchange a $100 bill into $1 bills and spend those in parallel) with no sequential bottlenecks that something like ERC20 (Ethereum) suffers from.

| Key | Required? | Description                                  |
| --- | --------- | -------------------------------------------- |
| p   | Yes       | Protocol: `bsv-20`                           |
| op  | Yes       | Operation: `transfer`                        |
| id  | Yes\*     | `<txid>_<vout>` of mint output               |
| amt | Yes       | Amount of tokens transferred in this output. |

#### `Tickerless` Example

```
{ 
  "p": "bsv-20",
  "op": "transfer",
  "id": "3b313338fa0555aebeaf91d8db1ffebd74773c67c8ad5181ff3d3f51e21e0000_1"
  "amt": "10000",
}
```

**Inputs**

1. previous transfer output holding 500 tokens
2. previous transfer output holding 500 tokens

**Outputs**

P2PKH inscription

* {"p":"bsv-20","op":"transfer","id":"3b313338fa0555aebeaf91d8db1ffebd74773c67c8ad5181ff3d3f51e21e0000\_1","amt":"100"}

P2PKH inscription

* {"p":"bsv-20","op":"transfer","id":"3b313338fa0555aebeaf91d8db1ffebd74773c67c8ad5181ff3d3f51e21e0000\_1","amt":"500"}

P2PKH inscription

* {"p":"bsv-20","op":"transfer","id":"3b313338fa0555aebeaf91d8db1ffebd74773c67c8ad5181ff3d3f51e21e0000\_1","amt":"400"}
