# URL References

### o:// - On-Chain Data References

You can embed references to other ordinals inside the on-chain HTML or markdown.

An ordinalID is the precise satoshi identifier.

Origin is the txid of an inscription, and the output index like `txid_vout`

```html
<img src="ord://ordinalID" />
```

```html
<img src="ord://origin" />
```

Or in markdown

```md
[My Ordinal!]("o://ordinalID")
```

```markdown
[My Ordinal!]("o://origin")
```
