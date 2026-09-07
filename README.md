# DOCS-DB - High-Performance Key-Value Store

A lightweight, persistent key-value database optimized for write-heavy workloads, built from scratch in C++20.

## Architecture

```
                    +-----------------+
                    |   REPL / Server |   (Entry Points)
                    +--------+--------+
                             |
                    +--------v--------+
                    |    LSM Tree      |   (Orchestrator)
                    +--+-----+-----+--+
                       |     |     |
            +----------+  +--+--+  +----------+
            |  Memtable |  | WAL |  | Segments |
            |(RB-Tree)  |  |     |  | (Levels) |
            +----------+  +-----+  +-----+----+
                                      |
                              +-------+-------+
                              |  SST Files    |
                              | (Bloom Filter |
                              |  Sparse Index)|
                              +---------------+
```

## Key Components

- **LSM Tree** - Write-optimized storage with leveled compaction
- **Memtable** - In-memory Red-Black Tree (64MB) as write buffer
- **Write-Ahead Log** - Durability via append-only log with crash recovery
- **Bloom Filter** - 3-hash probabilistic filter to skip unnecessary disk reads
- **Sparse Index** - Every 128th key indexed for fast point lookups
- **RESP-2 Server** - Redis-compatible async server using asyncio + uvloop

## Build & Run

### Part-A (C++ Core)
```bash
cd Part-A
chmod +x setup.sh build.sh
./setup.sh
./build.sh

# Run tests
cd build/test
./bloom_test && ./rb_tree_test && ./lsm_tree_test && ./wal_test && ./docs_db_test

# Run REPL
cd build
./repl ../repl.txt
```

### Part-B (Python Server)
```bash
cd Part-B
pip install -r requirements.txt
./python_wrapper.sh
export LD_LIBRARY_PATH=../Part-A/build/src:$LD_LIBRARY_PATH
python src/server.py
```

## Benchmark Results

Tested with `redis-benchmark` - 1M requests, 1000 parallel connections:

| Operation | p99 Latency | Throughput |
|-----------|-------------|------------|
| GET | ~25 ms | ~44K req/s |
| SET | ~28 ms | ~44K req/s |

## Tech Stack

C++20, Boost, Google Test, Python, asyncio, uvloop, RESP-2 protocol
