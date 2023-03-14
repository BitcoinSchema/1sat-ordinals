# Collections

You can also mint lots of ordinals at once, and link them to a specific collection.

```
o1 - 1SAT_P2PKH <INSCRIPTION> OP_RETURN MAP SET app <mint_platform> type ord collection <unique-collection-name>
o2 - 1SAT_P2PKH <INSCRIPTION> OP_RETURN MAP SET app <mint_platform> type ord collection <unique-collection-name>
```

Once transaction size surpasses 10MB, they remain associated after batching:

Transaction 1:

```
o1 - 1SAT_P2PKH <INSCRIPTION> OP_RETURN MAP SET app 1sat.app type ord collection 1satpunks
...
oN - 1SAT_P2PKH <INSCRIPTION> OP_RETURN MAP SET app 1sat.app type ord collection 1satpunks
```

Transaction 2:

```
o1 - 1SAT_P2PKH <INSCRIPTION> OP_RETURN MAP SET app 1sat.app type ord collection 1satpunks
...
oN - 1SAT_P2PKH <INSCRIPTION> OP_RETURN MAP SET app 1sat.app type ord collection 1satpunks
```
