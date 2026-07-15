---
description: Large files split across an ordinal transfer chain
---

# Streams

Large payloads can be split across **multiple inscriptions on one ordinal transfer chain**, then reassembled by OrdFS.

## On-chain layout

1. **Chunk 0 (start of stream):** the payload’s actual content type (e.g. `video/mp4`, `application/octet-stream`). Conventionally mark it as streamable, e.g. `video/mp4; stream=ordfs` (parameter on the type string). Body is the first slice of bytes.
2. **Further chunks:** each successive **spend** of the ordinal carries the next slice with content type exactly **`ordfs/stream`**.
3. Chain ends when a spend has no further content, spend is missing, or a later output’s type is **not** `ordfs/stream` (after the first chunk).

All chunks must be in the instance’s BEEF/spends graph so the stream walk can follow the ordinal. Same [scope rules](ordfs.md#scope-of-a-deployment) as other OrdFS loads.

## Serving

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

`GET /content/{outpoint}` on a stream origin returns only that outpoint’s inscription (typically chunk 0), not the assembled stream. Use the stream route for full media.

## Deploying a stream

1. Split the file into chunks sized for your inscription limits (1 MiB bodies are a practical default).
2. Mint chunk 0 as a 1-sat inscription with the public content type (and optional `stream=ordfs` parameter).
3. Transfer/reinscribe the same ordinal for each subsequent chunk with type `ordfs/stream` and the next bytes (order is spend order).
4. Point clients at `/1sat/ordfs/stream/{firstChunkOutpoint}` (or an outpoint mid-chain if you only need a suffix — walk starts from the requested outpoint).

If a middle chunk is missing from the store, the stream stops or errors when that spend cannot be loaded.

## Reference mint (`@1sat/actions`)

The TypeScript SDK’s `inscribe` action can build this layout:

- `stream: true` — multi-tx stream with **1 MiB** chunk bodies  
- `streamChunkSize: N` — multi-tx stream with custom body size  

Omit both for a single-transaction inscription. Stream outputs are tagged with `sha256:<content-hash>` and `stream-i:<index>`. See [Libraries](Libraries.md) and [1sat-sdk](https://github.com/b-open-io/1sat-sdk).

## See also

- [OrdFS](ordfs.md) — gateway resolution, seq, MAP, HTTP routes  
- [Directories](directories.md) — multi-file sites via `ord-fs/json`  
- [Content References](content-ref.md) — shared payload via `ref=ordfs`  
