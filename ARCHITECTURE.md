# Architecture -- ReplayBuffer

## Purpose

Thread-safe action replay buffer backed by SQLite. Records successful navigation sequences during autonomous QA sessions and replays them when encountering a matching screen state. Sequences are persisted and survive process restarts.

## Structure

```
pkg/
  replay/   ReplayBuffer with SQLite persistence, in-memory cache, hash-based lookup
```

## Key Components

- **`replay.ReplayBuffer`** -- Core type: Record, FindMatch, MarkSuccess, Delete, All, Len, Close. All methods thread-safe via mutex
- **`replay.ActionSequence`** -- Recorded sequence with ID, Platform, Actions, CreatedAt, SuccessCount
- **`replay.RecordedAction`** -- Single action with Type, Value, and ScreenHash
- **`replay.ScreenHash(screenshot)`** -- SHA-256 hash of screenshot bytes for matching

## Data Flow

```
Record(sequence) -> upsert into SQLite + in-memory cache (by ID)
    |
FindMatch(hash, platform) -> scan in-memory cache for sequences where
    first action's ScreenHash matches and platform matches
    -> multiple matches? return highest SuccessCount
    -> return copy (prevent data races)
    |
MarkSuccess(seqID) -> increment SuccessCount in SQLite + cache
```

## Dependencies

- `github.com/mattn/go-sqlite3` -- SQLite driver (requires CGO)
- `github.com/stretchr/testify` -- Test assertions

## Testing Strategy

Table-driven tests with `testify` and race detection. Tests use temporary database files. Tests cover Record/FindMatch/MarkSuccess lifecycle, platform filtering, best-match selection by success count, concurrent access, and Close idempotency.
