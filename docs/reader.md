# The reader app — `go-reddit/reader`

A native **macOS** Reddit reader whose entire UI is drawn by
[go-widgets](https://github.com/go-widgets/toolkit) — compiled to WebAssembly,
blitted into a `<canvas>`, and hosted in a **WKWebView** that is opened from Go
with **zero cgo** (`CGO_ENABLED=0`). Module path `github.com/go-reddit/reader`.

## Architecture

```
┌─ "Reddit Reader.app"  (pure Go, CGO=0, self-contained) ───────────────┐
│                                                                        │
│   internal/ui        go-widgets toolkit renders the feed → RGBA buffer │
│   cmd/front  (wasm)  blits that buffer into a browser <canvas>         │
│   internal/server    serves the wasm + proxies /api → Reddit           │
│   internal/webview   opens a WKWebView via purego (no cgo)             │
│                                                                        │
│   Transport (default): a WKURLSchemeHandler serves every request       │
│   in-process — no TCP, no unix socket, no listening port at all.       │
│                                                                        │
│        fetch  reader://app/api/feed ──► go-reddit/reddit ──► reddit.com │
└────────────────────────────────────────────────────────────────────────┘
```

The layout mirrors the Reddit web UI: a **topbar** with sort tabs
(hot/new/top/rising) and the current feed, a **sidebar** of bookmarked
subreddits, and the scrollable **feed** of post cards.

Three choices worth calling out:

1. **go-widgets, not HTML.** The cards, score badges, sidebar and topbar are
   painted by the go-widgets painter into an RGBA buffer, exactly as in a native
   window — the WebView is just a surface. Text is anti-aliased TrueType
   rendered at device resolution, so it stays crisp at any zoom or on a Retina
   display.
2. **CGO=0 everywhere.** The WKWebView, NSWindow and NSApplication are driven
   through the Objective-C runtime via
   [purego](https://github.com/ebitengine/purego). The shipped binary links only
   system libraries; AppKit/WebKit are `dlopen`ed at runtime.
3. **No socket.** Instead of a loopback HTTP server, the default transport
   registers a private `reader://` URL scheme and answers every request
   in-process from the same `http.Handler`. Nothing on the machine can reach the
   app's content, and no port is consumed. Pass `-http` to fall back to a
   `127.0.0.1` server (used off macOS and for automated testing).

## Running it

```sh
task run                       # == go run .   (front page)
go run . -sr golang -sort top  # start on a specific feed
task app                       # build dist/Reddit Reader.app
task demo                      # built-in sample feeds, no network
```

For live Reddit from a datacenter IP, set OAuth credentials — see
[Authentication](oauth.md).

## Flags & environment

| Flag | Env | Meaning |
|------|-----|---------|
| `-sr` | `READER_SUBREDDIT` | subreddit (empty = front page) |
| `-sort` | `READER_SORT` | `hot`/`new`/`top`/`rising`/`controversial`/`best` |
| `-demo` | | serve a built-in sample feed, no network |
| `-http` | | use a loopback TCP server instead of the URL-scheme transport |
| `-no-window` | | open the default browser instead of a native window |
| `-serve-only` | | run headless over TCP and print the URL |
| | `READER_OAUTH_CLIENT_ID` / `_SECRET` | app-only OAuth |
| | `READER_OAUTH_USERNAME` / `_PASSWORD` | script-grant OAuth |

## Build layout

- `internal/ui` — the go-widgets scene: layout, hit-testing, rendering. Pure,
  native-testable (100% coverage), no build tag.
- `cmd/front` — the `GOOS=js GOARCH=wasm` entry point (syscall/js glue).
- `internal/server` — static bundle + `/api` proxy (100% coverage).
- `internal/webview` — the purego WKWebView + `WKURLSchemeHandler` bridge.

The wasm bundle is a build artifact; `scripts/build-wasm.sh` (run by every
`task` target) produces `internal/server/assets/reader.wasm`.
