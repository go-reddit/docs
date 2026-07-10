<p align="center"><img src="https://raw.githubusercontent.com/go-reddit/brand/main/social/go-reddit.png" alt="go-reddit/docs" width="720"></p>

# go-reddit/docs

Documentation for the **go-reddit** organization — the pure-Go Reddit client
[`reddit`](https://github.com/go-reddit/reddit) and the native macOS
[`reader`](https://github.com/go-reddit/reader) app. Built with
[MkDocs Material](https://squidfunk.github.io/mkdocs-material/) and versioned
with [mike](https://github.com/jimporter/mike).

Served at <https://go-reddit.github.io/docs/>.

`.github/workflows/docs.yml` builds the site and publishes the versioned docs to
the `gh-pages` branch on every push to `main`.

## Local preview

```bash
python -m venv .venv && . .venv/bin/activate
pip install -r requirements.txt
mkdocs serve      # http://127.0.0.1:8000
```
