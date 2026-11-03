# gmail-cleaner

Go CLI that runs YAML-defined rules against a personal Gmail account to trash, delete, archive, or label messages.

## Build & Run

```bash
go build -o gmail-cleaner ./cmd/gmail-cleaner
go install ./cmd/gmail-cleaner
```

No test suite exists yet. Validate config without hitting Gmail:

```bash
gmail-cleaner rules validate -c rules.yaml
```

## Project Layout

```
cmd/gmail-cleaner/main.go    — CLI entrypoint, all cobra commands defined here
internal/auth/auth.go        — OAuth 3-legged flow, token caching (~/.config/gmail-cleaner/)
internal/config/config.go    — YAML rule loading + validation
internal/gmailclient/client.go — Thin wrapper over gmail/v1 (Search, Trash, Delete, Label, Filter, etc.)
internal/engine/engine.go    — Applies rules to messages; honors dry-run and safety flags
rules.example.yaml           — Example rules config (committed); rules.yaml is gitignored
```

## Key Conventions

- **Single-file packages**: each `internal/` package is one `.go` file.
- **Cobra CLI**: all subcommands are defined inline in `main.go` (not separate files).
- **Config directory**: OAuth credentials + token live in `~/.config/gmail-cleaner/` by default, overridable via `--config-dir`.
- **Safety guards**: `action: delete` requires `--allow-delete` flag. Always encourage `--dry-run` first.
- **Batch operations**: uses `batchModify` / `batchDelete` (1000/batch) where available; per-message fan-out with bounded concurrency (5) for trash/untrash.
- **Logging**: `log/slog` to stderr; user-facing output to stdout via `fmt`.

## Sensitive Files (never commit)

- `credentials.json` — OAuth client secret
- `token.json` — cached refresh token
- `rules.yaml` — may contain personal email addresses
- `CLEANUP.md` — personal inbox analysis

## Commands

| Command | Purpose |
|---|---|
| `auth` | Interactive OAuth flow, caches token |
| `run` | Execute rules (supports `--dry-run`, `--allow-delete`, `--rule`) |
| `rules validate` | Parse/validate rules YAML without touching Gmail |
| `stats` | Label message counts (`--label`, `--json`) |
| `count <query>` | Count messages matching a Gmail query |
| `top-senders` | Rank senders by frequency (`--query`, `--limit`, `--sample`) |
| `label` | Label existing messages + install Gmail filter (`--from`, `--label`) |
| `purge` | Trash or permanently delete messages by label (`--action`, `--in-trash`, `--all`) |
| `untrash` | Restore messages from Trash (`--label`, `--from`, `--all`) |

## Dependencies

- `github.com/spf13/cobra` — CLI framework
- `golang.org/x/oauth2` + `google.golang.org/api` — Gmail API + OAuth
- `gopkg.in/yaml.v3` — rule config parsing
