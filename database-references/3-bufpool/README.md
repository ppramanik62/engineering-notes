# 3. bufpool

How a database keeps pages in memory: frames, pins, the write-ahead rule, eviction policies and latches.

My implementation: [`bufpool` in onyxdb](https://github.com/PPRAMANIK62/onyxdb/tree/main/crates/bufpool).

## References

1. [CMU 15-445 notes: Memory Management](https://15445.courses.cs.cmu.edu/spring2026/notes/04-bufferpool.pdf) and [Hash Tables](https://15445.courses.cs.cmu.edu/spring2026/notes/07-hashtables.pdf): frames, the page table, pinning, replacement policies, locks against latches, and the hash table behind the page table, as taught in class.
2. [The Internals of PostgreSQL, ch. 8: Buffer Manager](https://www.interdb.jp/pg/pgsql08.html) (Hironobu Suzuki): a real buffer pool, with buffer descriptors, a partitioned buffer table, pinning, clock-sweep eviction and ring buffers.
3. [Are You Sure You Want to Use MMAP in Your DBMS?](https://db.cs.cmu.edu/papers/2022/cidr2022-p13-crotty.pdf) (Crotty, Leis and Pavlo, CIDR '22): why databases don't let the operating system manage their pages.
4. [The LRU-K Page Replacement Algorithm for Database Disk Buffering](https://www.cs.cmu.edu/~christos/courses/721-resources/p297-o_neil.pdf) (O'Neil, O'Neil and Weikum, SIGMOD '93): evict by the time of the K-th most recent reference, so one scan can't flush the pool.

## What I learned

- A database keeps its own cache of pages instead of trusting the OS with `mmap`. The OS can write a changed page to disk at any time, even before the log, and a failed read shows up as a `SIGBUS` wherever the page was touched.
- In the mmap paper's tests, `mmap` was 2 to 20 times slower than reading the file directly once memory filled up. Eviction ran on one kernel thread, and every eviction interrupted the other cores.
- A pin and a latch do different jobs. A pin keeps a page in its frame. A latch keeps its bytes from changing while someone reads them.
- Latches aren't locks. Latches protect memory for a short time and avoid deadlocks by taking them in a fixed order. Locks protect a transaction's data, and a deadlock between them is detected and one transaction is aborted.
- The write-ahead rule: a changed page can reach the file only after the log is on disk up to that page's LSN.
- PostgreSQL's buffer pool has a pin count and a latch per buffer, and a hash table split into 128 partitions, each with its own lock.
- PostgreSQL's clock sweep caps the usage count at 5 (`BM_MAX_USAGE_COUNT`). Suzuki's chapter says 15, so it's worth checking claims like that against the source.
- PostgreSQL went from LRU to ARC to 2Q to the clock sweep, all in 2005.
- Plain LRU looks only at a page's last use, so one big scan can push out every popular page. LRU-K looks at the K-th most recent use instead, so a page used once is evicted before a page used twice.
- LRU-K can be fooled by a page used twice in a row. The paper treats uses close together as one, its correlated reference period.
