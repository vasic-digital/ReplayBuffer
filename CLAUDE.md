# CLAUDE.md - ReplayBuffer Module

## Overview

`digital.vasic.replaybuffer` is a generic, reusable Go module for recording and replaying successful action sequences. Backed by SQLite for persistence across process restarts.

**Module**: `digital.vasic.replaybuffer` (Go 1.24+)

## Build & Test

```bash
go build ./...
go test ./... -count=1 -race
go vet ./...
```

Requires CGO enabled for SQLite (`github.com/mattn/go-sqlite3`).

## Code Style

- Standard Go conventions, `gofmt` formatting
- Imports grouped: stdlib, third-party, internal (blank line separated)
- Line length target 80 chars (100 max)
- Naming: `camelCase` private, `PascalCase` exported
- Errors: always check, wrap with `fmt.Errorf("...: %w", err)`
- Tests: table-driven where appropriate, `testify`, naming `Test<Struct>_<Method>_<Scenario>`
- SPDX headers on every .go file

## Package Structure

| Package | Purpose |
|---------|---------|
| `pkg/replay` | ReplayBuffer with SQLite persistence, in-memory cache, hash-based lookup |

## Key Types

- `replay.ReplayBuffer` -- Thread-safe buffer with SQLite backing store
- `replay.ActionSequence` -- Recorded sequence with platform, actions, success count
- `replay.RecordedAction` -- Single action with type, value, and screen hash

## Design Patterns

- **In-Memory Cache + SQLite**: All data loaded on startup for fast matching, persisted for durability
- **Copy-on-Read**: `FindMatch` and `All` return copies to prevent data races
- **Double-Close Safety**: `sync.Once` ensures Close is idempotent

## Constraints

- **No CI/CD pipelines** -- no GitHub Actions, no GitLab CI
- **Generic library** -- no application-specific logic
- **Requires CGO** -- SQLite driver needs CGO enabled

## Commit Style

Conventional Commits: `feat(replay): add sequence expiration`
