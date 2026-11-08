# Database references

The reading behind each part of a database, one folder per topic, numbered in the order I read them.

## Virtual file system

[1-vfs](1-vfs). Everything a database stores ends up in a file, and the calls that get it there are weaker than they look. What `write` and `fsync` really promise, what a crash leaves behind, and why a database puts one thin layer over the file system instead of calling it directly.

## Page structure

[2-pagefile](2-pagefile). A file is a flat run of bytes, so a database cuts it into fixed-size pages and gives each one a header. How a page is laid out, how rows are packed into it, and how a checksum catches a page that came back wrong.

## Buffer manager

[3-bufpool](3-bufpool). Disk is slow, so pages are cached in memory and written back later. Which page to throw out when memory is full, how to stop a page being evicted while it's in use, and why the write-back order matters more than the speed.

## Write-ahead log

[4-wal](4-wal). Once a write is committed it has to still be there after the power goes out. An append-only log of checksummed records, what `fsync` buys you, how replay handles a half-written tail, and what happens when `fsync` itself fails.

## Course notes

[../cmu](../cmu). CMU 15-445 lecture notes and videos, indexed by topic. Several of the folders above lean on these.
