---
description: Resolve ordinal content and metadata over HTTP
---

# OrdFS

OrdFS is an HTTP gateway for 1Sat ordinals: it turns on-chain inscription state into ordinary web resources. Apps load inscription bytes, walk transfer history, and read accumulated MAP metadata without reimplementing ordinal crawls.

It does not mint or transfer ordinals. It **reads** content and chain state that already exist on Bitcoin.

## What you can do with it

- **Serve content** — fetch inscription (or B-protocol) bytes by outpoint with the right `Content-Type` (images, video, HTML, JSON, …).
- **Follow the ordinal** — resolve origin, a specific sequence, or the current tip of the transfer chain.
- **Read application state** — merge MAP fields written along that chain (collections, `opns.idKey`, custom keys).
- **Host small apps** — directories (`ord-fs/json`), path traversal, SPA-style fallback to `index.html`.
- **Stream large media** — multi-inscription streams with HTTP Range when content is chunked on-chain.

Services built on names and paymail use the same model: look up a name’s **origin**, then use OrdFS for tip content and merged MAP (see [Payments](name-service/payments.md)).

## How resolution works

An **outpoint** (`txid_vout`) names a transaction output. OrdFS loads that output, extracts content when present, and can walk the ordinal’s spend chain.

| Concept | Meaning |
|--------|---------|
| **Origin** | First 1-sat inscription outpoint in the ordinal lineage |
| **seq** | Position along the transfer chain (see below) |
| **Content** | Inscription envelope, or B-protocol data when no inscription type is set |
| **MAP merge** | All MAP entries up through the target sequence; later keys win |

### Sequence (`seq`)

Appended to the pointer as `{outpoint}:{seq}` (e.g. `…_0:-1`).

| seq | Behavior |
|-----|----------|
| *(omitted)* | Content at that outpoint only — no chain crawl |
| `-2` | Content at the **origin** |
| `0`, `1`, … | Content as of that absolute sequence (ownership step) |
| `-1` | **Tip** — current end of the transfer chain |

Ownership transfers and content reinscriptions are tracked separately. Requesting a sequence returns the **latest content at or before** that step, so a pure transfer does not clear prior inscription bytes.

### MAP

MAP (`1PuQa7K62MiKCtssSLKy1kh56WWU7MtUR5`) on outputs along the chain is merged chronologically. That is how mutable fields such as OpNS identity (`opns.idKey`) stay attached to a name across transfers without changing the inscription body.

## Scope of a deployment

An OrdFS instance is backed by a **transaction store and spend index** — typically the same graph an overlay has admitted, not necessarily the full blockchain.

It does not need a complete global history to be useful. If a request needs an ancestor, spend, or outpoint **outside** what that instance has, OrdFS responds with **404**. That is normal for a bounded deployment, not a protocol failure.

## HTTP surface

**Content** is mounted at the host root. Other OrdFS routes sit under `/1sat/ordfs`. Paths use outpoint form `txid_vout`.

| Use | Method / path shape |
|-----|---------------------|
| Content | `GET /content/{outpoint}[:seq][/filepath]` |
| Metadata | `GET /1sat/ordfs/metadata/{outpoint}[:seq]` |
| Bulk metadata | `POST /1sat/ordfs/metadata` with a list of outpoints |
| Stream | `GET /1sat/ordfs/stream/{outpoint}` |
| Preview | `GET` / `POST` `/1sat/ordfs/preview…` |

Useful response headers when present: `X-Outpoint`, `X-Origin`, `X-Ord-Seq`, `X-Map`, `X-Parent`. Fixed-sequence content can be cached as immutable; tip (`seq=-1`) is not.

### Directories

An inscription with content type `ord-fs/json` is a **directory**. Its body is a JSON object mapping names to outpoint pointers:

```json
{
  "index.html": "abc123def…_0",
  "style.css": "fedcba…_0",
  "app.js": "012345…_0"
}
```

On `GET /content/{dirOutpoint}/…`:

- The **directory root** resolves to `index.html` when present.
- Each **path segment** is looked up in the map and loaded as content (which may itself be another directory).
- If a path is missing, OrdFS serves `index.html` when available (SPA-style fallback).
- Query **`?raw`** returns the directory JSON as-is instead of following entries.

That is how multi-file HTML apps sit on-chain as a tree of inscriptions and are served like a static site under `/content/…`.

### Streams

Large media can be split across **several inscriptions on one ordinal transfer chain**:

1. The **first** chunk uses the real media type with a `stream=ordfs` parameter (for example `video/mp4; stream=ordfs`).
2. **Later** chunks use content type `ordfs/stream`.

OrdFS detects that pattern, walks the spend chain, and **concatenates** the chunks into one response. `GET /1sat/ordfs/stream/{outpoint}` is the dedicated stream route; content routes honor the same markers when applicable. **HTTP Range** is supported for partial reads (seek / progressive download).

## Names and payments

[OpNS](name-service/opns.md) defines how names are mined and where each name’s **origin** is. OrdFS resolves **forward** from that origin (tip content and merged MAP). [Payments](name-service/payments.md) composes the two: origin from OpNS, identity key from OrdFS MAP.

## See also

- [Metadata](adding-metadata/README.md) — MAP and schema types on ordinals  
- [HTML inscriptions](html-ordinals/README.md) — content that often loads via OrdFS URLs  

