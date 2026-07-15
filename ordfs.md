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

An inscription with content type **`ord-fs/json`** is a **directory**. Its body is a JSON object: keys are **single path segment** names, values are **pointers** to other inscriptions (files or nested directories).

**Keys must not contain `/`.** Each directory is one depth only — a flat map of name → pointer. Multi-level URLs (`…/lib/util.js`) come from **recursive** resolution: an entry whose content type is also `ord-fs/json` is another single-level directory, not a key with slashes.

```json
{
  "index.html": "_1",
  "style.css": "_2",
  "lib": "aa11bb22…ff_0",
  "readme.md": "ord://cc33dd…_0"
}
```

#### Deploying a directory

1. Inscribe each file (and any nested directory inscriptions) as its own 1-sat output.
2. Inscribe the directory itself with:
   - Content type: `ord-fs/json`
   - Body: JSON map as above
3. Prefer putting **siblings in the same transaction** and pointing at them with relative vouts (`_1`, `_2`, …) so one mint tx holds the tree root and leaves. Absolute outpoints work when children live in other txs.
4. Serve via content URL. The directory outpoint is the site root:

```text
GET /content/{dirTxid}_{dirVout}/
GET /content/{dirTxid}_{dirVout}/style.css
GET /content/{dirTxid}_{dirVout}/lib/util.js
```

Every pointed-to outpoint must exist in **this OrdFS instance’s** transaction store (same scope rules as above). Missing children 404.

#### Pointer forms

| Pointer | Meaning |
|---------|---------|
| `_N` | Output index `N` in the **same transaction** as the directory inscription (sibling) |
| `txid_vout` or `txid.vout` | Absolute outpoint |
| `txid` (64 hex) | Treated as that transaction’s first resolvable content output |
| `ord://…` | Same as the forms above with an optional `ord://` prefix (stripped) |

#### Recursive resolution

`GET /content/{pointer}[:seq]/filepath}` drives directory walk:

1. Load the root pointer. If content type is not `ord-fs/json`, serve the bytes as a normal file.
2. If it **is** a directory and **filepath is empty**:
   - With **`?raw`**: return the directory JSON (`Content-Type: ord-fs/json`).
   - Otherwise: **redirect** to `{path}/index.html`.
3. Split filepath on `/` into segments. For each segment, in order:
   - Look up the name in the **current** directory map.
   - **SPA fallback:** if the name is missing and this is the **last** segment only, use `index.html` if present.
   - Load that entry’s pointer (same pointer rules).
   - If there are **more** segments and the loaded content is again `ord-fs/json`, **recurse** into that subdirectory with the remaining path.
   - If this is the last segment (or the entry is not a directory), **serve that content**.
4. Nesting is capped at **8** directory levels (`directory nesting too deep` if exceeded).

Example: `/content/{root}/lib/util.js` where `root` is `ord-fs/json` with `"lib" → subdirOutpoint`, and that subdir is `ord-fs/json` with `"util.js" → fileOutpoint`, loads the file through two map lookups.

Intermediate segments that are not directories (or missing keys mid-path without SPA fallback) fail with not found / bad request as appropriate.

#### Practical notes

- Include an `index.html` entry for roots and SPAs that rely on redirect and last-segment fallback.
- Nested apps: put another `ord-fs/json` inscription behind a key (e.g. `"docs"`) and link to `/content/{root}/docs/…`.
- Relative `_N` pointers only work when the directory’s own outpoint is known (normal content serving).

### Streams

Large payloads can be split across **multiple inscriptions on one ordinal transfer chain**, then reassembled by OrdFS.

#### On-chain layout

1. **Chunk 0 (start of stream):** the payload’s actual content type (e.g. `video/mp4`, `application/octet-stream`). Conventionally mark it as streamable, e.g. `video/mp4; stream=ordfs` (parameter on the type string). Body is the first slice of bytes.
2. **Further chunks:** each successive **spend** of the ordinal carries the next slice with content type exactly **`ordfs/stream`**.
3. Chain ends when a spend has no further content, spend is missing, or a later output’s type is **not** `ordfs/stream` (after the first chunk).

All chunks must be in the instance’s BEEF/spends graph so the stream walk can follow the ordinal.

#### Serving

```text
GET /1sat/ordfs/stream/{outpoint}
```

OrdFS:

1. Resolves origin / chain from the starting outpoint.
2. Walks spends forward, concatenating content bodies in order.
3. Sets `Content-Type` from the **first** chunk’s type (so clients see `video/mp4`, not `ordfs/stream`).
4. Supports **HTTP Range** (`Range: bytes=start-end`) for seek and progressive download; only the relevant portions of chunks are written.

Example:

```bash
curl -H "Range: bytes=0-1023" "https://{host}/1sat/ordfs/stream/{txid}_{vout}"
```

#### Deploying a stream

1. Split the file into chunks sized for your inscription limits (1 MiB bodies are a practical default).
2. Mint chunk 0 as a 1-sat inscription with the public content type (and optional `stream=ordfs` parameter).
3. Transfer/reinscribe the same ordinal for each subsequent chunk with type `ordfs/stream` and the next bytes (order is spend order).
4. Point clients at `/1sat/ordfs/stream/{firstChunkOutpoint}` (or an outpoint mid-chain if you only need a suffix — walk starts from the requested outpoint).

If a middle chunk is missing from the store, the stream stops or errors when that spend cannot be loaded — same instance-scope rules as content.

#### Reference mint (`@1sat/actions`)

The TypeScript SDK’s `inscribe` action can build this layout:

- `stream: true` — multi-tx stream with **1 MiB** chunk bodies  
- `streamChunkSize: N` — multi-tx stream with custom body size  

Omit both for a single-transaction inscription. Stream outputs are tagged with `sha256:<content-hash>` and `stream-i:<index>`. See [Libraries](Libraries.md) and [1sat-sdk](https://github.com/b-open-io/1sat-sdk).

## Names and payments

[OpNS](name-service/opns.md) defines how names are mined and where each name’s **origin** is. OrdFS resolves **forward** from that origin (tip content and merged MAP). [Payments](name-service/payments.md) composes the two: origin from OpNS, identity key from OrdFS MAP.

## See also

- [Metadata](adding-metadata/README.md) — MAP and schema types on ordinals  
- [HTML inscriptions](html-ordinals/README.md) — content that often loads via OrdFS URLs  

