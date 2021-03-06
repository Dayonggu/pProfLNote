# Things about distributed system

## Techniques

### Streaming
* "Db is cache of log" -- Pat Helland
* *Streaming joins*
  * `stream-stream` join : each stream processor need keep *state* of both stream, only emit result after receive both result from both streams
  * `stream-table` join : *enriching activity events* from records in database table ( at that moment)
    * one is a stream, another is a db's chanage log, keep local copy of database at the given time
    * so *Change Data Capture* would help, so you know the value in database record at the moment of the streaming event
    * use the old value until time of the next CDC event capture a change
  * `table-table` join:
    * materialized the stream event into VIEWS, and join over the views
    * both stream could be database change log
  * *SCD* (slow changing dimension) : for some data that would not be changed frequently, using the unique version identifier for the version of the data (to be used in the join)
    * basically, a version id of a piece of data, a function to time,  vId = f(time), f is a discrete function, or mapping table, each item is one SCD range
    * since the value is not changed frequently, so keep a cache for values of each given period
#### Fault tolerance
* in stream job, you can not simple rerun a failed task as in the batch processing world
* to be *effectively execute once* , you need some other type of ways:
  * *micro-batch* as in `spark`, basically a windowing solution, cut stream to small *batches* and one-minutes etc
  * *checkpoints* used in  `flink`, write out checkpoints, and if failed, restart from checkpoint  
### Event Sourcing
* similar idea as *CDC* but do at the higher level
* Application directly generate the *event log* in its logic, not reply on parsing DB log
* *event log* here means to be *appending only*
  * while in *CDC*, you still can update the DB as usual, but just parsing the redo-log of things changed.
* However, log-compaction is not simple here as in *CDC*, which the latest version always have ALL information of a record
  * here a new Event, may not override all information of a previous event, you need scan the full history to figure out the snapshot of the final status
    * So, you need make some snapshot, checkpoint, to avoid scan full history every time  
### Change Data Capture
* CDC, used to keep multiple data system in sync
  * you want the same data for a DB, a search engine and your data warehouse, and a distributed cache
  * let them all read data from CDC log and apply the changes (better then directly dual-writes to them explicitly)
* basically an extension of the DB logging
* make one *DB* as the leader, create the CDC event log, and all other systems as *derived data system*, catch up with the CDC log
* a log-based message broker is a good fit for transport the CDC *events*
* usually done by *parse redo log* of a DB, and generate the CDC log
### Log based message queues
* e.g. `Kafka`, `Amazon Kinesis`
* basically, put message a *log* in segement on file
* *consumers* just need to read from log file, can do this repeatly, separately, and not destructive ( can redo, replay)
* until the log segment is collected when there is a disk-shortage
* but you surely can put alert when a consumer lag back too much, and OP folks can handle it

### SORT-MERGE
* A trick in Map-reduce system to so *join*
* Use two *Mappers* to read original data from two tables, but use the same key, eg:
  * use the *userId* as key, one from a *user-url-visit*, one from a *user-info*, and get streams like  
    * from *user-url-visit*: `< uID, url>`
    * from *user-info* : `<uID, DOB>` : (date of born)
  * Based on the key (uID), the records are sent to dedicated reducer node for a given uID
  * before feed to reducer, the system would first make a **SORT**, and then the `DOB` info is always goes first then the `url`, so you get `<uId -> {dob, url1, url2,...}>`
  * and then the *Reducer* can make a easy **Merge** to sequence of {<dob1, url1>, <dob1, url2>, <dob1, url3>,}
  * Then a next round of map-reduce would got the *join* result of *histogram of url viewers by age*

### Map-Side join
* *send all data for a given key to the same node* (for `join`, `groupby` etc) would have *skew* problem if you got some *hotkey*.
* `pig` would first sample a portion of the data, detect the hotkey, and send data of it to a few reducer nodes randomly. Where some other system would require a *hotkey* is explicitly provided.  
* `Hive` use *Map-side join*  (tbc)
* *reducer side join* (as talked above) is a general way, if you cannot make any assumption of the data set, and thus would pay cost of sorting and sending data from *mapper* node to *reducer* node
* so-called *Map side join* is trying to do join locally on *mapper* node, if there are some circumstances stand, such as
* *broadcasting hash join*: simple replicate *smaller* set to every mapper's node,
  * either building a in-memory hashmap for the smaller dataset (which must fit in memory)
  * put smaller set locally to dedicated read-only index, and leverage the OS's page-caching to make sure in most case the smaller set is in memory
* *Partition hash join*, when the both the large and small data set are partitioned with the same sharding key, and thus a given mapper would have all the data it need locally to make the join
* *Map-side marge join*: two data sets are partitioned by the same key, and is already sorted. So it is easily to do in-memory merge join

### Geo-Hash
* cut all area into Cells, each cell has a value
* smaller (finer level) cells have more bits
* **neighbors would share the prefix**. e.g. only one unit of different if A and B share a border
  * unit here could be a few bits

### Replication
* four theoretical ways
  * Statement-based replication
    * redo the sql statement on secondary
    * problems:
      * NOW() is local time
      * auto-increasing column is a problem
      * other side-effect functions
    * **useless**
    * *MySql* used this before 5.1, *VoltDb* still using, but require Tx must be *deterministic*
  * WAL shipping
    * WAL logs are *append-only* sequences
    * physical log file replication (page level)
      * Log contains diff on *page*       
    * *SQL Azure*, *PG*, *ORACLE* are using
  * Logical(row-based) log replication
    * *logical* log: talking about ROW not PAGE differences  
      * for `insert`, the log record contains all *columns* of the row
      * for `delete`, log record contains enough information to unique identify the row; the *primary* key, or if not primary key, it must give the whole content to determine the exact row
      * for `update`, log record contains enough information to unique identify the row, and *new* values  
    * decoupled with storage engine, so the *leader* and *follower* can safely run different version of code
    * and logical log is easy for external app to parse/read
    * *MySql* uses this
  * physical page replication
    * The system does't have WAL, go directly with page  
  * Trigger-based replication
    * You can use *trigger* in DBMS, to register *application code* to be executed at some circumstance
      * like put a subset of data into separate tables
    * a way to use application to control data migration
    * eg. `Bucardo` for PG, and `Databus` for Oracle
      *  **INTERESTING** https://bucardo.org/Bucardo/  an asynchronous PostgreSQL replication system;
      * allowing for both *multi-master and multi-slave* operations
      * at its heart a *Perl daemon* that listens for `NOTIFY` requests and acts on them, by connecting to remote databases and copying data back and forth

#### Conflict resolve
* *CRDT* : conflict-free replicated datatypes
  *  data structure which can be replicated across multiple computers in a network, where the replicas can be updated independently and concurrently without coordination between the replicas, and where it is always mathematically possible to resolve inconsistencies which might result
  * check `Riak`
  * check *Gossip protocol* which uses *CRDT*

### consistent controls
* Linearizability violation
  * Client2 reads later than client1, but got an older data
    * if Client2 reach to a follower node 2 which did not receive the latest data yet (from *leader*)
* *Linearizability system*: if a client ever read the *new* data, it would not read the old data again in the subsequent reads ==> *if read new, read new forever*
  * but it is ok to have  another client2, read at a later time but got older data
  * `Zookeeper`, `etcd` are often used to implement distributed locks. They use *consensus algorithms* to implement *linearizable operation*
    * *consensus* here means, with *Leader* but both "Read" and "WRITE" need get responses from >quorum of nodes (*??*)
  * related tech: *Read repair* (and antientropy)
* *Read Repair*
  * In Cassandra: *Read Repair* means that when a query is made against a given key, we perform a digest query against *all the replicas of the key* and push the most recent version to any out-of-date replicas
  * In Dynamo, a *WRITER* must  read the latest state of a quirum of nodes before sending its *WRITE*
  * Read repair serves as an anti-entropy mechanism during read path.
* *Lamport timestamps*
  * basically, append the nodeID to timestamp
  * if you have two timestamps (from different nodes) with the same *counter value*, the one with the greater node ID is the greater timestamp  

### Two-phase commit
* We need a *coordinator*, or *tx manager* (who generate global unique TxId)
* actually three steps
  0. *coordinator* tells all nodes, we are going to *WRITE*
  1. after receiving response from all nodes, *coordinator* would start the *prepare phase*
    * each node must make sure the tx would definately be committable (like write all data to disk), then vote for *yes*
    * after a node say "yes", it must be guarantee the tx would be commit, even if it crashes, it must be do it after recovery
  2. after receiving responses of *yes* from all nods,  *coordinator* first write its decision of *commit* to its WAL log, and then start the *commit phase*, and wait for all nodes say "committed"
    * if there is some nodes not say *committed*,    *coordinator* has to *wait for them forever* (until recovery)
    * if   *coordinator* crashes after make the "commit" decision, before sending the "commit" request, all nodes have to *wait for it (to recovery) forever*  

### Lamport timestamp
* basically a <`TimeStamp`, `Counter`, `NodeId`> structure
* `Counter` is kind of like sequence number, a counter of total requests ever in the whole multiple-node system.  The value of it would be spread to each node in the system by the requests from client
* so we have *timestamp*, then *counter value* then "nodeId". three layers of data to define a *total  order* of events in the system which is consistent to *casual order*
* spread the *counter value*
  * When a client send a request (the 1st time) <*request*, 0> first to Node1, it would get return as <*response payload*, *counterValue*>:  *counterValue* is the current value Node1 knows, assume it is `100`
  * then next time when it sends request, whether to Node1 or Node2, the request would be <*request*, `100`>
    * if this time the request is sent to Node2, on which the current know sequence is only 88, it would be bumped up to 101 by this request
* **NOTE** A *total order* is not sufficient to solve problems in distributed system, like *unique constrains* across multiple nodes.
  * you need *total order broadcasting*, or *consensus* algorithm
  * `Zookeeper`, `etcd` implements total order broadcasting.
* key different of *Lamport ts* Vs *Total order broadcasting*
  * *Total order broadcasting* would guarantee no gap between sequence number
  * *TimeStamp* based solution would have gap (so you could not guess whether you are missing one message in between two received ones)


### Consensus algorithm
* Where do we need *consensus* ?
  0. Leader election
  0. Atomic commit
    * nodes agree on whose request get passed
#### Paxos
#### Raft
* a consensus algorithm that is designed to be easy to understand. It's equivalent to *Paxos* in fault-tolerance and performance.
* to do *leader election*
  * note: not every tx need a consensus run, just leader election
* algorithm
  * Raft algorithm divides time into small *terms* of arbitrary length, with monotonically increasing number, called term number
  * every *term* starts with a *election* of leader. If a *candidate* receives a majority of agreement, it becomes the new *leader*
  * *Servers* increase their *term number* if their term number is less than the term numbers of other servers in the cluster.
    * this means others are already in a new epoch, this node need to catch up
    * and a *Candidate* or *Leader* demotes to *follower*, if figure out its term number is behind others     
    * if a request is achieved with a *stale term number*, the said request is *rejected*
  * use two RPC call
    * `RequestVotes` : to gather votes during an election
    * `AppendEntries` : by the *Leader* node for replicating the *log entries* and also as a *heartbeat* mechanism
  * **leader election**
    * leader would send *heartbeat* to all followers
    * if any of the followers, *timeout* to receive the *heartbeat*, it would start a new round of *leader election*
      * `time_out` period length is randomly selected (usually between 150ms - 300ms) , to ensure that *split votes* are rare and that they are resolved quickly.
    * usually, it should make itself a *candidate* and send the `RequestVotes`
      * after receiving the majority of votes from the cluster nodes, it becomes the new *leader*, and send *heartbeat* to other nodes to claim that " i am the leader"
        * but if its *heartbeat* message with a lower term number of other nodes ( destination node would notice that), its `AppendEntries` would be rejected, and other nodes may becomes new *candidates*
      * if *NOT* receiving the majority, this *term* would endup no leader. The *candidate* would return to *follower* status  

### Google True time
* *confidence interval* of local time -> [earliest, latest]
* `spanner` implements *snapshot isolation* across data-center by using the *TrueTime API* (which report an interval for a *timeStmap*)
  * If there is no *overlap* between interval A and B, we know A *must happen* before B
  * If there is an *overlap*, we are unsure -> need to have a consensus algorithms to resolve

###  Fencing token
* Client1 grabs a lock from *locking service* (with an expiration time), and then start a long unresponding work, like GC. Its lease may expired during this time, but Client1 would still think he owns the lock after wake up.
* Meanwhile,  Client2 may also grab the lock successfully, since Client1's lease already expired
* Solution:
  * using *Fencing token*
  * client1 grabs the lock and was assigned a token *100*
  * client2 grabs the lock and was assigned a token *101* ( since *locking service* considers *100* already expired)
  * When Client1 try to write data with the old token, the write would fail

## Challenges in cloud
* Unreliable
  * remote node
  * network
  * Clocks
  * response time guarantee
    * pauses: GC, disk flush, paging

## Architectures
### Service mesh
* Data Bus connected by *Proxies*  
* *Ingress*

## Appendix
### Related Papers
* Tango: Distributed Data Structures over a Shared Log http://www.cs.cornell.edu/~taozou/sosp13/tangososp.pdf

### CAP:
* *Consistent* or *Available* when **Partitioned**
* P is not a choice, since network problem can not be avoid anyway
### Networking 101
* print slides: 6, 13
* Levels:
  * L1: Physical/link layer : frames/bits
  * L2: network layer, IP : Packet
    * IpV4: 32-bits source and dest addresses (each)
    * IpV6: 128-bits
  * L3: Transport layer, TCP/UDP : segment
  * L4: application: http, ftp, ... : Data
* private ip ranges:
  * 192.168.0.0 - 192.168.255.255 (64K)
  * 172.16.0.0  - 172.31.255.255  (1M)
  * 10.0.0.0    - 10.255.255.255  (16M)
* *NAT*(network address translation) is used to translate from private network ip to public network ip
* Http2.0 support *streaming*: multiple requests on same TCP connection, all responses come back in an Async-way
* DNS: hostname *-->* IP
  * distributed data system
  * application layer protocol, over UDP  
* *Routing*
  * Autonomous System (`AS`)
    * a group of rounters, under the same administration(e.g. within the same company), run the same routing algorithm
  * *ARP protocol* convert ip to MAC address, through a *ARP table*   
