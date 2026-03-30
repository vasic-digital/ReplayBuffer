# AGENTS.md - Multi-Agent Coordination Guide

## Overview

This document provides guidance for AI agents working with the `digital.vasic.replaybuffer` module.

## Module Identity

- **Module path**: `digital.vasic.replaybuffer`
- **Language**: Go 1.24+
- **Dependencies**: `github.com/mattn/go-sqlite3` (SQLite driver), `github.com/stretchr/testify` (tests)
- **Scope**: Generic, reusable action replay buffer. No application-specific logic.

## Package Responsibilities

| Package | Owner Concern | Agent Must Not |
|---------|--------------|----------------|
| `pkg/replay` | Sequence recording, hash-based lookup, SQLite persistence | Add application-specific logic, break CGO compatibility |

## Coordination Rules

### 1. Thread Safety Invariants

Every exported method on `ReplayBuffer` is safe for concurrent use. Agents must:

- Never remove mutex protection from shared state.
- Never introduce a public method that requires external synchronization.
- Always run `go test -race` after changes.

### 2. Interface Contracts

The `ReplayBuffer` API is a stability boundary. Breaking changes require explicit human approval:

- `NewReplayBuffer(dbPath)` constructor signature
- `Record(seq)` / `FindMatch(hash, platform)` behavior
- `ActionSequence` and `RecordedAction` struct fields

### 3. Database Schema

The `replay_sequences` table schema is a compatibility contract. Agents must:

- Never remove columns.
- Add new columns with DEFAULT values only.
- Use `CREATE TABLE IF NOT EXISTS` and `CREATE INDEX IF NOT EXISTS`.

### 4. Test Requirements

- All tests use `testify/assert` and `testify/require`.
- Test naming convention: `Test<Struct>_<Method>_<Scenario>`.
- Tests use `t.TempDir()` for database paths -- never write to fixed paths.
- Race detector must pass: `go test ./... -race`.

## Agent Workflow

### Before Making Changes

```bash
go build ./...
go test ./... -count=1 -race
```

### After Making Changes

```bash
gofmt -w .
go vet ./...
go test ./... -count=1 -race
```

### Commit Convention

```
<type>(<package>): <description>

# Examples:
feat(replay): add sequence expiration support
fix(replay): handle corrupted JSON in loadAll
test(replay): add persistence edge case coverage
```

## Boundaries

### What Agents May Do

- Fix bugs in any package.
- Add tests for uncovered code paths.
- Refactor internals without changing exported APIs.
- Add new exported methods that extend existing types.
- Update documentation to match code.

### What Agents Must Not Do

- Break existing exported interfaces or method signatures.
- Remove thread safety guarantees.
- Add application-specific logic (this is a generic library).
- Introduce new external dependencies without human approval.
- Modify `go.mod` without explicit instruction.
- Change the SQLite schema in backwards-incompatible ways.

## Key Files

| File | Purpose |
|------|---------|
| `pkg/replay/buffer.go` | All production code |
| `pkg/replay/buffer_test.go` | All tests |
| `go.mod` | Module definition |
| `README.md` | User-facing documentation |
| `CLAUDE.md` | Agent build/test guidance |
