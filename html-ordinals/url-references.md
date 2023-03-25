---
description: Using ord:// and sat:// URL protocol handlers to reference inscriptions.
---

# URL References

### ord:// - On-Chain Inscription References

You can embed references to other inscriptions inside the on-chain HTML or markdown. You can address.

An ordinal number is the precise satoshi identifier.

`Inscription ID` is the transaction id, and output index of a specific inscription, formatted as: `txid_vout`

### Original inscription by Ordinal Number

In a text/html inscription:

```html
<img src="ord://ordinalNumber" />
```

In a text/markdown inscription:

```markdown
[My Ordinal!]("ord://ordinalNumber")
```

### Specific inscription by Inscription ID

In a text/html inscription:

```html
<img src="ord://inscriptionID" />
```

In a text/markdown inscription:

```md
[My Ordinal!]("ord://inscriptionID")
```

## sat:// - Dynamic Inscription References

The sat prefix return the latest inscription for a given ordinal number, or ordinal ID.

### Latest inscription by Ordinal Number

In a text/html inscription:

```html
<img src="ord://ordinalNumber" />
```

In a text/markdown inscription:

```markdown
[My Ordinal!]("ord://ordinalNumber")
```

### Latest inscription by Inscription ID

In a text/html inscription:

```html
<img src="ord://inscriptionID" />
```

In a text/markdown inscription:

```md
[My Ordinal!]("ord://inscriptionID")
```

## Return Value Comparison

| value          | ord://               | sat://             |
| -------------- | -------------------- | ------------------ |
| ordinal Number | original inscription | latest inscription |
| inscription ID | specific inscription | latest inscription |
