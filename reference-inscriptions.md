---
description: Inscribe references to content by URI
---

# Reference Inscriptions

You can inscribe references to on-chain content using the text/uri-list content type defined here:\
[https://www.rfc-editor.org/rfc/rfc2483#section-5](https://www.rfc-editor.org/rfc/rfc2483#section-5)

### Examples

```
# This is a comment
https://mydomain.com/something.pdf
```

In this example we reference a JSON inscription. The file extension is useful for hinting before the content has been loaded, allowing clients to make UI decisions sooner (like choosing the correct component to render the content).

Using [OrdFS](ordfs.md)-compatible content routes, you can reference on-chain files by relative path:

```
# This is an image inscription
/content/<outpoint | contentHash>.png
```

See [OrdFS](ordfs.md), [Directories](directories.md), and [Streams](streams.md) for resolution rules. Gateways such as [ordfs.network](https://ordfs.network) expose a compatible `/content/` path.

### Multiple Files

You can also reference many files in a single text/uri-list inscription:

```
/content/<outpoint1>.html
/content/<outpoint2>.js
/content/<outpoint3>.webp
/content/<outpoint4>.css
```

In this example we can make the contents of a webpage and all of its dependencies available as a single package.

### Referencing shared content (collections & large mints)

A reference points at content that already exists on-chain, so many inscriptions
can share one payload instead of each re-inscribing it. For a collection — or any
large mint — inscribe the artwork **once** and have every item reference that
outpoint. A 10,000-item drop then costs one image plus a tiny reference per item,
not 10,000 copies, and the same holds when items are minted on demand.

Produce a referenced item as an **`ord-fs/json` directory**, resolved by
[OrdFS](ordfs.md). Its content-type honestly signals that the content must be
resolved, and it fails safe — a consumer that doesn't resolve it sees a
non-media type rather than mis-rendering a pointer as image bytes.

- **Single shared or unique image** — a one-entry directory using the `.`
  default-entry key. OrdFS serves the `.` entry in place at the item's own URL
  (see [Directories](ordfs.md#directories)), so no interior filename is needed:

  ```json
  { ".": "<imageOutpoint>" }
  ```

- **Multi-leaf item** — add named leaves for per-item metadata or attachments
  alongside the default entry:

  ```json
  { ".": "<imageOutpoint>", "meta.json": "_1" }
  ```

The `<imageOutpoint>` pointer may be a same-transaction relative `_N` or an
absolute `<txid>_<vout>` (cross-tx, e.g. mint-on-demand). A resolver also
**accepts** a `text/uri-list` item (RFC 2483) for interoperability with tools
that emit it, but new items are produced as `ord-fs/json`. (An earlier
`ref=ordfs` media-type-parameter form was considered and dropped: declaring a
real image type over a pointer body mis-renders in any consumer that reads the
inscription without stripping the parameter.)

#### Tradeable items, shared content

Keep the collection and item MAP envelopes unchanged:

- **Collection** — remains the classic `image/*` MAP `collection` ordinal.
- **Item** — remains a tradeable 1-sat MAP `collectionItem` ordinal with the
  same `collectionId`, mint, rarity, trait, and attachment fields. Only its
  inscription content changes from image bytes to a reference.
- **Referenced content** — can be an existing inscription or a 0-sat bitcom
  `B` output. Directory manifests may point to shared or item-specific content
  without changing collection membership.

The collection and item carry an authorship signature whose signer identifies
the collection author. Use **SIGMA** for the 1-sat ordinal (its prefix commits
to a transaction input, so the signature is replay-resistant and cannot be
lifted onto another item); legacy AIP-signed collections remain valid. Signing
follows output type: a signed 1-sat ordinal uses SIGMA and a signed 0-sat `B`
output uses AIP; immutable leaves may also be unsigned.

#### Resolution notes

- A referenced or leaf output resolves only if its transaction is in the serving
  OrdFS instance's store (same [scope rules](ordfs.md#scope-of-a-deployment)).
- Leaf content must be an inscription envelope or a **bitcom `B`** output; a raw
  `OP_RETURN` pushdata that is not `B` carries no servable content type.
- `{outpoint}:-1` "latest" applies to a **1-sat** ordinal — it follows the
  transfer chain. 0-sat B leaves are immutable and resolve at their exact
  outpoint, and directory traversal resolves leaves pinned; request a nested
  manifest directly with `:-1` to resolve its tip.
