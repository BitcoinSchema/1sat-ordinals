# Location Tagging

Create an Ordinal with initial location:

```
1SAT_P2PKH <INSCRIPTION> OP_RETURN MAP app <appname> type "ordinal" context geohash geohash <geohash> | AIP <sig...>
```

Move the location by spending the ordinal with a MAP geohash of the new location. Updates can be checked by an indexer, the signature validated it came from the inscriber, and surface the new location to the user along with an update history. See the signing section for more information on signing Ordinals.

```
i1 - 1 sat o1 from inscription tx
o1 - 1SAT_P2PKH OP_RETURN MAP geohash <geohash> | AIP <sig...>
```

Complex example: To inscribe a video that exists across multiple transactions and place it on a specific geohash with identity signature

```
tx1 - o1 - 1SAT_P2PKH OP_RETURN BCAT tx2 tx3 tx4... | MAP SET context geohash geohash <geohash> | AIP <sig...> -1

tx2 - o1 - BCAT_PART <tx2_data>
```
