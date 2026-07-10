# Authentication (OAuth)

Reddit increasingly blocks the anonymous `.json` endpoints with a **403**,
always from datacenter IPs and sometimes from residential ones. OAuth is the
reliable path for any server-side or datacenter caller. Register an app at
<https://www.reddit.com/prefs/apps> to get a **client id** and **secret**.

The client fetches and caches the bearer token automatically, refreshing it
before expiry, and routes requests through `oauth.reddit.com`.

## Application-only (`client_credentials`)

Reads public content with no user account — the right choice for most
back-ends:

```go
c := reddit.NewClient(
	reddit.WithUserAgent("myapp/1.0 (by /u/you)"),
	reddit.WithOAuth(clientID, clientSecret),
)
```

## Script grant (`password`)

Acts as a specific user (the account that owns the app):

```go
c := reddit.NewClient(
	reddit.WithUserAgent("myapp/1.0 (by /u/you)"),
	reddit.WithOAuthScript(clientID, clientSecret, username, password),
)
```

## In the reader app

The [reader](reader.md) reads the same credentials from the environment, so you
never pass secrets on the command line:

| Env | Grant |
| --- | --- |
| `READER_OAUTH_CLIENT_ID` / `READER_OAUTH_CLIENT_SECRET` | application-only |
| `READER_OAUTH_USERNAME` / `READER_OAUTH_PASSWORD` | script grant |

```sh
READER_OAUTH_CLIENT_ID=…  READER_OAUTH_CLIENT_SECRET=…  task run
```

!!! tip "User-Agent still matters"
    Even with OAuth, send a descriptive `User-Agent` (via `WithUserAgent`);
    Reddit rate-limits generic ones regardless of authentication.
