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
