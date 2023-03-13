# Signing

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
