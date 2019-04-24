# Design patterns
## Cloud Design patterns
* resource: https://docs.microsoft.com/en-us/azure/architecture/patterns

### sidecar
* put some "side" functionality in a side application,  co-located on the same host as the main service, and have the same life-cycle  
* simplify main service, out-sourcing works like *monitoring, logging, configuration, cron-job, platform abstraction* to the side-car application
* can be written in different languages, and scale independently
