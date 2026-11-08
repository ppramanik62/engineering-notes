# Database papers

My notes from learning how databases work: the papers, specs and design docs I read, and what I learned from each.

Alongside, I'm building [onyxdb](https://github.com/PPRAMANIK62/onyxdb), a SQL database in Rust.

The sources are in [database-references](database-references), one section per topic. [problem-solving-sessions](problem-solving-sessions) is separate: one design problem a day, talked through and written up.

## Milestones

onyxdb is built in milestones, each a version you can run.

| Milestone | What works | Topics |
|---|---|---|
| M0 | pages on disk, cached and checksummed | 1–3 |

## Now reading

4. Write-ahead log: how a database promises that a committed write survives a crash.

## Done

| # | Topic | Sources |
|---|-------|---------|
| 1 | [Virtual file system](database-references/README.md#virtual-file-system): how a database talks to files, and what a crash does to them | SQLite VFS docs, Dan Luu, OSDI '14, ATC '20 |
| 2 | [Page structure](database-references/README.md#page-structure): how a database stores data in fixed-size pages | CMU 15-445, SQLite file format, PostgreSQL page layout, Ross Williams on CRCs |
| 3 | [Buffer manager](database-references/README.md#buffer-manager): how a database keeps pages in memory, and when it writes them back | CMU 15-445, The Internals of PostgreSQL, the mmap paper (CIDR '22), LRU-K (SIGMOD '93) |
