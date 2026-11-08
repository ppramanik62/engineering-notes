# 4. wal

How a database promises that a committed write survives a crash: an append-only log of checksummed records, `fsync`, replay up to a torn tail, and what a failed `fsync` does.

My implementation: [`wal` in onyxdb](https://github.com/PPRAMANIK62/onyxdb/tree/main/crates/wal).

## References

1. [CMU 15-445 notes: Database Logging](https://15445.courses.cs.cmu.edu/spring2026/notes/22-logging.pdf) and [Database Recovery](https://15445.courses.cs.cmu.edu/spring2026/notes/23-recovery.pdf): steal and force, shadow paging, write-ahead logging, checkpoints and ARIES, as taught in class.
2. [Atomic Commit In SQLite](https://www.sqlite.org/atomiccommit.html) (SQLite docs): how a rollback journal survives a crash at any moment, step by step.
3. [LevelDB log format](https://github.com/google/leveldb/blob/main/doc/log_format.md) (LevelDB docs): records split across 32 KiB blocks, each fragment with a checksum and a type.
4. [Ensuring data reaches disk](https://lwn.net/Articles/457667/) (Jeff Moyer, LWN, 2011): the layers between `write` and the disk, and the calls that make a file durable.
