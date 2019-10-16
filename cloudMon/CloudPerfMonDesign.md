# Design basics of perf monitoring System
## What to monitor
### Metrics
  * Server Metrics
    * tranditionals
  * dependencies
    * IaaS perf
    * Storage layer perf
    * Underline services
      * Like Queue service
  * Application specifics
    * Requests : count, rate
    * Internal code level metrics:
      * like the length of a Queue
      * like other Engine-DMV metrics
  * Errors/Alarms
  * Logs:
    * consider change logs to Metrics
    * associated a *Metrics* to a log line: parsing logline for special info and fire a Metric for it
  * four **golden** signal
    * traffic
    * latency
    * Errors
    * Saturation (how full is your service)


## Functionalities
* Dashboard, select periods of display  
* Log aggreation, metriclize, searching, classification
* Ways to Tracing a transaction
  * associated everything regards one specific pieces of *REQUEST* or *transaction*
* Error/alarm. reporting
  * send emails. sms
  * threshold is configurable (per customer)
* Infrastructure health
* full stack Monitoring
  * some how mapping different layers metrics ,together to discover the problems

## How (Architectual)
* Rules
  * prefer simpler, lightweight monitoring, and good post-analyze system
    * avoid “magic” systems that try to learn thresholds or automatically detect causality
  * avoid complex dependency hierarchies
* SLO : service level object
  * *Latency* : tency over one minute must be between 25 and 100 ms for 99% of requests.
  * *Availability Indicators*:
    *  Mean Time Between Failures (MBTF).
    * Mean Time To Recover (MTTR)
## Remaining questions

## Existing framework
