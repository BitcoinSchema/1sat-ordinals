# Libraries & software

This site is the **protocol** specification for 1Sat Ordinals. Application code and hosted services live in separate repositories.

## Current

### 1sat-sdk

TypeScript monorepo for building on 1Sat Ordinals: wallets, actions (inscribe, transfer, list, tokens, OpNS bindings), HTTP clients, script templates, CLI, and browser connect.

| | |
|--|--|
| **Repository** | [github.com/b-open-io/1sat-sdk](https://github.com/b-open-io/1sat-sdk) |
| **npm scope** | [`@1sat/*`](https://www.npmjs.com/org/1sat) |

Common packages (install what you need):

| Package | Role |
|---------|------|
| [`@1sat/cli`](https://www.npmjs.com/package/@1sat/cli) | CLI (`bunx @1sat/cli`) |
| [`@1sat/actions`](https://www.npmjs.com/package/@1sat/actions) | High-level wallet actions |
| [`@1sat/wallet`](https://www.npmjs.com/package/@1sat/wallet) | BRC-100 wallet engine |
| [`@1sat/client`](https://www.npmjs.com/package/@1sat/client) | HTTP client for hosted APIs |
| [`@1sat/templates`](https://www.npmjs.com/package/@1sat/templates) | Script templates (OpNS, BSV-21, OrdLock, …) |
| [`@1sat/connect`](https://www.npmjs.com/package/@1sat/connect) | Browser wallet popup protocol |
| [`@1sat/react`](https://www.npmjs.com/package/@1sat/react) | React hooks |

Underlying crypto and transaction types typically come from [`@bsv/sdk`](https://www.npmjs.com/package/@bsv/sdk).

See the SDK repository README for setup, skills, and examples. This docs site does **not** fully specify the SDK API surface.

### 1sat-stack

Go server that hosts the live 1Sat stack: transaction/BEEF storage, ordinal resolution ([OrdFS](ordfs.md)), OpNS overlay, paymail, marketplace (OrdLock), owner sync, and related HTTP APIs. Modules can be embedded, remote, or disabled per deployment.

| | |
|--|--|
| **Repository** | [github.com/b-open-io/1sat-stack](https://github.com/b-open-io/1sat-stack) |
| **Public host (example)** | `https://api.1sat.app` |

Capability mounts commonly sit under `/1sat/…` (e.g. `/1sat/opns`, `/1sat/ordfs`). Content is at root **`/content/…`**. Exact routes depend on configuration; use the deployment’s OpenAPI/swagger where enabled.

App clients usually talk to a stack host via `@1sat/client` rather than reimplementing endpoints.

---

## Historical / deprecated

The following libraries and servers predate the current SDK and stack. They remain useful for archaeology and old integrations but are **not** the recommended path for new work.

### Javascript

#### js-1sat-ord (deprecated)

Legacy JS library. Docs site: [js.1satordinals.com](https://js.1satordinals.com/). Repo: [bitcoinschema/js-1sat-ord](https://github.com/bitcoinschema/js-1sat-ord). Prefer **1sat-sdk** (`@1sat/*`).

#### bmapjs

Transaction parser with 1Sat inscription and metadata support: [rohenaz/bmap](https://github.com/rohenaz/bmap) (`npm i bmapjs`). Still useful for BMAP-shaped parsing; not a full 1Sat app stack.

### Go

#### go-1sat-ord (deprecated)

Legacy Go helpers for creating 1Sat transactions: [bitcoinschema/go-1sat-ord](https://github.com/bitcoinschema/go-1sat-ord). Prefer **1sat-stack** templates/packages and **1sat-sdk** for app-level flows.

#### go-bmap

Transaction parser with 1Sat inscription and metadata support: [bitcoinschema/go-bmap](https://github.com/bitcoinschema/go-bmap).
