---
description: Share one on-chain payload across many inscriptions via OrdFS refs
---

# Content References (OrdFS Ref)

Inscribe the same file many times without repeating the bytes. Each **edition** is a normal 1-sat ordinal (own sat, origin, ownership, MAP). Its inscription body is a **pointer** to a **source** inscription that holds the real payload. OrdFS follows the pointer when serving `/content/`.

Implemented in [1sat-stack](https://github.com/b-open-io/1sat-stack) OrdFS (`ref=ordfs` on the content path).

## On-chain layout

### Source (inscribed once)

| Field | Value |
|--------|--------|
| Content type | Public MIME of the payload (e.g. `image/png`, `text/html`) |
| Body | Full file bytes |

Optional: if the source is an OrdFS stream origin, use the stream layout from [OrdFS — Streams](ordfs.md#streams) (e.g. `video/mp4; stream=ordfs` on chunk 0).

### Edition (ref)

| Field | Value |
|--------|--------|
| Content type | Same public MIME as the source, plus parameter **`ref=ordfs`** |
| Body | A single **pointer** to the source (see below) |

Example content type:

```text
image/png; ref=ordfs
```

Envelope shape is unchanged:

```text
OP_FALSE OP_IF "ord" OP_1 <content-type> OP_0 <pointer> OP_ENDIF
```

The parameter is a gateway directive (same idea as `stream=ordfs` on stream origins). Indexers that strip media-type parameters still treat the edition as `image/png` (or whatever base type).

### Pointer body

Body is UTF-8 text: one pointer, no trailing path, no multi-line list. Same pointer forms as [OrdFS directories](ordfs.md#pointer-forms):

| Pointer | Meaning |
|---------|---------|
| `txid_vout` or `txid.vout` | Absolute outpoint of the source |
| `txid` (64 hex) | That transaction’s first resolvable content output |
| `ord://…` | Same forms with optional `ord://` prefix (stripped) |
| `_N` | Output index `N` in the **same transaction** as this edition (sibling source) |

Examples:

```text
aa11bb22cc33dd44ee55ff6677889900aa11bb22cc33dd44ee55ff6677889900_0
```

```text
_0
```

(when source and edition are co-minted in one transaction and the source is vout 0)

Do not put the media type or extra JSON in the body. The type field already carries the public MIME.

## What is not a content ref

| Pattern | Role |
|---------|------|
| `ord-fs/json` | Directory map; path walk, not “this outpoint *is* that content” |
| Parent (field 3) | Collection / hierarchy metadata — not content source |
| Bare `image/png` with an outpoint string as body | Invalid as a ref; gateways would treat the body as PNG bytes |

Without `ref=ordfs`, OrdFS must not follow the body as a pointer.

## OrdFS serving

### Content

```text
GET /content/{editionOutpoint}[:seq][/filepath]
```

When the resolved inscription’s content type includes the parameter `ref=ordfs`:

1. Parse the body as a pointer (forms above).
2. Load the **source** (**one hop** only — nested refs are not followed).
3. Replace the response **body** and **Content-Type** with the source’s.
4. Leave ordinal identity alone: `X-Outpoint`, `X-Origin`, `X-Ord-Seq`, `X-Map`, and `X-Parent` stay on the **requested** outpoint (transfer chain / seq). They do not track the content-ref target.

If the pointer is missing, invalid, or the source is not in this OrdFS instance’s store → **404** (same scope rules as other OrdFS loads).

### Raw (no follow)

```text
GET /content/{editionOutpoint}?raw
```

Return the edition inscription as stored: type `…; ref=ordfs`, body = pointer bytes. Do not follow.

### Metadata

```text
GET /1sat/ordfs/metadata/{editionOutpoint}[:seq]
```

Metadata describes the **inscription envelope** at the ordinal-resolved outpoint (type as inscribed, including `ref=ordfs`, pointer body length). It does not follow content refs. Use `/content/` when you need resolved bytes and type.

### Directory interaction

If a directory entry points at an edition outpoint, path resolution loads that entry and applies the same one-hop content-ref follow before serving (or before treating the result as a nested `ord-fs/json` directory).

### Stream sources

Content-ref follow loads the **source outpoint’s** inscription only (one hop). It does not walk an OrdFS stream chain.

| Request | Behavior |
|---------|----------|
| `GET /content/{edition}` → source is stream origin | First chunk body and that chunk’s content type |
| Full multi-chunk media | `GET /1sat/ordfs/stream/{sourceOutpoint}` |

## Deploying editions

1. Inscribe the **source** once with the real MIME type and full body (or deploy a stream as in [OrdFS](ordfs.md#streams)).
2. For each edition, inscribe a 1-sat output with:
   - Content type: `{sourcePublicType}; ref=ordfs`
   - Body: pointer to the source (`txid_vout` after confirmation, or `_N` if co-minted)
3. Optional MAP / collection metadata on the **edition** as usual (name, collection item, etc.).
4. Clients and marketplaces use `/content/{editionOutpoint}` for display; ownership and transfers apply to the edition’s ordinal.

### Co-mint (same transaction)

```text
vout 0  source   image/png                 <png bytes>
vout 1  edition  image/png; ref=ordfs      _0
vout 2  edition  image/png; ref=ordfs      _0
…
```

Relative `_N` only works when the edition’s own outpoint is known (normal content serving).

## Indexing notes

- Type events should continue to use the base type after stripping parameters (e.g. `type:image`, `type:image/png`), same as other parameterized types.
- The edition body is small (pointer only); do not treat it as the media payload for size or hash of the artwork unless following the ref.
- Optional wallet tags (off-chain): `ref:{sourceOutpoint}`, and/or `sha256:` of the source body when known — not part of the on-chain standard.

## Relation to other standards

| Standard | Difference |
|----------|------------|
| [Streams](ordfs.md#streams) | One ordinal, many spends, concatenate bytes. Ref: many ordinals, one shared source, no re-inscribe. |
| [Directories](ordfs.md#directories) | Map of names → pointers for path hosting. Ref: the outpoint itself *is* the logical file. |

## See also

- [OrdFS](ordfs.md) — content serving, directories, streams  
- [Libraries](Libraries.md)  
