# Anything about distributed system

## Techniques
### Geo-Hash
* cut all area into Cells, each cell has a value
* smaller (finer level) cells have more bits
* neighbours would share the prefix. e.g. only one unit of different if A and B share a border
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

### Google True time
* *confidence interval* of local time -> [earliest, latest]
* `spanner` implements *snapshot isolation* accross datacenter by using the *TrueTime API* (which report an interval for a *timeStmap*)
  * If there is no *overlap* between interval A and B, we know A *must happen* before B
  * If there is an *overlap*, we are unsure -> need to have a consensus algorithms to resolve

###  Fencing token
* Client1 grabs a lock from *locking service* (with an expiration time), and then start a long unresponsing work, like GC. Its lease may expired during this time, but Client1 would still think he owns the lock after wake up.
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

## Appendix

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
