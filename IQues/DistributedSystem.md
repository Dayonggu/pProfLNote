# Anything about distributed system

## Techniques
### Replication logs
* four theoretical ways
  * Statement-based replication
    * redo the sql statement on secondary
    * problems:
      * NOW() is local time
      * auto-increasing column is a problem
      * other side-effect functions
    * **useless**
  * WAL shipping
    * physical log file replication (page level)
    * **SQL Azure**
  * Logical(row-based) log replication
    * decoupled with storage engine
  * physical page replication
    * The system does't have WAL, go directly with page  

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
