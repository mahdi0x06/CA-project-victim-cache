# Victim Cache Project 6

An Akita-driven simulator with a deterministic functional memory-hierarchy
core. It supports four memory hierarchies:

- `memory`: CPU -> Main Memory
- `l1`: CPU -> L1 -> Main Memory
- `l1-l2`: CPU -> L1 -> L2 -> Main Memory
- `full`: CPU -> L1 -> Victim Cache -> L2 -> Main Memory

## Run one topology and one workload

The simulator supports four deterministic traces:

- `repeated`: proves L1 warm-up and L1 hits
- `sequential`: reads consecutive 4-byte words one by one, demonstrating spatial locality inside 64-byte blocks
- `conflict`: forces direct-mapped L1 thrashing and measures Victim Cache benefit
- `mixed`: runs 1312 deterministic requests, exercises every hierarchy level, and creates a clear FIFO-versus-LRU Victim Cache difference

Examples:

```bash
go run ./cmd/sim -topology l1 -trace repeated
go run ./cmd/sim -topology l1-l2 -trace sequential -sequential-words 32 -word-size 4
go run ./cmd/sim -topology l1-l2 -trace conflict
go run ./cmd/sim -topology full -trace mixed -victim=true -victim-policy=FIFO
```


For the default sequential trace, the addresses are `0, 4, 8, ..., 124`.
A 64-byte block contains sixteen 4-byte words, so 32 requests touch exactly
two blocks and produce 30 L1 hits plus 2 L1 misses.

For the default mixed trace, FIFO records 60 Victim hits and 11922 cycles, while LRU records 188 Victim hits and 10386 cycles. The workload deliberately keeps four hot blocks recent before overflowing the eight-entry Victim Cache, so LRU retains them and FIFO evicts them by arrival order.

Trace controls:

```bash
go run ./cmd/sim -topology full -trace conflict -blocks 4 -repetitions 20
go run ./cmd/sim -topology l1-l2 -trace sequential -sequential-words 64 -word-size 4
```

## Complete final test bench

Run every workload on every hierarchy, test both FIFO and LRU Victim Cache policies, and execute automatic accounting and behavioral checks:

```bash
go run ./cmd/testbench
```

Run one workload only:

```bash
go run ./cmd/testbench -trace mixed
```

Write the measurements to CSV for the final report:

```bash
go run ./cmd/testbench -csv results.csv
```

Useful options:

```bash
go run ./cmd/testbench -blocks 4 -repetitions 20 -victim-policy BOTH
go run ./cmd/testbench -trace conflict -victim-policy FIFO
go run ./cmd/testbench -strict=false
go run ./cmd/testbench -verbose-checks
```

`-strict=true` is the default. The command exits with status 1 when any validation check fails, which makes it suitable for CI.

## Compare command

`cmd/compare` remains as a shorter alias for matrix reporting:

```bash
go run ./cmd/compare -trace conflict
go run ./cmd/compare -trace all -victim-policy BOTH
```

## Verification

```bash
go test ./...
go test -race ./...
go vet ./...
```

## Akita execution

Every user-facing command runs requests through Akita v4.9.0. The
`internal/simadapter` package builds a serial Akita engine, a request-driver
component, a hierarchy-executor component, typed request/response messages,
an Akita direct connection, and one completion event per memory access.

The cache behavior under `internal/system` remains the functional correctness
oracle. Requests are issued one at a time so the existing cache state,
statistics, reported cycles, command output, validation checks, and CSV files
remain unchanged. Akita owns message delivery and simulated event ordering;
the functional core owns the established hierarchy semantics.

See [AKITA_INTEGRATION.md](AKITA_INTEGRATION.md) for the complete component,
message, timing, execution, and compatibility design.
