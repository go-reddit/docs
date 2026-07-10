# go-reddit documentation

**A pure-Go (CGO=0) Reddit stack** — a stdlib-only client for Reddit's JSON API
and a native macOS reader built on top of it, both with **zero cgo**.

- [`go-reddit/reddit`](https://github.com/go-reddit/reddit) — the API client.
  Module path `github.com/go-reddit/reddit`. Dependency-free (standard library
  only), so it builds for every Go target with `CGO_ENABLED=0`.
- [`go-reddit/reader`](https://github.com/go-reddit/reader) — the macOS reader.
  Module path `github.com/go-reddit/reader`. Its UI is drawn by
  [go-widgets](https://github.com/go-widgets/toolkit), compiled to WebAssembly,
  and hosted in a WKWebView opened from Go through
  [purego](https://github.com/ebitengine/purego) — no cgo.

!!! success "Status: client complete, reader shipping on macOS"
    `reddit` covers **subreddit, front-page and comment listings** across every
    sort (`hot`/`new`/`top`/`rising`/`controversial`/`best`) and time window,
    read **anonymously or over OAuth**, with automatic bearer-token
    fetch/refresh and typed `*APIError` status codes — at **100% coverage,
    every error branch included**. `reader` renders the full feed UI with
    go-widgets, serves the wasm and proxies `/api` **in-process** through a
    `WKURLSchemeHandler` (no socket, no port), and builds a double-clickable
    `.app`. Both are `gofmt` + `go vet` clean and CI-green across the six 64-bit
    Go targets.

## Quick taste

```go
package main

import (
	"context"
	"fmt"

	"github.com/go-reddit/reddit"
)

func main() {
	c := reddit.NewClient(reddit.WithUserAgent("myapp/1.0 (by /u/you)"))
	page, err := c.Subreddit(context.Background(), "golang", reddit.SortHot,
		reddit.ListingOptions{Limit: 25})
	if err != nil {
		panic(err)
	}
	for _, p := range page.Posts {
		fmt.Printf("%5d  %s  (u/%s)\n", p.Score, p.Title, p.Author)
	}
}
```

```sh
# The macOS reader, on a specific feed:
go run github.com/go-reddit/reader@latest -sr golang -sort top
```

## Repositories

| Repo | What it is |
| --- | --- |
| [`reddit`](https://github.com/go-reddit/reddit) | the API client — listings, comments, OAuth, typed `*APIError`; stdlib-only, `CGO_ENABLED=0` |
| [`reader`](https://github.com/go-reddit/reader) | the native macOS reader — go-widgets UI in wasm, hosted in a purego WKWebView |
| [`docs`](https://github.com/go-reddit/docs) | this documentation site (MkDocs Material, versioned with mike) |
| [`go-reddit.github.io`](https://github.com/go-reddit/go-reddit.github.io) | the organization landing page (Hugo) |
| [`brand`](https://github.com/go-reddit/brand) | logo and brand assets |

## Principles

- **Pure Go, `CGO_ENABLED=0`** — trivial cross-compilation, a single static
  binary, no C toolchain. The client is stdlib-only; the reader `dlopen`s
  AppKit/WebKit at runtime via purego.
- **Anonymous or OAuth**, with the reliable OAuth path for datacenter callers
  that Reddit now blocks anonymously.
- **Typed, honest errors** — a `*APIError` carrying the HTTP status.
- **100% test coverage** is the target, enforced as a CI gate.

## Where to go next

- [The client (reddit)](client.md) — `NewClient`, listings, comments, sorts and
  options, and the `*APIError` type.
- [Authentication (OAuth)](oauth.md) — app-only and script grants, and why
  OAuth is the reliable path.
- [The reader app](reader.md) — the go-widgets + WKWebView architecture and the
  `reader://` transport.
- [Roadmap](roadmap.md) — what is done and what is planned.
