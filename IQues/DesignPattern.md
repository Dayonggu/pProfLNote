# Design patterns
## Cloud Design patterns
* resource: https://docs.microsoft.com/en-us/azure/architecture/patterns

### Availability patterns
* defines the proportion of time that the system is functional and working.
* Health Endpoint Monitoring:
  * Implement functional checks in an application that external tools can access through exposed endpoints at regular intervals.
* Queue-Based Load Leveling
  * Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth intermittent heavy loads.
* Throttling
  * Control the consumption of resources used by an instance of an application, an individual tenant, or an entire service.

### Data Management patterns
*  the *key element* of cloud applications

#### Cache aside
* Load data on demand into a cache from a data store
* application is responsible for reading and writing from the database and the cache doesn't interact with the database at all
  * like the caches you saw in M-Client
  * Application to simulate a *Read-Through*: if not found in cache, read from DB, and put into cache
  * when *Update*, application to invalid cached item
    * *NOTE*: application updates the database directly *synchronously*
* so  allows you to *issue complex database queries* involving joins and nested queries and manipulate data any way you want.
* *Consistency* : Implementing the Cache-Aside pattern *doesn't guarantee* consistency between the data store and the cache
* *Local Caching* : A cache could be local to an application instance and stored in-memory.
  * so it might be necessary to expire data held in a private cache and refresh it more frequently.

#### Read-Through and Write Through cache
* Another cache pattern in *distributed cache*
  * Application only talks with cache, not aware of DB
  * application treats cache as the main data store and reads data from it and writes data to it
  * The cache is responsible for reading and writing this data to the database, thereby relieving the application of this responsibility.
  * can *write behind* : Write-behind lets your application quickly update the cache and return. Then, it lets the cache update the database in the background.
* modern cache solution usually provide this
* you just need to impl: `SqlReadThruProvider/SqlWriteThruProvider`
* then usually just `cache.get(key, provider, READ_MODE=READ_THRU)`
* more resource about caching: https://docs.microsoft.com/en-us/azure/architecture/best-practices/caching

#### CQRS
* Command and Query Responsibility Segregation (CQRS) pattern
* Separate the *read* and *write* interface, can maximize performance, scalability, and security
* Traditionally, all *CRUD* is talking with the same data representation, AKA, *DTO*
* In *CQRS*
  * the data models used for querying and updates are different
  * The query model for reading data and the update model for writing data can access the same physical store, perhaps by using SQL views or by generating projections on the fly
  * However, it's common to separate the data into *different physical* stores
    * The read store can be a read-only replica of the write store, or after a *transformation* --> different structure
    * can have multiple read-only replicas of the read store
    * Separation of the read and write stores also allows each to be scaled appropriately to match the load
* *Issues*
  * resiliency and eventual consistency, how to make sure the READ replica is reflecting the latest writes
  * Consider applying CQRS to limited sections of your system where it will be most valuable
  * A typical approach to deploying eventual consistency is to use *event sourcing* in conjunction with CQRS so that the write model is an append-only stream of events driven by execution of commands.

#### Event Sourcing pattern
* handling operations on data that's driven by a sequence of *events*
* each of which is recorded in an *append-only* store
* events are persisted in an event store that acts as the *system of record*
  * kind of like *logging*
* The event store typically publishes these events so that consumers can be notified and can handle them if needed.
  * so different *consumers* could create different copies of *Read View* of the data  
* Event sourcing can help prevent concurrent updates from causing conflicts because it avoids the requirement to directly update objects in the data store. However, the *domain model* must still be designed to *protect* itself from requests that might result in an *inconsistent state*.
* Issues:
  * only eventually consistent
  * event store is immutable, so: The only way to update an entity to undo a change is to add a compensating event
  * If the format (rather than the data) of the persisted events needs to change, perhaps during a migration, it can be difficult to combine existing events in the store with the new version
    *  It might be necessary to iterate through all the events making changes so they're compliant with the new format
* *consider*  
  * using a version stamp on each version of the event schema
  *  Adding a timestamp to every event can help to avoid issues, or to annotate each event resulting from a request with an incremental identifier
  * Event publication might be “at least once,” and so consumers of the events *must be idempotent*. They must not reapply the update described in an event if the event is handled more than once

### sidecar
* put some "side" functionality in a side application,  co-located on the same host as the main service, and have the same life-cycle  
* simplify main service, out-sourcing works like *monitoring, logging, configuration, cron-job, platform abstraction* to the side-car application
* can be written in different languages, and scale independently


## Cloud best practices
* resource: https://docs.microsoft.com/en-us/azure/architecture/best-practices
