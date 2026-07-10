# Roadmap

## Done

- **`reddit` — the client.** Subreddit, front-page and comment listings across
  every sort (`hot`/`new`/`top`/`rising`/`controversial`/`best`) and time
  window; cursor paging; recursively nested comment replies; anonymous and
  OAuth (app-only + script) with automatic bearer-token fetch/refresh over
  `oauth.reddit.com`; typed `*APIError` status codes. Stdlib-only,
  `CGO_ENABLED=0`, **100% coverage including every error branch**.
- **`reader` — the macOS app.** Full feed UI (topbar sort tabs,
  bookmarked-subreddit sidebar, scrollable post cards) painted by go-widgets and
  compiled to wasm; in-process `WKURLSchemeHandler` transport over a private
  `reader://` scheme (no socket, no port); a `-http` loopback fallback; window
  resize re-layout and ⌘+/⌘-/⌘0 zoom; a `-demo` offline mode; a
  double-clickable `.app` bundle. `CGO_ENABLED=0` end to end, native tests at
  100% coverage.
- **Org conformance.** Hugo landing with a light/dark/system theme toggle,
  MkDocs Material + mike docs, brand assets, all repos CI-green across the six
  64-bit Go targets.

## Planned / by design

- **Ruby binding.** A [`go-ruby-reddit`](https://github.com/go-ruby-reddit)
  binding surfaces the client to embedded Ruby (rbgo) — downstream of this org.
- **More endpoints.** User and search listings, and write actions, as needs
  arise — always behind OAuth.
- **Cross-platform reader shells.** The go-widgets UI and the client are
  platform-agnostic; a non-macOS window host (the `-http` transport already runs
  the UI in any browser) is the natural next surface.
