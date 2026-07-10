# The client — `go-reddit/reddit`

A dependency-free (stdlib-only, `CGO_ENABLED=0`) Go client for Reddit's public
JSON API. Module path `github.com/go-reddit/reddit`.

```sh
go get github.com/go-reddit/reddit
```

## Creating a client

```go
// A descriptive User-Agent is required; Reddit rate-limits generic ones.
c := reddit.NewClient(reddit.WithUserAgent("myapp/1.0 (by /u/you)"))
```

`NewClient` takes functional options:

| Option | Effect |
| --- | --- |
| `WithUserAgent(ua)` | set the required, descriptive `User-Agent` |
| `WithOAuth(id, secret)` | application-only (`client_credentials`) grant |
| `WithOAuthScript(id, secret, user, pass)` | script (`password`) grant, acting as a user |
| `WithHTTPClient(hc)` | supply your own `*http.Client` (timeouts, transport) |

See [Authentication](oauth.md) for the OAuth options in detail.

## Listings

```go
page, err := c.Subreddit(context.Background(), "golang", reddit.SortHot,
	reddit.ListingOptions{Limit: 25})
for _, p := range page.Posts {
	fmt.Printf("%5d  %s  (u/%s)\n", p.Score, p.Title, p.Author)
}

// Page forward with the returned cursor:
next, _ := c.Subreddit(context.Background(), "golang", reddit.SortHot,
	reddit.ListingOptions{Limit: 25, After: page.After})
```

| Method | Endpoint |
| --- | --- |
| `Subreddit(ctx, name, sort, opts)` | `r/<name>/<sort>.json` |
| `Frontpage(ctx, sort, opts)` | `/<sort>.json` (logged-out front page) |
| `Comments(ctx, subreddit, id, opts)` | `r/<sub>/comments/<id>.json` |

**Sorts:** `SortHot`, `SortNew`, `SortTop`, `SortRising`, `SortControvers`,
`SortBest`. **Time windows** for top/controversial: `TimeHour`, `TimeDay`,
`TimeWeek`, `TimeMonth`, `TimeYear`, `TimeAll`.

`ListingOptions` carries `Limit`, the `After` / `Before` cursors, and the
`Time` window.

## Comments

```go
res, _ := c.Comments(context.Background(), "golang", "abc123",
	reddit.ListingOptions{Limit: 100})
fmt.Println(res.Post.Title)
for _, cm := range res.Comments {   // Replies nested recursively
	fmt.Printf("u/%s: %s\n", cm.Author, cm.Body)
}
```

Each comment's `Replies` field holds the nested child comments, recursively.

## Errors

Non-2xx responses surface as a typed `*reddit.APIError` carrying the status
code, so callers can distinguish a 429 (rate limit) from a 403 (blocked) or a
404 (gone):

```go
_, err := c.Subreddit(ctx, "golang", reddit.SortHot, reddit.ListingOptions{})
var apiErr *reddit.APIError
if errors.As(err, &apiErr) {
	switch apiErr.StatusCode {
	case 429:
		// back off
	case 403:
		// anonymous access blocked — authenticate (see OAuth)
	}
}
```

!!! warning "Anonymous access is increasingly blocked"
    Reddit returns **403** for anonymous `.json` requests from datacenter IPs
    (and sometimes residential ones). For anything server-side, use
    [OAuth](oauth.md) — the client then routes through `oauth.reddit.com` and
    works everywhere.
