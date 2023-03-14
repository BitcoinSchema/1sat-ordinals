# Text Inscriptions

A substandard has emerged for recording other protocols as text files, using the inscription as an envelope. This is done by creating an inscription with `text/plain;charset=utf-8` in the `content-type` field along with a JSON payload for the data:

```json
{ "p": "brc-20", "op": "mint", "tick": "meme", "amt": "1" }
```

Two such early protocols are sns and brc-20. Since 1Sat Inscriptions follow the same data protocol, text-based inscription sub-protocols are instantly portable to BSV.
