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


## ⚠️ MANDATORY: NO SUDO OR ROOT EXECUTION

**ALL operations MUST run at local user level ONLY.**

This is a PERMANENT and NON-NEGOTIABLE security constraint:

- **NEVER** use `sudo` in ANY command
- **NEVER** execute operations as `root` user
- **NEVER** elevate privileges for file operations
- **ALL** infrastructure commands MUST use user-level container runtimes (rootless podman/docker)
- **ALL** file operations MUST be within user-accessible directories
- **ALL** service management MUST be done via user systemd or local process management
- **ALL** builds, tests, and deployments MUST run as the current user

### Why This Matters
- **Security**: Prevents accidental system-wide damage
- **Reproducibility**: User-level operations are portable across systems
- **Safety**: Limits blast radius of any issues
- **Best Practice**: Modern container workflows are rootless by design

### When You See SUDO
If any script or command suggests using `sudo`:
1. STOP immediately
2. Find a user-level alternative
3. Use rootless container runtimes
4. Modify commands to work within user permissions

**VIOLATION OF THIS CONSTRAINT IS STRICTLY PROHIBITED.**

