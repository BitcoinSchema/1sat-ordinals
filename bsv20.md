# BSV-20 v2

## Abstract

This proposal introduces a fungible token standard based off of the BRC20[1](./#footnote-1) standard on BTC introduced by Domo [here](https://domo-2.gitbook.io/brc-20-experiment/) but customised to work on BSV (Bitcoin Satoshi Vision). In order to avoid cofusion with the BRC-20 standard on BTC, we are calling this standard BSV-20.

## Motivation

The purpose of this proposal is to provide a standard that offers the same functionality that BRC20-BTC offers that works on BSV instead of BTC. A guiding motivation behind this standard proposal is to try and keep the standard as simple as possible.

## Specification

This specification is meant be as similar to the BTC BRC20 standard so it also follows the `first is first` approach. In order to deploy or mint a token, the process is almost identical to the protocol on BTC: you would create a transaction output with the data fields below and tranasction are indexed on a 'first is first' approach with duplicates or overflows being invalid and ignored. The main difference between bsv-20 (this protocol) and bsv-20 protocol (on BTC) is:

* A content type of `application/bsv-20` is used in place of `text/plain`. This change means a bsv-20 indexer does not need to parse every text inscription to test for embedded JSON content (and failing most of the time) in order to determine if an inscription is BSV-20 related.

## New in V2 - Tickerless Mode

`First is fist` deployment and minting allows for a single use-case where a token is publicly mintable, by anyone, outside of the control of any token issuer. V2 of the BSV-20 protocol introduces a new `tickerless` mode functionality. `Tickerless` mode forgoes the `first is first` nature of BRC20-BTC, and allows the capibilities to have a smart contract, or an adminstrator, control distribution. Additionally, every transaction of a `tickerless` mode token, forms part of a single on-chain DAG (Directed Acyclic Graph), such that the transaction can easily be tracked back to that token's genesis `mint`.

In `first is first` mode, tokens are deployed and minted in separate transactions, with the only correlation between them being the token's ticker. Additionally, the ticker may be deployed multiple times, and may have more tokens minted than are defined in the token deployment. The only way to determine if any token UTXO is valid is to scan the entire blockchain and and perform complex analysis. `Tickerless` mode takes a different approach, where a token is deployed, and it's entire supply is minted in a single transaction output. This output can be locked with any logic which can be implemented in Bitcoin script. It can be as simple as a standard adminstrator address (P2PKH script), Proof-of-Work distribution, or even an entire Proof-of-Stake blockchain running on top of Bitcoin SV.

In order to deploy/mint a token, you create a JSON ordinal inscription output with the data fields below and content type of `application/bsv-20`.

### V2 - Deploy+Mint (Tickerless mode)

| Key | Required? | Description |
| --- | --------- | ----------- |
| p | Yes | Protocol: `bsv-20` |
| op | Yes | Operation: `deploy+mint` |
| amt | Yes | Supply of token. Max 2^64-1 |
| dec | No | Decimals: set decimal precision, defaults to 0. This is different from BRC20 which defaults to 18 |


*Example*:
To deploy and mint a token, you would create an inscription with the following json (ContentType: application/bsv-20):
```
{ 
  "p": "bsv-20",
  "op": "deploy+mint",
  "amt": "21000000",
  "dec": "10"
}
```

Unlike `first is first` mode of v1, no `tick` field is defined. A token is identified by an `id` field, which is the transaction id and output index where the token was minted, in the form of `<txid>_<vout>`.

### V1 - Deploy (`First is first` mode only)

#### Notes

* The first deployment of a ticker is the only one that has claim to the ticker. Tickers are not case sensitive (DOGE = doge)
* If two events occur in the same block, prioritization is assigned via order they were confirmed in the block. (first to last)
* The first mint to exceed the maximum supply will receive the fraction that is valid. (ex. 21,000,000 maximum supply, 20,999,242 circulating supply, and 1000 mint inscription = 758 balance state applied)
* Number of decimals cannot exceed 18 (default)
* Maximum supply cannot exceed uint64\_max

In order to deploy an BSV-20 `first is first` token, you must make sure that that token ticker has not already been deployed and then create a TXO with the following data present in the script. It does not matter whether this UTXO is spendable or not.

| Key  | Required? | Description                                                                                                  |
| ---- | --------- | ------------------------------------------------------------------------------------------------------------ |
| p    | Yes       | Protocol: `bsv-20`
| op   | Yes       | Operation: `deploy`                                                            |
| tick | Yes       | Ticker: 4 letter identifier of the bsv-20 |
| max  | Yes       | Max supply: set max supply of the bsv-20 |
| lim  | No        | Mint limit: If letting users mint to themselves, limit per ordinal. If ommitted or 0, mint amt us unlimited. |
| dec  | No        | Decimals: set decimal precision, default to 0 |

#### Example

To deploy the `ordi` token, you would create an inscription with the following json (with `ContentType: application/bsv-20`):

```json
{ 
  "p": "bsv-20",
  "op": "deploy",
  "tick": "ordi",
  "max": "21000000",
  "lim": "1000"
}
```

### V1 - Mint (`First is first` mode)

In order to mint tokens of a specific BSV-20 token, you must make sure that that token ticker has already been deployed and then create a UTXO with 1 satoshi value as well as with the following data present in the script. This UTXO should be spendable in order for you to be able to transfer these minted tokens.

| Key  | Required? | Description                                                                                        |
| ---- | --------- | -------------------------------------------------------------------------------------------------- |
| p    | Yes       | Protocol: `bsv-20` |
| op   | Yes       | Operation:  `mint` |
| tick | Yes       | Ticker: 4 letter identifier of the bsv-20 |
| amt  | Yes       | Amount to mint: States the amount of the bsv-20 to mint. Has to be less than "lim" above if stated |

#### Example

To mint `ordi` tokens, you would create an inscription with the following json (with `ContentType: application/bsv-20`):

```json
{ 
  "p": "bsv-20",
  "op": "mint",
  "tick": "ordi",
  "amt": "1000"
}
```

### Transfer (All modes)

Tokens in BSV-20 are held in UTXOs, similar to native bitcoins. This is different from BRC20, which holds balance in an account model. In order to transfer tokens, you spend that specific UTXO and create new outputs the same way you spend regular Satoshis, but the output(s) must contain `transfer` inscriptions.

If more tokens are transferred in the output(s) than are available in the input(s) then the transaction is considered invalid and the tokens are burned. If less tokens are created in the outputs than are available in the intput(s), the unallocated tokens are burned.

Using the same procedure as regular Satoshi transfers allows us to benefit from the parallelisation Bitcoin benefits from, where you can split a specific UTXO with a large amount into smaller UTXOs and spend those in parallel (the same way you could exchange a $100 bill into $1 bills and spend those in parallel) with no sequential bottlenecks that something like ERC20 (Ethereum) sufffers from.
| Key | Required? | Description |
| --- | --------- | ----------- |
| p | Yes | Protocol: `bsv-20` |
| op | Yes | Operation: `transfer` |
| id | Yes* | `<txid>_<vout>` of mint output (* - `tickerless` only) |
| tick | Yes* | Ticker: 4 letter identifier of the bsv-20 (* - `first is first` only) |
| amt | Yes | Amount of tokens transferred in this output. |


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
- {"p":"bsv-20","op":"transfer","id":"3b313338fa0555aebeaf91d8db1ffebd74773c67c8ad5181ff3d3f51e21e0000_1","amt":"100"} 
  
P2PKH inscription
- {"p":"bsv-20","op":"transfer","id":"3b313338fa0555aebeaf91d8db1ffebd74773c67c8ad5181ff3d3f51e21e0000_1","amt":"500"} 


P2PKH inscription
- {"p":"bsv-20","op":"transfer","id":"3b313338fa0555aebeaf91d8db1ffebd74773c67c8ad5181ff3d3f51e21e0000_1","amt":"400"} 


#### `First is first` Example
To transfer the `ordi` tokens that you minted as shown above, you would create a transaction spending the minting UTXO (providing the signature and public key normally to spend the P2PKH script) with an output (or many) with similar scripts with the following json (with `ContentType: application/bsv-20`), as shown below:

To mint `ordi` tokens, you would create an inscription with the following json:

```json
{ 
  "p": "bsv-20",
  "op": "transfer",
  "tick": "ordi",
  "amt": "1000"
}
```

| Inputs | Outputs |
| ------ | ------- |
| Signature Public\_key (spending mint of 1000 ordis) | inscription(`{"p":"bsv-20","op":"transfer","tick":"ordi","amt":"100"}`) |
| | inscription(`{"p":"bsv-20","op":"transfer","tick":"ordi","amt":"500"}`) |
| | inscription(`{"p":"bsv-20","op":"transfer","tick":"ordi","amt":"400"}`) |


## Grandfathering in older inscriptions

There are several inscriptions on the blockchain prior to the release of this spec that use `text/plain` as the content type instead of `application/bsv-20`. We will continue to index these up to block height `793000`. Starting with this block, BSV-20 inscriptions must have a content type of `application/bsv-20` to be considered valid by indexers.

## Implied Transfers Deprecated

V1 of this spec, supported functionality of `implied` transfers. An `implied` transfer was achieved by transferring a `mint` or `transfer` inscription without the creation of a new `transfer` inscription. This functionality was put in place to support users who transferred their inscriptions before there were any publicly accessible tools to create `transfer` inscriptions, and was always a temporary solution. This functionality will be discontinued at block height 807000.

## References

* 1: [BRC20 on BTC](https://domo-2.gitbook.io/brc-20-experiment/)
