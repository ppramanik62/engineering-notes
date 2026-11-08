# Database references

The sources behind each part of a database, by topic. A section gets added when I read it.

## Virtual file system

Everything a database stores ends up in a file, and the calls that put it there promise less than they look like they do. Built as [`vfs`](https://github.com/PPRAMANIK62/onyxdb/tree/main/crates/vfs) in onyxdb.

1. [The SQLite OS Interface or "VFS"](https://www.sqlite.org/vfs.html) (SQLite docs): how a real database puts every file operation behind one interface, and why.
2. [Files are hard](https://danluu.com/file-consistency/) (Dan Luu): why "just write to a file" is not safe.
3. [All File Systems Are Not Created Equal](https://www.usenix.org/system/files/conference/osdi14/osdi14-paper-pillai.pdf) (OSDI '14): real databases and version-control tools that lose data when the power fails, and why.
4. [Can Applications Recover from fsync Failures?](https://www.usenix.org/system/files/atc20-rebello.pdf) (ATC '20): what ext4, XFS and Btrfs do when `fsync` fails, and how none of PostgreSQL, LMDB, LevelDB, SQLite or Redis handles it safely in every case.

## Page structure

A file is a flat run of bytes, so a database cuts it into fixed-size pages and gives each one a header, a checksum and a way to pack rows into it. Built as [`pagefile`](https://github.com/PPRAMANIK62/onyxdb/tree/main/crates/pagefile) in onyxdb.

1. [CMU 15-445 notes: Database Storage I](https://15445.courses.cs.cmu.edu/spring2026/notes/03-storage1.pdf) and [II](https://15445.courses.cs.cmu.edu/spring2026/notes/05-storage2.pdf): files, pages, slotted pages, tuple layout and large values, then the other ways to store tuples, as taught in class.
2. [SQLite Database File Format](https://www.sqlite.org/fileformat2.html) (SQLite docs): a real, fully specified format with a header, page types, cell pointers, freeblocks, overflow chains and a freelist.
3. [PostgreSQL Database Page Layout](https://www.postgresql.org/docs/current/storage-page-layout.html) (PostgreSQL docs): the same problem solved with line pointers and a page checksum, to compare against SQLite's choices.
4. [A Painless Guide to CRC Error Detection Algorithms](http://www.ross.net/crc/download/crc_v3.txt) (Ross Williams): how a CRC works, from polynomial division up to a table-driven version.

## Buffer manager

Disk is slow, so pages live in memory and go back to the file later. Which page to throw out, how to stop one being thrown out while it's in use, and why the order of the writes matters more than their speed. Built as [`bufpool`](https://github.com/PPRAMANIK62/onyxdb/tree/main/crates/bufpool) in onyxdb.

1. [CMU 15-445 notes: Memory Management](https://15445.courses.cs.cmu.edu/spring2026/notes/04-bufferpool.pdf) and [Hash Tables](https://15445.courses.cs.cmu.edu/spring2026/notes/07-hashtables.pdf): frames, the page table, pinning, replacement policies, locks against latches, and the hash table behind the page table, as taught in class.
2. [The Internals of PostgreSQL, ch. 8: Buffer Manager](https://www.interdb.jp/pg/pgsql08.html) (Hironobu Suzuki): a real buffer pool, with buffer descriptors, a partitioned buffer table, pinning, clock-sweep eviction and ring buffers.
3. [Are You Sure You Want to Use MMAP in Your DBMS?](https://db.cs.cmu.edu/papers/2022/cidr2022-p13-crotty.pdf) (Crotty, Leis and Pavlo, CIDR '22): why databases don't let the operating system manage their pages.
4. [The LRU-K Page Replacement Algorithm for Database Disk Buffering](https://www.cs.cmu.edu/~christos/courses/721-resources/p297-o_neil.pdf) (O'Neil, O'Neil and Weikum, SIGMOD '93): evict by the time of the K-th most recent reference, so one scan can't flush the pool.

## Write-ahead log

Once a write is committed it has to still be there after the power goes out. An append-only log of checksummed records, `fsync`, replay up to a torn tail, and what a failed `fsync` does. Built as [`wal`](https://github.com/PPRAMANIK62/onyxdb/tree/main/crates/wal) in onyxdb.

1. [CMU 15-445 notes: Database Logging](https://15445.courses.cs.cmu.edu/spring2026/notes/22-logging.pdf) and [Database Recovery](https://15445.courses.cs.cmu.edu/spring2026/notes/23-recovery.pdf): steal and force, shadow paging, write-ahead logging, checkpoints and ARIES, as taught in class.
2. [Atomic Commit In SQLite](https://www.sqlite.org/atomiccommit.html) (SQLite docs): how a rollback journal survives a crash at any moment, step by step.
3. [LevelDB log format](https://github.com/google/leveldb/blob/main/doc/log_format.md) (LevelDB docs): records split across 32 KiB blocks, each fragment with a checksum and a type.
4. [Ensuring data reaches disk](https://lwn.net/Articles/457667/) (Jeff Moyer, LWN, 2011): the layers between `write` and the disk, and the calls that make a file durable.

## Course notes

[CMU 15-445](cmu), lecture notes and videos indexed by topic. Several of the sections above lean on these.
