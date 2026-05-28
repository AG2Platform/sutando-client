# `@sutando/client`

Sutando's React frontend — the conversation page that fronts the voice
agent, task stream, and dynamic content panels.

This repo is one implementation of the
[Sutando wire contract](./WIRE.md). The companion server lives at
[**sutando**](https://github.com/sonichi/sutando); any backend that
honors `WIRE.md` can serve this UI, and anyone can replace this UI with
their own as long as it honors `WIRE.md`.

## Standalone build

```bash
pnpm install
pnpm build       # writes ./dist/
```

Point a running Sutando voice-agent at the built bundle:

```bash
CLIENT_DIST_DIR=/abs/path/to/sutando-client/dist bash <sutando-repo>/src/startup.sh
```

## Standalone dev

```bash
pnpm dev         # vite dev server on http://localhost:5173
```

The dev server expects a Sutando voice-agent to be running so the page
can hit `/sse-status`, `/voice-mode`, etc. If the agent is on a
non-default origin, pass query overrides:

```
http://localhost:5173/?api=http://localhost:8080&agent-api=http://localhost:7843
```

## Used as a git submodule inside sutando

In the canonical sutando installation, this repo is checked out at
`sutando/client/` as a git submodule. `pnpm install` at the sutando root
picks the directory up as a workspace package automatically.

```bash
git clone --recurse-submodules https://github.com/sonichi/sutando.git
cd sutando
pnpm install
pnpm build:client    # equivalent to running `pnpm build` here
```

If you already cloned without `--recurse-submodules`:

```bash
git submodule update --init --recursive
```

## Architecture

Follows the conventions in
[`sutando/CLAUDE.md` § Frontend Conventions](https://github.com/sonichi/sutando/blob/main/CLAUDE.md):

| Layer | Purpose | Size budget |
|-------|---------|-------------|
| `pages/<page>/` | Thin orchestration; one per route | n/a |
| `components/atoms/` | Pure presentational | < 70 lines |
| `components/molecules/` | Composed of atoms | 70–150 lines |
| `components/organisms/` | Feature-complete sections | > 150 lines |
| `contexts/` | React Context providers | – |
| `hooks/` | Data fetching + business logic | – |
| `utils/` | Pure functions | – |
| `const-values/` | Copy + static config | – |
| `lib/` | Infrastructure (`config`, `api`, `sse`) | – |

**Rules:**
- Components render UI only. No `fetch` in components — hooks own that.
- No hardcoded strings. Copy lives in `const-values/`.
- One file ≤ ~150 lines. Split before it grows past that.
- `===` not `==`. `const` over `let`. Early returns over nesting.

## Routing

One route today (`conversation`), so there's no `react-router`.
`src/lib/config.ts` parses `?page=` into `initialRouteId` (default
`conversation`) so the shell is ready when additional panes land;
until then `App.tsx` mounts `ConversationPage` unconditionally.

## Server-agnostic config

`src/lib/config.ts` resolves the WebSocket URL + API origins at runtime:

1. `?ws=` / `?api=` / `?agent-api=` query string (highest priority)
2. `window.__SUTANDO_CONFIG__` (injected by the host shell)
3. `window.location.host` (default — works for desktop WKWebView, remote
   browser over Tailscale, and a future mobile thin-client without
   rebuilding)

## Contributing

PRs welcome. If you change the wire shape (endpoints, SSE event names,
request/response payloads), update
[`WIRE.md`](./WIRE.md) in the same PR — that file is the contract this
UI shares with every Sutando backend.

License: [MIT](./LICENSE)
