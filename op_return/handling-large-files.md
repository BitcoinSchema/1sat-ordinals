# Handling Large Files

If you were to inscribe files larger than 10MB (at the time of writing this) the transaction would generally not be accepted by miners. To work around this, a large data file can be spread across multiple transactions. We can achieve this using the BCAT protocol. BCAT lists, in order, the transaction ids required to assemble the data file. It is a Bitcom protocol that uses the prefix "15DHFxWZJT58f9nhyGnsRBqrgwK4W6h4Up". You can find more information about BCAT protocol [here](https://bcat.bico.media/).

While you cannot add the bcat data directly into the inscription fields (since ordinals protocol does not support concatenating external transactions), you can inscribe a thumbnail as the ordinal image, and attach the video using BCAT in OP\_RETURN as follows.

```bash
1SAT_P2PKH <INSCRIPTION> OP_RETURN <BCAT fields...> tx2 tx3 tx4...
```
