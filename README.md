# digital.vasic.replaybuffer

A thread-safe action replay buffer backed by SQLite. Records successful navigation sequences and replays them when encountering a matching screen state. Sequences are persisted and survive process restarts.

## Installation

```bash
go get digital.vasic.replaybuffer
```

Requires CGO for SQLite (`github.com/mattn/go-sqlite3`).

## Quick Start

```go
package main

import (
    "fmt"

    "digital.vasic.replaybuffer/pkg/replay"
)

func main() {
    // Open (or create) a replay database.
    rb, err := replay.NewReplayBuffer("./replay.db")
    if err != nil {
        panic(err)
    }
    defer rb.Close()

    // Record a successful action sequence.
    err = rb.Record(replay.ActionSequence{
        ID:       "login-flow-1",
        Platform: "android",
        Actions: []replay.RecordedAction{
            {Type: "tap", Value: "500,300", ScreenHash: "abc123"},
            {Type: "type", Value: "user@example.com", ScreenHash: "def456"},
            {Type: "tap", Value: "500,600", ScreenHash: "ghi789"},
        },
    })
    if err != nil {
        panic(err)
    }

    // Look up a known sequence by screen hash.
    screenshot := captureScreen() // your capture logic
    hash := replay.ScreenHash(screenshot)

    match := rb.FindMatch(hash, "android")
    if match != nil {
        fmt.Printf("Found replay: %s (%d actions)\n",
            match.ID, len(match.Actions))

        // After successful replay, mark it.
        _ = rb.MarkSuccess(match.ID)
    }

    // Check stats.
    fmt.Printf("Stored sequences: %d\n", rb.Len())
}
```

## API Reference

### ReplayBuffer

The core type. All methods are safe for concurrent use.

| Method | Description |
|--------|-------------|
| `NewReplayBuffer(dbPath string)` | Open/create SQLite-backed buffer |
| `Record(seq ActionSequence) error` | Save a sequence (upsert by ID) |
| `FindMatch(hash, platform string) *ActionSequence` | Find best match by screen hash |
| `MarkSuccess(seqID string) error` | Increment success counter |
| `Delete(seqID string) error` | Remove a sequence |
| `All() []ActionSequence` | Get copy of all sequences |
| `Len() int` | Count stored sequences |
| `Close() error` | Release database connection (safe to call twice) |

### Utility Functions

| Function | Description |
|----------|-------------|
| `ScreenHash(screenshot []byte) string` | SHA-256 hash of screenshot data |

### ActionSequence

| Field | Type | Description |
|-------|------|-------------|
| `ID` | `string` | Unique identifier |
| `Platform` | `string` | Target platform (e.g. "android", "web") |
| `Actions` | `[]RecordedAction` | Ordered list of actions |
| `CreatedAt` | `time.Time` | When first recorded |
| `SuccessCount` | `int` | Number of successful replays |

### RecordedAction

| Field | Type | Description |
|-------|------|-------------|
| `Type` | `string` | Action type (e.g. "tap", "type", "dpad_down") |
| `Value` | `string` | Action value (coordinates, text, etc.) |
| `ScreenHash` | `string` | SHA-256 hash of the pre-action screenshot |

### Matching Algorithm

`FindMatch` looks for sequences whose first action's `ScreenHash` matches the given hash and platform. When multiple sequences match, the one with the highest `SuccessCount` is returned. Returns a copy to prevent data races.

### Persistence

Sequences are stored in a SQLite database with WAL journaling and a 5-second busy timeout. The `start_hash` column is indexed for fast lookups. All data is loaded into memory on startup for fast matching.

## License

Apache-2.0
