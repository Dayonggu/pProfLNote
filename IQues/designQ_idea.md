# Design system

## Specific functionaly model

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
