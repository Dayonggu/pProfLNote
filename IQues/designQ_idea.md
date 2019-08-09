# Design system

## Specific functional model

* Request Rate limit:
  * *Q:* each client id can at most have x request per second
  * keep a request history with each clientId, with time_stamp
    * clientId, time_stamp, request
  * consider as an array `A`, size = *R*, which is the max_rate
  * initialized as LONG_TIME_AGO
  * check the `now() - A[(cur+1)%R] > R` or not
  * size is R, so cur+1 is the R-th previous request from now
* TinyURL
  * single machine: basically a base-conversion problem
    * you have a DB with three col: ID, fullURL, tinyURL; ID is auto-inc
    * you have a map of chars can be used in the url ,with size Z , usually Z = 62  
    * for each full URL, get its tinyURL by convert ID as base Z ( ID%Z till to 0)
  * multiple machines:
    * using distributed KV store, consistent hashing request to different server
    * Insert
        * Hash an input long url into a single integer;
        * Locate a server on the ring and store the key--longUrl on the server;
        * Compute the shorten url using base conversion (from 10-base to 62-base) and return it to the user.
     * Retrieve
       * Convert the shorten url back to the key using base conversion (from 62-base to 10-base);
       * Locate the server containing that key and return the longUrl.
  * Top N error code in last 5 min in log
    * *Circle buffer*: Vector<Map<ErrorCode, Cnt>>
    * if log is not in time order
      * use *TreeMap* , B+ Tree, ordered by time, with errorcode cnt
    * for *massive* log:
      * page by fixed time range, within range, using B+Tree, ordered by timestamp  


## Distributed system in general
### Backend system design problems
* way to scaling:
  * cloning, just add new instances
  * splitting services to microservices
    * split by functionality
  * Data/request partitioning: Sharding
  * For **READ**: add *Cache, Lb, Proxy, Indexing*
  * For **WRITE** : *Queue*, *Async*

### Some systems, frameworks
* *Envoy*
* *Istio* : open source implemenation of *Service Mesh*
  * http://istio.io
