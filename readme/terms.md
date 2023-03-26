# Terms

#### Inscription ID

An outpoint representing a particular inscription, comprised of a transaction ID and output index with the following formatting: `` `txid_vout` ``

#### Inscription Number

This is an auto-incrementing id given to each inscription as they appear in mined blocks, starting with inscription #0. Because inscription numbers are dependent on the order of transactions in a block, inscription numbers are not assigned until a transaction is confirmed. This also makes inscription number sensitive to reorgs.

#### Origin

This is like a "low resolution" ordinal number. It represents the last time a particular ordinal became a 1 satoshi output. Since utxos can be split and joined over time, an ordinal can have more than one origin. This is a 1Sat protocol specific concept, not present in the original ordinals protocol.

#### Ordinal Number

An ordinal is an identifier given to a specific satoshi. It is given when the sat comes into existence during a coinbase transaction. To learn more about this concept read the original "ordinal theory" documentation.
