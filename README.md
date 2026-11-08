# Database papers

My notes from learning how databases work: the papers, specs and design docs I read, and what I learned from each.

Alongside, I'm building [onyxdb](https://github.com/PPRAMANIK62/onyxdb), a SQL database in Rust.

The reading itself is in [database-references](database-references), one folder per topic. [problem-solving-sessions](problem-solving-sessions) is separate: one design problem a day, talked through and written up.

## Milestones

onyxdb is built in milestones, each a version you can run.

| Milestone | What works | Topics |
|---|---|---|
| M0 | pages on disk, cached and checksummed | 1–3 |

## Now reading

4. `wal`: how a database promises that a committed write survives a crash.

## Done

| # | Topic | Sources |
|---|-------|---------|
| 1 | [vfs](database-references/1-vfs): how a database talks to files, and what a crash does to them | SQLite VFS docs, Dan Luu, OSDI '14, ATC '20 |
| 2 | [pagefile](database-references/2-pagefile): how a database stores data in fixed-size pages | CMU 15-445, SQLite file format, PostgreSQL page layout, Ross Williams on CRCs |
| 3 | [bufpool](database-references/3-bufpool): how a database keeps pages in memory, and when it writes them back | CMU 15-445, The Internals of PostgreSQL, the mmap paper (CIDR '22), LRU-K (SIGMOD '93) |
