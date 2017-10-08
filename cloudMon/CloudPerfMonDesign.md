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

## Remaining questions

## Existing framework
