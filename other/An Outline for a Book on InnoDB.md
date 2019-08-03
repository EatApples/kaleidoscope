@see https://www.xaprb.com/blog/2015/08/08/innodb-book-outline/

Baron Schwartz's Website Archives Talks
An Outline for a Book on InnoDB
Published Aug 8, 2015 in Databases

Years ago I pursued my interest in InnoDB’s architecture and design, and became impressed with its sophistication. Another way to say it is that InnoDB is complicated, as are all MVCC databases. However, InnoDB manages to hide the bulk of its complexity entirely from most users.

I decided to at least outline a book on InnoDB. After researching it for a while, it became clear that it would need to be a series of books in multiple volumes, with somewhere between 1000 and 2000 pages total.

At one time I actually understood a lot of this material, but I have forgotten most of it now.

I did not begin writing. Although it is incomplete, outdated, and in some cases wrong, I share the outline here in case anyone is interested. It might be of particular interest to someone who thinks it’s an easy task to write a new database.

High-Level Outline:

- Introduction
- Intro to Major Features
- The InnoDB Architecture
- Indexes
- Transactions in InnoDB
- Locking in InnoDB
- Deadlocks
- Multi-Version Concurrency Control (MVCC)
- Old Row Versions and the Undo Space
- Data Storage and Layout
- Data Types
- Large Value Storage and Compression
- The Transaction Logs
- Ensuring Data Integrity
- The Insert Buffer (Change Buffer)
- The Adaptive Hash Index
- Buffer Pool Management
- Memory Management
- Checkpoints and Flushing
- Startup, Crash Recovery, and Shutdown
- InnoDB's I/O Behavior and File Management
- Data Manipulation (DML) Operations
- The System Tables
- Data Definition (DDL) Operations
- Foreign Keys
- InnoDB's Interface to MySQL
- Index Implementation
- Data Distribution Statistics
- How MySQL executes queries with InnoDB
- Internal Maintenance Tasks
- Tuning InnoDB
- Mutexes and Latches
- InnoDB Threads
- Internal Structures
- XtraBackup
- InnoDB Recovery Tools
- Inspecting Status

Section-By-Section Detailed Outline:

- Introduction
  - History of InnoDB, what its roots are, context of the integration into
    MySQL and the Oracle purchase, etc
  - Based on Gray & Reuters book
  - high-level organization: USER visible things first, after enough
    high-level overview to understand the big features and moving parts; then
    INTERNALS afterwards.

- Intro to Major Features
  - Transactions
    - ACID
  - MVCC
    - multi-version read consistency
    - row-level locking
    - standard isolation levels
    - automatic deadlock detection
  - Foreign Keys
  - Clustered Indexes
  - Page Compression
  - Crash Recovery
  - Exercises

- The InnoDB Architecture
  - This will be a high-level introduction, just enough to understand the
    following chapters
  - Storage on disk
    - Pages, and page sizes
    - Extents
    - Segments
    - Tablespaces
      - The main tablespace and its major components
        - Data pages
        - data dictionary
        - insert buffer
        - undo log area
        - doublewrite buffer
        - reserved spaces for hardcoded offsets
        - see Vadim's diagram on HPM blog
      - individual tablespaces for file-per-table
    - redo log files
  - Storage in memory
    - the buffer pool and its major components
      - Data pages
    - other memory usage
      - adaptive hash index
  - Major data structures
    - LRU list
    - flush list
    - free list
  - Exercises

- Indexes
  - clustered
    - logically nearby pages can be physically distant, so sequential scan isn't
      guaranteed to be sequential
    - no need to update when rows are moved to different pages
    - changing PK value is expensive
  - insert order
    - random inserts are expensive: splits, fragmentation, bad space utilization/fill factor
    - sequential inserts build least fragmented table
    - consider auto_increment
    - optimize table rebuilds, will compact PK but not secondary
      - they are built by insertion, not by sort, except for plugin fast creation
      - secondaries can be dropped and recreated in fast mode in the plugin to defrag
      - but this won't shrink the tablespace
  - primary vs secondary
    - secondary causes 2 lookups, also might cause lookup to PK to check row
      visibility because secondary has version per-page, not per-row
    - PK ranges are very efficient
    - PK updates are in-place, secondary are delete+insert
  - automatic promotion of unique secondary
  - auto-creation of a primary key
  - primary columns are stored in secondary indexes
    - long primary keys expensive
  - uniqueness; is checking deferred, or immediate?
  - there is no prefix compression as there is in myisam, so indexes can be larger
  - rows contain trxn info, but secondary indexes do also at the page level
    (enables index-covered queries; more on this later)
  - Exercises

- Transactions in InnoDB
  - A transaction is a sequence of actions that starts in one legal state, and
    ends in another legal state (need a good definition)
    - from Heikki's slides: atomic (all committed or rolled back at once),
      consistent (operate on a consistent ivew of the data, and leave the data
      in a consistent state at the end); isolated (don't see effects of other txns
      on the system until after commit); durable (all changes persist, even after
      a failure)
      - consistency means that any data I see is consistent with all other data I see
        at a single point in time (there are exceptions to this)
  - How are they started, and in what conditions?
  - what is the transaction ID and its relationship to the LSN?
  - what is the system version number (LSN)?
  - what is a minitransaction/mtr?
  - when are they committed?
  - how are savepoints implemented?
  - what happens on commit?
    - XA and interaction with the binary logs
    - fsync()s
    - group commit
  - what happens on rollback?
  - they are meant to be short-lived and commit; what if they stay open a long
    time or roll back?

- Locking in InnoDB
  - locking is needed to ensure consistency, and so that transactions can
    operate in isolation from each other
  - row-level
  - non-locking consistent reads by default
  - pessimistic locking
  - locks are stored in a per-page bitmap
    - compact: 3-8 bits per lock
    - sparse pages have more memory and CPU overhead per locked row
  - no lock escalation to page-level or table-level
  - row locking
    - S locks
    - X locks
    - there is row-level locking on indexes, (but it may require access to PK
      to see if there is a current version of the row to lock, right?)
    - Heikki says a delete-marked index record can carry a lock.  What are the
      cases where this would happen?  Why does it matter?  I imagine that a
      DELETE statement locks the row, deletes it, and leaves it there locked.
    - supremum row can carry a lock, but infimum cannot (maybe we should discuss
      later in another chapter)
  - table locking - IS and IX locks
    - InnoDB uses multiple-granularity locking, see http://en.wikipedia.org/wiki/Multiple_granularity_locking
    - also called two-phase locking, perhaps?  Baron's a bit confused
    - before setting a S row lock, it sets an intention lock IS on the table
    - before setting a X row lock, it sets an intention lock IX on the table
  - auto-increment locks
    - needed for stmt replication
    - before 5.0: table-level for the whole statement, even if the insert provided the value
    - released at statement end, not txn end
    - this is a serious bottleneck
    - 5.1 and later: two more types of behavior (Look up _____'s issues, I think they
      had some problem with it; also Mark Callaghan talked about a catch-22 with it)
    - row replication lets lock be released faster, or avoid it completely
      - complex behavior: interleaved numbers, gaps in sequences
  - locks set by types of statements
    - select doesn't set locks unless it's serializable
    - select lock in share mode
      - sets shared next-key locks on all index records it sees
    - select for update
      - sets exclusive next-key locks on all index records it sees, as well as locking the PK
    - insert into tbl1 select from tbl2
      - sets share locks on all rows it scans in tbl2 to prevent phantoms
      - can be reduced in 5.1 with read-committed; does a consistent non-locking read,
        - but requires rbr or it will be unsafe for replication
    - update and delete set exclusive next-key locks on all records they find (in pk/2nd?)
    - insert sets an x-lock on the inserted record (not next-key, so it doesn't prevent others)
      - if there is a duplicate key error, sets s-lock on the index record (why?)
    - replace is like an insert
      - if there is a key collision, places x-next-key-lock on the updated row
    - insert...select...where sets x-record-lock on each destination row, and S-next-key-locks
      on source rows, unless innodb_locks_unsafe_for_binlog is set
    - create...select is an insert..select in disguise
    - if there is a FK, anything that checks the constraint sets S-record-locks on the records
      it looks at.  Also sets locks if the constraint fails (why?)
  - lock types and compatibility matrix
  - how is lock wait timeout implemented, and what happens to trx that times out?
  - infimum/supremum "records" can be locked but don't correspond to real rows
    (should we defer this till later?)
  - gap locking, etc are discussed later

- Deadlocks
  - how are deadlocks detected?
    - cycle in the waits-for graph
    - or, too many recursions (200)
    - or, too many locks checked (1 million, I think?)
    - performance impact, disabling
  - do txn check when the set a lock (I think so) or is there a deadlock thread as Peter's slides say?
  - deadlock behavior
    - how is a victim chosen? which one is rolled back?
      - txn that modified the fewest rows
    - how about deadlocks with > 2 transactions involved and SHOW INNODB STATUS
  - causes of deadlocks
    - foreign keys
    - gap locking
    - transactions accessing table through different indexes
  - how to reduce deadlocks
    - make txns use same index in same order
    - short txns, fewer modifications, commit often
    - use rbr and read_committed in 5.1, reduces gap locking
    - in 5.0, use innodb_locks_unsafe_for_binlog to remove gap locking

- Multi-Version Concurrency Control (MVCC)
  - ensures isolation with minimal locking
    - isolation means that while transactions are changing data, other transactions
      see only a legal state of the database---either as of the start of the txn
      that is changing stuff, or at the end after it commits, but not midway
  - readers can read without being blocked by writers, and vice versa
    - writers block each other
  - TODO: some of the details here need to be moved to the next chapter, or they need to be merged
  - old row versions are kept until no longer visible, then deleted in background (purge)
  - Read views
    - how LSN is used for mvcc, "txn sees between X and Y": DB_TRX_ID (6 bytes?) column
    - updates write a new version, move the old one to the undo space, even before commit
      - short rows are faster to update
      - whole rows are versioned, except for BLOBs
    - deletes update the txn ids, leave it in place---special-case of update
    - inserts have a txn id in the future---but they still write to undo space and it is
      discarded after commit
    - mvcc causes index bloat for deletions, but not for updates; updates cause undo space bloat
    - what the oldest view is used for, in srv0srv.c
  - Transaction isolation levels
    - SERIALIZABLE
      - Locking reads as if LOCK IN SHARE MODE.
      - Bypass multi versioning
      - No consistent reads; everyone sees latest state of DB.
    - REPEATABLE-READ (default)
      - Read commited data at it was on start of transaction
      - the snapshot begins with the first consistent read in the txn
      - begin transaction with consistent snapshot starts it "now"
      - update/delete use next-key locking
      - Is only for reads; multiversioning is for reads, not for writes.
        For example, you can delete a row and commit from one session.  Another session can
        still see the row, but if it tries to update it, will affect 0 rows.
      - FOR UPDATE and LOCK IN SHARE MODE and INSERT..SELECT will drop out of
        this isolation level, because you can only lock or intend to update the
        most recent version of the row, not an old version.
        http://bugs.mysql.com/bug.php?id=17228
        "transaction isolation level question from a student in class" thread Mar 2011
        "Re: REPEATABLE READ doesn't work correctly in InnoDB tables?" ditto
    - READ-COMMITED
      - Read commited data as it was at start of statement---"up to date"
      - each select uses its own snapshot; internally consistent, but not over whole txn
      - in 5.1, most gap-locking removed; requires row-based logging.
      - unique key checks in 2nd indexes, and some FK checks, still need to set gap locks
        - prevents inserting child row after parent is deleted
        - which FK checks?
    - READ-UNCOMMITED
      - Read non committed data as it is changing live
      - No consistency, even within a single statement
    - which is best to use? is read-committed really better, as a lot of people believe?
    - http://www.mysqlperformanceblog.com/2011/01/12/innodb-undo-segment-siz-and-transaction-isolation/
    - phantom rows: you don't see it in the first query, you see it when you query again;
      stmt replication can't tolerate them (not a problem in row-based); avoided by gap locking
  - Lock types for MVCC:
    - Next-key locks
    - gap locks
      - gap locks are simply a prohibition from inserting into the gap
        - they don't give permission to do anything, they just block others
        - example: if I hold an exclusive next-key lock on a record, I can't always insert
          into the gap before it; someone might have a gap lock or a waiting next-key lock
          request.
      - different modes of locks are allowed to coexist, because of gap merging via purge
      - holding a gap lock prevents others from inserting, but doesn't permit me to insert;
        I must wait for conflicting locks to be released (many txns can hold the same gap lock)
      - supremum can be gap-locked, infimum can't---why not?
      - which isolation levels use them?
      - types: next-key (locks key and gap before it), gap lock (just the gap before the key),
        record-only (just the key), insert-intention gap lock (held while waiting to insert
        into a gap).
      - searches for a unique key use record-only locks to minimize gap locking (pk updates,
        for example)
    - insert-intention locking
    - index locking (http://dom.as/2011/07/03/innodb-index-lock/)
  - when MVCC is bypassed:
    - updates: can only update a row that exists currently (what about delete?)
    - select lock in share mode
      - sets shared next-key locks on all index records it sees
    - select for update
      - sets exclusive next-key locks on all index records it sees, as well as locking the PK
    - insert into tbl1 select from tbl2
      - sets share locks on all rows it scans in tbl2 to prevent phantoms
      - can be reduced in 5.1 with read-committed; does a consistent non-locking read,
        - but requires rbr or it will be unsafe for replication
    - locking reads are slower, because they have to set locks (check for deadlocks)
  - How locks on a secondary index must lock the primary key, the impact of
    this on the txn isolation level
  - indexes contain pointers to all versions
    - Index key 5 will point to all rows which were 5 in the past
    - TODO: clarify this with Peter
  - Exercises
    - is there really such a thing as a unique index in InnoDB?

- Old Row Versions and the Undo Space
  - old row versions are used for MVCC and rollback of uncommitted txns that fail
    - they provide Consistency: each txn sees data at a consistent point in time
      so old row versions are needed
      - cannot be updated: history cannot change; thus old row versions aren't locked
  - each row in index contains DB_ROLL_PTR column, 7 bytes, points to older version
  - They are stored in a linked list, so a txn that reads old rows is slow,
    and rows that are updated many times are very slow, and long running txns
    that update a lot of rows can impact other txns
    - "rows read" is logical at the mysql level, but at the innodb level, many rows could be read
  - there is no limit on number of old versions to keep
  - history list
  - rollback segment / rseg
    - what is the difference between this and an undo segment / undo tablespace?
  - purge of old row versions
    - how it is done in the main loop
    - how it is done in a separate thread
    - interaction with long-running transactions, when a row version can be
      purged, txn isolation level of long-running transactions
    - it leaves a hole / fragmentation (see __________?id=17673)
  - purging can change gaps, which are gap-locked; what happens to them?
    - a deleted row is removed from an index; two gaps merge.  The new bigger gap
      inherits both the locks from the gaps, and the lock that was on the deleted row
  - innodb_max_purge_lag slows down updates when purge falls behind

- Data Storage and Layout
  - Tablespaces
    - what is in the global tablespace; why it grows; why it can't be shrunk
    - main tablespace can be multiple files concatenated
    - legacy: raw device
    - tablespace header => id, size
  - segments, how they are used
    - leaf and non-leaf node segments for each index (makes scans more sequential IO)
      thus each index has two segments for these types of pages
    - rollback segment
    - insert buffer segment
    - segment allocation: small vs large, page-at-time vs extent-at-a-time
    - how free pages are recycled within the same segment
    - when a segment can be reused
      All pages in extent must be free before it is used in
      different segment of same tablespace
  - file-per-table
    - advantages: reclaim space, store data on different drives (symlinking and its pitfalls),
      backup/restore single tables, supports compression
    - disadvantages: filesystem per-inode mutexes, longer recovery, uses more space
    - free space within a segment can be used by same table only
    - how to import and export tables with xtradb, how this is different from
      import and export in standard innodb
  - file growth/extension, and how it is done
    - ibdata1: innodb_autoextend_increment
    - individual table .ibd files don't respect that setting
    - http://bugs.mysql.com/56433 bug about mutex lock during extension, Yasufumi patched
  - file formats (redundant, compact, barracuda, etc)
  - Page format
    - types of pages
  - Row format
    - never fragmented, except blobs are stored in multiple pieces
    - there can be a lot of empty space between rows on the page
    - infimum/supremum records
  - how SHOW TABLE STATUS works: see issue 17673

- Data Types
  - Data types supported, and their storage format
  - Nulls
    - are nulls equal with respect to foreign keys?  what about unique indexes?
  - Exercises

- Large Value Storage and Compression
  - Page compression
    - new in plugin
    - requires file-per-table and Barracuda
    - Pages are kept uncompressed in memory
      - TODO: Peter says both compressed and uncompressed can be kept in memory
    - compression is mostly per-page
    - uses zlib, zlib library version must match exactly on recovery, because inflate/deflate sizes must match exactly, so can't do recovery on different mysql version than the crash was on, ditto for xtrabackup backup/prepare; if libz is not linked statically, this can cause problems (use ldd to see); recovery might be immature for compressed table spaces. http://bugs.mysql.com/bug.php?id=62011
    - TODO: peter says Uses fancy tricks: Per page update log to avoid re-compression
    - not really configurable
    - syntax: ROW_FORMAT=COMPRESSED KEY_BLOCK_SIZE=4; Estimate how well the data will compress
    - problems:
      - fs-level might be more efficient b/c page size is too small for good compression ratio
      - we have to guess/hint how much it can be compressed
      - setting is per-table, not per-index, but indexes vary in suitability
    - TODO: Peter says KEY_BLOCK_SIZE=16; - Only compress externally stored BLOBs - Can reduce size without overhead
  - Blob/large value storage (large varchar/text has same behavior)
    - small blobs are stored in the page whole, if they fit (max row len: ~8000 bytes, ~1/2 page)
      http://www.facebook.com/notes/mysql-at-facebook/how-many-pages-does-innodb-for-tables-with-large-columns/10150481019790933
    - large blobs are stored out-of-page on a dedicated extent, first 768 bytes in-page
      - same allocation rules as for any extent: page by page, then extent at a time
      - this can waste a lot of space; it makes sense to combine blobs if possible
    - Barracuda format lets us store the whole thing out-of-page, without the 768-byte prefix
    - no need to move blobs to their own table---innodb won't read them unless needed
    - but the 768-byte prefix can make rows larger anyway
    - blob I/O is always "pessimistic"
  - how are BLOBs handled with MVCC and old row versions?
    -  externally stored blobs are not updated in-place, a new version is created
    - does a row that contains a blob, which gets updated without touching the blob, create a new
      row that refers to the same copy of the blob as the old row does?
    - how is undo space handed? TODO: in-page/in-extent with the row, or in the undo log area?

- The Transaction Logs
  - circular
  - File format: not page formatted, record formatted
  - 512 byte units (prevents o_direct, causes read-around writes)
    - if logs fit in os buffer, may improve performance; otherwise puts
      pressure on memory, written circularly and never read except for
      read-around writes, so OS caching is useless
    - tunable in XtraDB
  - records are physiological: page # and operation to perform
  - records are idempotent
  - only redo, not undo
  - what LSN is (bytes written to tx log, tx ID, system version number)
    - where it is used (each page is versioned, each row has 2 lsns in the pk)
    - given a LSN, where does it point to in the physical files? (it's modulo, I think)
  - Changing the log file size
  - Headers and magic offsets, e.g. where the last checkpoint is written
  - Never implemented:
    - multiple log groups
    - log archiving (maybe implemented once, then removed?)

- Ensuring Data Integrity
  - Page checksums (old, new, and faster implementations)
    - checked when page is read in
    - updated when it is flushed
    - how much overhead this causes
    - disable-able, not recommended
  - the doublewrite buffer
    - it isn't really a "buffer" like other buffers
    - avoiding torn pages / partial page writes
    - it is a short-term page-level log; pages contain tablespaceid+pageid
    - process: write to buffer; sync; write original location; sync
    - after crash recovery, we check the buffer and the original location, update original if needed
    - unlike postgres, we don't write a full page to log after checkpoint (logs aren't page-oriented)
    - how big it is; configuring it to a different file in xtradb
    - uses sequential IO, so overhead is not 2x
    - higher overhead on SSD, plus more wear
    - safe to disable on ZFS
  - Exercises

- The Insert Buffer (Change Buffer)
  - changes to non-unique secondary index leaf pages that aren't in the buffer pool are saved for later
  - it is transactional, not a volatile cache
  - how much performance improvement it gives: said to be 15x reduction in random IO
  - how it is purged/merged
    - in the background, when there is time.  This is why STOP SLAVE can trigger a huge flood of IO.
      - the rate is controlled by innodb_io_capacity, innodb_ibuf_accel_rate
      - done by main thread (?)
      - might not be fast enough---dedicated thread in xtradb
    - also (transparently) in the foreground, when the page that has un-applied changes is read from disk for some other reason.
    - if a lot of changes need to be merged, it can slow down page reads.
  - it is changed to "change buffer" in recent plugin
  - tunability
    - it can take up to 1/2 of the buffer pool, which isn't tunable in standard innodb
    - xtradb lets you disable it, and set the max size
    - newer innodb plugin changed insert buffer to change buffering, and lets you disable them
    - disabling can be good for SSDs (why?)
  - inspecting status; status info after restart only shows since restart, not over the lifetime of the database
  - it is stored in ibdata1 file.  Pages are treated same as normal buffer pool pages, subject to LRU etc.
  - what happens on shutdown: you can set fast_shutdown off, so a full merge happens (slow)
  - after a restart, it can slow down, because pages aren't in the buffer pool, so random IO is needed to find them and merge changes into them
  - Things that were designed but never implemented
    - multiple insert buffers

- The Adaptive Hash Index
  - think of it as "recently accessed index cache"
  - it's a kind of partial index: build for values that are accessed often
  - fast lookups for records recently accessed, which are in the buffer pool
  - is a btree that works for both pk and secondary indexes
    - can be built for full index entries, and for prefixes of them, depending on how they are looked up
  - not configurable, except you can disable it
  - how much does it help performance?
  - there is only one, it has a single mutex, can slow things a lot
  - xtradb lets you partition it

- Buffer Pool Management
  - dirty pages vs clean pages
  - the insert buffer in memory
    - configuration and status inspection
  - the LRU list: pages in leat recently used order; has midpoint in newer innodb
    - young-old sublists and performance: http://www.mysqlperformanceblog.com/2011/11/13/side-load-may-massively-impact-your-mysql-performance/
  - the flush list: pages in oldest modified order
  - the free list: pages that are not used
  - overhead per buffer pool page makes it consume more memory than expected,
    e.g. see http://www.facebook.com/note.php?note_id=491430345932 and
    http://www.mysqlperformanceblog.com/2010/08/23/innodb-memory-allocation-ulimit-and-opensuse/
  - multiple buffer pools

- Memory Management
  - system versus own malloc
  - the additional memory pool: stores dictionary
  - the adaptive hash index in memory
  - latch/lock storage (is that stored in the buffer pool?)

- Checkpoints and Flushing
  - fuzzy vs sharp checkpoints
  - what happens when innodb has not enough free pages, or no space in the log files?
    - checkpoint spikes/stalls/furious flushing
  - smoothing out checkpoint writes
  - flush algorithms
    - standard in old innodb
    - adaptive checkpointing (xtradb)
    - adaptive flushing (innodb-plugin)
    - neighbor page flushing: next/prev pages (hurts SSD; tunable in xtradb)
  - flushing and page replacement
    - why page replacement? must clean a page before we can replace it with another from disk.
    - server tries to keep some pages clean: 10% in older versions, 25% in newer (innodb_max_dirty_pages_pct)
    - LRU algorithms: old, new
    - two-part lru list to guard against wiping out on scans (midpoint insertion)
    - the lru page replacement algorithm is explained by Inaam:
      http://www.mysqlperformanceblog.com/2011/01/13/different-flavors-of-innodb-flushing/
      1) if a block is available in the free list grab it.
      2) else scan around 10% or the LRU list to find a clean block
      3) if a clean block is found grab it
      4) else trigger LRU flush and increment Innodb_buffer_pool_wait_free
      5) after the LRU flush is finished try again
      6) if able to find a block grab it otherwise repeat the process scanning deeper into the LRU list
        There are some other areas to take care of like having an additional LRU
        for compressed pages with uncompressed frames etc. And
        Innodb_buffer_pool_wait_free is not indicative of total number of LRU
        flushes. It tracks flushes that are triggered above. There are other
        places in the code which will trigger an LRU flush as well.
  - flush list
    - contains a list of pages that are dirty, in LSN order
    - the main thread schedules some flushes to keep clean pages available
    - this is a checkpoint, as well, because it flushes from the end of the flush list
    - innodb_io_capacity is used by innodb here, but not by xtradb
      - assumed to be the disk's writes-per-second capacity
      - Peter writes: Affects number of background flushes and insert buffer
      merges (5% for each).  What does 5% mean?
    - when the server is idle, it'll do more flushing
    - flushing to replace is done in the user thread
  - What happens on shutdown

- Startup, Crash Recovery, and Shutdown
  - What is done to boot the system up at start?
  - Fast vs slow shutdown
    - implications for the insert buffer,
      http://dev.mysql.com/doc/innodb/1.1/en/innodb-downgrading-issues-ibuf.html
  - What is done to prepare for shutdown?
    -  setting innodb_max_dirty_pages_pct to prepare
    - you can't kill the server and it is blocking, so shutdown can take a while otherwise
  - What structures in the server have to be warmed up or cooled off? e.g. LRU
    list, dirty pages...
  - stages of recovery: doublewrite restore, redo, undo
    - redo is synchronous: scan logs, read pages, compare LSNs
      - it happens in batches
    - undo is in the background since 5.0
      - faster with big logs (why?)
      - alter table commits every 10k rows to avoid long undos
      - very large dml is a problem, causes long undo after crash; don't kill long txns lightly
    - how long will recovery take? 5.0 and 5.1 had slow algorithm; fixed in newer releases
      - buffer pool size also matters; in old versions, configure for small size, then restart
      - http://bugs.mysql.com/bug.php?id=29847
    - larger logs = longer recovery, but it also depends on row sizes, database size, workload
    - are there cases when recovery is impossible? during DDL, .FRM file is not atomic
  - how innodb checks and uses the binary log during recovery
  - the recovery threads---transactions are replayed w/o mysql threads, so they
    look different

- InnoDB's I/O Behavior and File Management
  - How files are created, deleted, shrunk, expanded
  - How InnoDB opens data files: o_direct, etc
  - buffered vs direct IO
    - buffered:
      - advantage: faster warmup, faster flushes, reduce inode locking on ext3
      - bad: swap pressure, double buffering, loss of effective memory
    - direct:
  - Optimistic vs pessimistic IO
    http://dom.as/2011/07/03/innodb-index-lock/
  - How InnoDB opens log files
    -  always buffered, except in xtradb
  - How InnoDB writes and flushes data files and log files
    - the log buffer
    - flushing logs to disk; innodb_flush_log_at_trx_commit; what is safe in what conditions
  - I/O threads
    - the dedicated IO threads
    - the main thread does IO in its main loop
    - dedicated threads for purge, insert buffer merge etc
  - read-ahead/prefetches for random and sequential IO; how an extent is determined to
    need prefetching
    - don't count on it much
    - random read-ahead removed in version X, added back; impact of it
  - merging operations together, reordering, neighbor page operations
    http://dom.as/2011/07/03/innodb-index-lock/
  - async io
    - simulated: arrays, slots
    - native on Windows and in Linux in version 1.1
  - which I/O operations can be foregrounded and backgrounded
    - most writes are in the background
    - flushes can be sync if there are no free pages
    - log writes can be sync or async, configurable
    - thresholds: 75% and 85% by default (confirm)
  - what operations block in innodb? background threads sometimes block
    foreground threads; MarkC has written about;
    http://bugs.mysql.com/bug.php?id=55004
  - I/O operations for things like insert buffer merge (causes reads) and old row version purge
  - the purpose of files like "/tmp/ibPR9NL1 (deleted)"

- Data Manipulation (DML) Operations
  - select
  - insert
  - update
  - delete
    - does it compact? ___________?id=14473

- The System Tables
  - sys_tables
  - sys_indexes
  - sys_foreign
  - sys_stats
  - sys_fields

- Data Definition (DDL) Operations
  - How CREATE TABLE works
  - How ALTER TABLE works
    - Doesn't it internally commit every 10k rows?
  - create index
    - fast index creation; sort buffers; sort buffer size
    - Creates multiple transactions: see email Re: ALTER TABLE showing up more than once in 'SHOW ENGINE INNODB STATUS' Transaction list?
  - optimize table
  - analyze table
  - How DROP TABLE works
    - with file-per-table, it is DROP TABLESPACE, which blocks the server; see
      https://bugs.launchpad.net/percona-server/+bug/712591
  - InnoDB's internal stored procedure language

- Foreign Keys
  - implications for locking: causes additional locking, opportunities for deadlocks
  - cascades
  - nulls
  - rows that point to themselves, or rows that have cycles; can they be deleted?
  - is checking immediate, or deferred? it is immediate, not done at commit.
  - names are case sensitive
  - indexes required; change in behavior in 4.1
  - data types must match exactly
  - how they interact with indexes
  - appearance in SHOW INNODB STATUS
  - they use the internal stored procedures
  - InnoDB has to parse the SQL of the CREATE statement

- InnoDB's Interface to MySQL
  - The Handler interface
  - Relationship with .frm file
  - Built-In InnoDB
  - The InnoDB Plugin
  - Converting rows to MySQL's row format
  - what columns are in every table (and can't be used in a real table)
  - Communicating ha::rows_in_range and ha::info and other statistics
  - Communicating index capability bits (enables covering index queries)
  - Interaction with the query cache
  - MySQL thread statuses
    - they appear in INNODB STATUS
    - what statuses can be present while query is inside innodb: "statistics"
      for example
  - Implementation in ha_innodb.cc
  - Hacks: magic CREATE TABLE statements like innodb_table_monitor,
    parsing the SQL for FK definitions
  - how table and row locks are communicated between engine and server
    - innodb_table_locks=1 means that innodb knows about server table locks; what does it do
      with them?
    - the server knows about row locks---and it can tell innodb to release non-matched rows?
  - Exercises

- Index Implementation
  - built on b-trees
  - leaf vs non-leaf nodes, the row format on them
  - secondary indexes
  - data pages store the rows in a heap within the page
  - page fill factor
  - page merges and splits
    - is something special done during deletes?

- Data Distribution Statistics
  - how they are gathered
  - inaccuracy
  - configurability of whether to gather them or not
  - stability/randomness (older InnoDB isn't properly random and is non-uniform)
    - how many samples? config options that affect that
    - ability to stop resampling
    - ability to store persistently with innodb_use_sys_stats_table

- How MySQL executes queries with InnoDB
  - high-level overview of optimization, statistics
  - table locking and lock releasing
    - releasing rows locks for rows eliminated by WHERE clause in 5.1; isolation
  - index-only (covering index) queries
    - how it works
    - when an index query must look up the pk (when a page has a newer lsn
      than the query's lsn; rows in secondary indexes don't have LSNs, only the
      page does)
    - what extra columns are included in the secondary indexes

- Internal Maintenance Tasks
  - Old Row Purge
  - Insert Buffer Merge
  - The statistics collector
    - rules for when stats are recomputed
      - by mysql: at first open, when SHOW TABLE STATUS / INDEX commands are used (configured with innodb_stats_on_metadata) or when ANALYZE TABLE is used)
      - by innodb: after size changes 1/16th or after 2B row insertions
        (disable with innodb_stats_auto_update=false)
    - stats are computed when table is first opened, too
    - bug: stats not valid for an index after fast-create (http://bugs.mysql.com/bug.php?id=62516)
    - Jervin's blog: http://www.mysqlperformanceblog.com/?p=7516&preview=true

- Tuning InnoDB
  - buffer pool size
  - using multiple buffer pools
  - log file size (1h worth of log writes)
  - log buffer size (10 sec worth of log writes; bigger for blobs; error if too small)
  - checkpoint behavior
  - flush_logs_at_trx_commit
  - dirty page pct in buffer pool
    - setting it lower doesn't smooth IO by causing constant writing---it causes much more
      IO and doesn't give a buffer to absorb spikes.
  - o_direct
  - all configuration variables

- Mutexes and Latches
  - how innodb implements rw-locks and mutexes
  - list of the major ones, and what they are for
  - the order they are locked in
  - log buffer
  - buffer pool
  - adaptive hash index
  - new_index->mutex
  - sync array / sync_array
    - ut_delay
  - kernel->mutex
    http://www.mysqlperformanceblog.com/2011/12/02/kernel_mutex-problem-cont-or-triple-your-throughput/comment-page-1/#comment-850337
  - approaches to scalability: XtraDB (split into many), InnoDB 1.1 (multiple buffer pools)
  - what are spins, spin rounds, OS waits, how are they configurable
  - what is the OS wait array
  - what are condition variables, and what are they for
    - broadcasts are expensive as the txn list grows, according to mark callaghan?
  - what are innodb events?  See http://mysqlha.blogspot.com/2011/02/this-happens-when-you-dont-have.html

- InnoDB Threads
  - thread per connection
  - thread statuses (innodb internal ones, not mysql ones)
  - user threads
  - recovery threads
  - IO threads
  - main thread
    - schedules other things
    - flush
    - purge
    - checkpoint
    - insert buffer merge
  - deadlock detector
  - monitoring thread
  - error monitor thread; see sync/sync0arr.c
  - statistics thread (??)
  - log thread
  - purge thread
  - innodb_thread_concurrency and the queue, concurrency tickets
    - limit includes threads doing disk io or storing data in tmp table

- Internal Structures
  - Data dictionary
    - auto-increment values; populated at startup
    - statistics
    - system info
    - the size overhead per tabel can be 4-10k; this is version dependent
    - xtradb lets you limit this
  - Arrays
    - os_aio_read_array, os_aio_write_array, os_aio_ibuf_array
    - mutex/latch/semaphore/whatever arrays
  - data structures and what they're used for
    - heaps (rows in the page, for example)
    - b-trees
    - linked lists (?)
    - arrays

- XtraBackup

- InnoDB Recovery Tools

- Inspecting Status
  - SHOW STATUS counters
    - Innodb_buffer_pool_wait_free, per Inaam, is not indicative of total number of
      LRU flushes. It tracks flushes that are triggered from an LRU flush; there are more
      places in the code which will trigger an LRU flush as well.
  - show innodb status
    - innodb status monitor
    - lock monitor
    - tablespace monitor
    - writing to a file
    - truncation and the in-memory copy and its filehandle
  - information_schema tables in the plugin (esp. locks and trx)
    - how it attempts to provide a consistent view of the tables; consult
      https://bugs.launchpad.net/bugs/677407
  - show mutex status

Further reading:

  * http://blogs.innodb.com/wp/2011/04/mysql-5-6-multi-threaded-purge/
  * XDES
  * The InnoDB core sub-systems are:
     1. The Locking sub-system
     2. The Transaction sub-system
     3. MVCC  views
     (http://blogs.innodb.com/wp/2011/04/mysql-5-6-innodb-scalability-fix-kernel-mutex-removed/)
  * The Wikipedia article on InnoDB?
  * InnoDB does bulk commits for things like ALTER, every 10k rows, to avoid problems internally.
Baron Schwartz

I’m the founder and CTO of VividCortex, author of several books, and creator of various open-source software. I write about topics such as technology, entrepreneurship, and fitness, and I tweet at @xaprb. More about me.

Subscribe for email updates!

Story logo
© 2019 Baron Schwartz
