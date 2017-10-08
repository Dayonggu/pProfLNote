# Architectual level info for Cloud Monitoring

## Requirements/specials of Cloud Monitoring

### Specials over tranditional Monitoring
* you may not have full access to underline system
  * for example, you want to monritoring over aWS, Heroko, Azure, etc
* Support auto-scaling
  * That says: you probably need to detect the scale change of the system being monitored
  * like detect new Nodes/VMs
* smart APM metrics, not trandition server metrics
  * SaaS application-monitoring solutions should combination of the following:
    * APM : code level app Performance visiblity
    * Trascation tracing, logs,
    * Errors, Alerts
    * Metrics: Server, appliation, custom metrics
* dependencies:
  * Service Bus, Table Storage, Redishit,  SQS, ...
  * Your appliation is based on what type of infras


## Appendix

### Terms

* *APM* Application Performance Monitoring

### APM products
#### NewRelic
#### SelectStar
#### Retrace
* Transcation tracing
  * re-trace what your code is doing
  * WebRequest, Key methods in code, dependencies your code called (SQL, caching, RPC, Restful calls)
  * Application Error/ warning
  * Logging statements
* Log aggregation, Searching, and Monitoring
* App Errors/alarms:  reporting, tracking
* Metrics
  * Windows perf counters, linux perf Counts
  * Custom app metrics
  * app framework metrics
    * GC statistics, Context Switches, requested Queue, Request/sec,
    * JMX, monitoring beans
    * special to apps: like batch sizes of data, or prcess
  * change logline to metrics
    * e.g.: search for special log markers, and consider as a metric, etc
* Monitoring application SLA
* Monitoring application's dependencies performacne metrics:
  * like SQL db, Redis, Elasticsearch, ...
* Integrated alerts
  * Email, SMS, notifications


### Online resources/links
* `https://stackify.com/application-monitoring/`
* `https://florin.myip.org/blog/monitoring-cloud-part-2-architecture`
* `http://www.aosabook.org/en/index.html`
