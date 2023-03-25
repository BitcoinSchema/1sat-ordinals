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

## #latest - Dynamic Inscription References

Use the `#latest` fragment to return the latest inscription for a given ordinal number, or inscription ID.

### Latest inscription by Ordinal Number

In a text/html inscription:

```html
<img src="ord://ordinalNumber#latest" />
```

In a text/markdown inscription:

```markdown
[My Ordinal!]("ord://ordinalNumber#latest")
```

### Latest inscription by Inscription ID

In a text/html inscription:

```html
<img src="ord://inscriptionID#latest" />
```

In a text/markdown inscription:

```md
[My Ordinal!]("ord://inscriptionID#latest")
```

## URL Resolution

| URL                        | resolves to          |
| -------------------------- | -------------------- |
| ord://ordinalNumber        | original inscription |
| ord://inscriptionID        | specific inscription |
| ord://ordinalNumber#latest | latest inscription   |
| ord://inscriptionID#latest | latest inscription   |
