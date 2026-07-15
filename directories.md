---
description: Host file trees with ord-fs/json directory inscriptions
---

# Directories

An inscription with content type **`ord-fs/json`** is a **directory**. Its body is a JSON object: keys are **single path segment** names, values are **pointers** to other inscriptions (files or nested directories).

**Keys must not contain `/`.** Each directory is one depth only — a flat map of name → pointer. Multi-level URLs (`…/lib/util.js`) come from **recursive** resolution: an entry whose content type is also `ord-fs/json` is another single-level directory, not a key with slashes.

Served by [OrdFS](ordfs.md) under `/content/{outpoint}/…`.

```json
{
  "index.html": "_1",
  "style.css": "_2",
  "lib": "aa11bb22…ff_0",
  "readme.md": "ord://cc33dd…_0"
}
```

## Deploying a directory

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

Every pointed-to outpoint must exist in **this OrdFS instance’s** transaction store (same [scope rules](ordfs.md#scope-of-a-deployment) as other content). Missing children 404.

## Pointer forms

| Pointer | Meaning |
|---------|---------|
| `_N` | Output index `N` in the **same transaction** as the directory inscription (sibling) |
| `txid_vout` or `txid.vout` | Absolute outpoint |
| `txid` (64 hex) | Treated as that transaction’s first resolvable content output |
| `ord://…` | Same as the forms above with an optional `ord://` prefix (stripped) |

[Content references](content-ref.md) reuse these forms for `ref=ordfs` pointer bodies.

## Recursive resolution

`GET /content/{pointer}[:seq]/filepath}` drives directory walk:

1. Load the root pointer. If content type is not `ord-fs/json`, serve the bytes as a normal file (including one-hop [content-ref](content-ref.md) follow when applicable).
2. If it **is** a directory and **filepath is empty**:
   - With **`?raw`**: return the directory JSON (`Content-Type: ord-fs/json`).
   - Otherwise: **redirect** to `{path}/index.html`.
3. Split filepath on `/` into segments. For each segment, in order:
   - Look up the name in the **current** directory map.
   - **SPA fallback:** if the name is missing and this is the **last** segment only, use `index.html` if present.
   - Load that entry’s pointer (same pointer rules). Apply content-ref follow on the entry when relevant.
   - If there are **more** segments and the loaded content is again `ord-fs/json`, **recurse** into that subdirectory with the remaining path.
   - If this is the last segment (or the entry is not a directory), **serve that content**.
4. Nesting is capped at **8** directory levels (`directory nesting too deep` if exceeded).

Example: `/content/{root}/lib/util.js` where `root` is `ord-fs/json` with `"lib" → subdirOutpoint`, and that subdir is `ord-fs/json` with `"util.js" → fileOutpoint`, loads the file through two map lookups.

Intermediate segments that are not directories (or missing keys mid-path without SPA fallback) fail with not found / bad request as appropriate.

## Practical notes

- Include an `index.html` entry for roots and SPAs that rely on redirect and last-segment fallback.
- Nested apps: put another `ord-fs/json` inscription behind a key (e.g. `"docs"`) and link to `/content/{root}/docs/…`.
- Relative `_N` pointers only work when the directory’s own outpoint is known (normal content serving).

## See also

- [OrdFS](ordfs.md) — gateway resolution, seq, MAP, HTTP routes  
- [Streams](streams.md) — large files split across a transfer chain  
- [Content References](content-ref.md) — shared payload via `ref=ordfs`  
