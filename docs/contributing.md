# Contributing

go-reddit is BSD-3-Clause and welcomes contributions to any of its repos.

## Ground rules

- **Pure Go, `CGO_ENABLED=0`.** No cgo, no vendored C. The client is
  stdlib-only; the reader reaches the platform through
  [purego](https://github.com/ebitengine/purego), not cgo.
- **100% test coverage** is the target and a CI gate — including error
  branches. Add tests with every change.
- **`gofmt` + `go vet` clean.** CI runs both, plus the test suite across the six
  64-bit Go targets (amd64, arm64, riscv64, loong64, ppc64le, s390x).
- **English only** for all issues, PRs, commits and docs.

## Building & testing

```sh
# The client
git clone https://github.com/go-reddit/reddit && cd reddit
go test ./... -cover

# The reader (macOS)
git clone https://github.com/go-reddit/reader && cd reader
task test          # native tests (no display needed)
task run           # launch the app
```

## Docs

This site is MkDocs Material, versioned with [mike](https://github.com/jimporter/mike):

```sh
python -m venv .venv && . .venv/bin/activate
pip install -r requirements.txt
mkdocs serve
```

Open a PR against `main`; the `docs` workflow republishes the `latest` version
on merge.
