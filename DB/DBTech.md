# Database Techniques

## Page, buffer pool
* sometimes LRU is the worst replacement policy
  * consider a *sequential disk scan*, this would continuous bring new page in memory, and would not reuse them again
  * if use *LRU* the scan would be flood, evict all previous existing pages in bufferpool by the page from the scan ( and thus, the other previous page lose the possiblity of reuse)
  * in this case, we should use *MRU*, the scan keeps on using the same page in bufferpool

## Eventually consistence
* *Dynamo style* DBs :  `Dynamo`, `Cassandra`, `Riak`, etc
  * Leader less, grossipy
  * *Quorum based*, configurable for `Write` set and `Read` set
    * but usually `w+r > n`
* *sloppy Quorum*
  * Traditional/classic quorum based system says:
    * For a given data, there is a *home* set of node for them, the number is *n*
    * *writes and reads* of that piece data must go to a node in *home* set
    * however, how about one or more nodes in the *home* set were (temporarily) unreachable? This could lead to a write or read never reach a *quorum*
  * *sloppy Quorum*  is just to temporarily *borrow* another node in the cluster, but not in the *home* set, to accept the writes, while the nodes in *home* set comes back, there would be a *hinted handoff*, to return the data (of the new version) back to the node in *home* set
    * technically, this is a just emergent handling process
    * it just said *w* copies of the data is stored in the cluster
    * it cannot guarantee no *stale* data would be read, even if client reads from *r* different nodes (meet the read quorum), since some of the new version copies are on different nodes out of the *home* set ( until *handoff* is done).    

## Concurrency
### Different types of *reads* and *isolation*
* *Dirty read* : Thread 2 reads *uncommitted* write from thread 1
  * Resolved by Isolation level `READ_COMMITED`
  * DB would remember the "old value" of a record as it is when a TX with *wite-lock*
  * other thread wants read it, the "old value" is provided (snapshot value?)
* *Dirty Write*: One thread write to record but not committed, then the other thread should not *Override* this uncommitted *dirty write*. DB makes sure this usually by blocking the second write to same record, until the first is done(committed or aborted).   
  * using *row-level-lock*
* *NON REPEATABLE READS* :
  * Thread 2 in one tx read the same rows twice, got different value
  * since Thread 1 *committed* a new value to the row
  * you need isolation level `REPEATABLE_READ` to avoid this
  * you need *snapshot isolation* level to solve this
    * Each transaction only *read* from a frozen snapshot of data at the beginning of the transaction
      * At beginning of each Tx, DB system makes a list of *on-going* transactions
      * all *writes* from Tx in this list, and late Tx (with a later TxID than current TX) are ignored to the current Tx
    * use *write* lock, so *reader would not block writer, and writer would not block reader* (but *block other writers*)
    * System need to maintain multiple versions of uncommitted data **MVCC** (see below)
* *PHANTOM READS*
  * Thread 1 insert/delete a row in a range, which thread 2 is working on
  * so Thread 2 may read data from a mysterious new or already deleted rows
  * You need `SERIALIZABLE` level

### write skew
* Two threads are doing their own *writes* to different record
* however, the logic of how to write, which to write or what to write are based on some *read* on a record, that the two threads may get skewed value
* e.g.: A and B would to write their own record as "Out-Of-Office", however the company says we must have at least one guy on duty.
  * where on read, A and B both count() for all records shown on duty and get *2* both
  * THen both A and B would decide to write its own record to  "Out-Of-Office", since they believe there is another guy is on duty

### MVCC
* Multiversion concurrency control
* When an MVCC database needs to update a piece of data, it will not overwrite the original data item with new data, but instead *creates a newer version* of the data item
* The most common *isolation level* implemented with MVCC is *snapshot* isolation.
  * a transaction observes a state of the data as when the transaction started
* MVCC uses timestamps (TS), and incrementing transaction IDs, to achieve transactional consistency
* When Tx `T1` want to write an object, and *Read Time Stamp(RTS)* of `T1`, must be earlier than the *RTS*'s all the other Transactions, such as `T2`, `T3`,
  * In other word, you read Object at t1, but others read the object at t0 < t1, you cannot write/update the object.
* Pros:
  * READ is not blocked ever
* Cons:
  * need to store multiple versions, and implement a VACUUM (to cleanup versions that would not be used )
  * if there are higher frequent concurrent WRITES, lots of transaction would fail and retry   

### Two-phase locking (2PL)
* a *pessimistic* concurrency control
* This is *NOTHING RELATED* to **2PC(2-phase-commit)**
* first grabs *shared-lock*
* if you want to write, upgrade to *exclusive-lock*
* *predicate lock* is object-level 2PL, it would protect object not even exist (so prevent the PHANTOM )
* *Index range lock* is a simplified approximation to predicate lock
  * Lock a value range on an index (so it would lock a particular object, to update which would lead to updating the index)

### serializable snapshot isolation (SSI)
* an *optimistic* concurrency control
  * *optimistic* means all Transactions just go, detect bad thing at commit time (and abort the transaction)
  * perform well is *contention* is not very high
  * in SSI, *read-only* transaction would not grab any lock, just go
* SSI based on *snapshot isolation*, apply an algorithm to detect *serialization
 conflicts*  
  * Detecting *read* stale *MVCC* object version
    * in snapshot isolation, we don't care (ignored) uncommitted *WRITES*
    * so at commit time of this Tx (eventually would make a *write* based on *reads*), we need check all these *ignored writes* are still not committed, if any of them committed. this Tx aborted.
  * Detection *write* that affect prior *reads*   
    * *comment:* Kind of like the same thing as above but from a different direction
    * When a Tx writes to DB, it must check the indexes for
      * is there any other transaction *read* the affected data?
    * This is like trying to grab a *write* lock, but
      * instead of being blocked until the *Readers* done
      * This Tx continues, just notify those transactions the data they read may no longer be up to date.

## Data Warehouse
### Data cube in DW
* Pre-aggregate cells
  * Add some pre-aggregate (of cells in the same rows or columns, or a cube/block ) extra cells,
  * good for windowing queries

### Storage structures
### R-Tree
* balanced search tree for *spatial access methods*
* group nearby objects, and represent them with minimum bounding Rectangle
* R1 can contains smaller rectangle, like R11, R12, etc, while R2 can contain R21, R22, ...
* check *GeoHash*  

### LSM
* *Log Structure Merge* Tree
* In memory *SSTable*, still *sorted*, in *BTree*
  * *SSTable* : Sorted Strings Table, contains a sequence of blocks (typically each block is 64KB in size, but this is configurable).
* Each node can hold its own SSTable
  * assume there is a universal clock, so each record updates would get a unique timestamp or sequence number, or whatever you naming it
* When need to write out to disk (as small indexed/sorted files), it would write to files with *Log Structure*
  * Data stream of K-V pair, sorted by key
* Then there is a different *process* to do the *merge*
  * merge small files, remove unnecessary versions (those with *TombStone*)
* Elements of a record could be in any level (if not in in-memory SSTable, then could be in first level files, second level, etc), so each *possible* levels must be consulted.
  * `Bloom Filter` is used here to decide the *possible* levels
  * Every level of indexed files would have a *key range*
  * If A level say *Yes* to bloom filter, there is a possibility, your data is on this level
  * But say *No*, means definitely *No*
* Vs BTree
  * LSM:
    * pros:
      * Less write needed, sequential write, high throughput
      * Not page oriented
      * Compress better, smaller disk files, no fragmentation
    * cons:
      * Compaction process impact read performance
      * multiple instance of key, not good for transaction isolation
        * you need to find all, and check by timestamp(sequence) to choose
  * B-Tree
    * pros:
      * Direct read
      * single instance of key, good for transaction isolation
    * cons:
      * more writes (whole page), random writes, page oriented
      * Leave disk fragmentation

## Paper selections
* **Tapir** : consistent tx with inconsistent replication
  * https://syslab.cs.washington.edu/papers/tapir-tr14.pdf
  * *key idea*
    * claim using both "distributed tx protocol" and "replciation protocol" to maintain consistency is *redundant*
    * invent a *IR* : inconsistent replication, operations through IR in two modes    
      * *inconsistent* mode: operations can run in any order
      * *consensus* mode: operations run in any order, but return a *single consensus* result
        * allow the application (using distributed tx protocol) to decide outcome (resolve) of conflicts ( sounds like B-C idea?)
      * Client side: invokeInconsist(op);  invokeConsensus(op, decide(results))->result : `decide()` to resolve conflict (app-side)
      * replica side: execInconsist(op);  execConsensus(op) -> result (of this replica)  ...  Sync(R) Merge(d,u) -> record
  * full path sounds like:
    * Client.invokeInconsist(op)
      * ==> Rep1.invokeInconsist(op) || Rep2.invokeInconsist(op) || ...
    * client side get a list of results,  invokeConsensus(op, decide(results))->result. make a decision on the result
      * ==> Rep1.execConsensus(op) ->result || Rep2.execConsensus(op) ->result || ...
    *  When need recovery : Rep1.Sync(R) Merge(d,u) -> record,  Rep2.Sync(R) Merge(d,u) -> record, ...
      * keep Replicas eventually converge  
  * **IR** protocol, use 4 sub-protocol
    * Operation Processing


##Appendix

#### term

* {\*}L:
  * DDL – Data Definition Language: CREATE, DROP, ALTER, TRUNCATE, COMMENT, RENAME
  * DQl – Data Query Language : SELECT
  * DML – Data Manipulation Language : INSERT, DELETE, UPDATE
  * DCL – Data Control Language : GRANT, REVOKE     
  * TCL-  Transaction Control Language : COMMIT, ROLLBACK, SAVEPOINT, SET TRANSACTION
