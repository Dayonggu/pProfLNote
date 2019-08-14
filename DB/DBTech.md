# Database Techniques
## R-Tree
* balanced search tree for *spatial access methods*
* group nearby objects, and represent them with minimum bounding Rectangle
* R1 can contains smaller rectangle, like R11, R12, etc, while R2 can contain R21, R22, ...
* check *GeoHash*  

## LSM
* *Log Structure Merge* Tree
* In memory SSTable, still *sorted*, in *BTree*
* Each node can hold its own SSTable
  * assume there is a universal clock, so each record updates would get a unique timestamp or sequence number, or whatever you naming it
* When need to write out to disk (as small indexed/sorted files), it would write to files with *Log Structure*
  * Data stream of K-V pair, sorted by key
* Then there is a different *process* to do the *merge*
  * merge small files, remove unnecessary versions (those with TombStone)
* Elements of a record could be in any level (if not in in-memory SSTable, then could be in first level files, second level, etc), so each *possible* levels must be consulted.
  * `Bloom Filter` is used here to decide the *possible* levels
  * Every level of indexed files would have a *key range*
  * If A level say *Yes* to bloom filter, there is a possiblity, your data is on this level
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
