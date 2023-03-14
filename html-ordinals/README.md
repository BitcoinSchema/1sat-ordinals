# HTML Ordinals

### Inscribing Web Pages

To inscribe a webpage, simply:

```bash
1SAT_P2PKH OP_IF "ord" OP_1 "text/html;charset=utf8" OP_0 <html_data> OP_ENDIF
```

### o:// - Ordinal URL References

You can embed references to other ordinals inside the on-chain HTML or markdown.

An ordinalID is the precise satoshi identifier.

```html
<img src="o://ordinalID" />
```

Or in markdown

```markdown
[My Ordinal!]("o://ordinalID")
```
