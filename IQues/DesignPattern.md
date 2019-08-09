# Design patterns
## Cloud Design patterns
* resource: https://docs.microsoft.com/en-us/azure/architecture/patterns
* next : https://docs.microsoft.com/en-us/azure/architecture/patterns/circuit-breaker

### Ambassador
* Tags: **Design**
* My tag: **SF-Proxy-Type**
* *helper* services that send network requests *on behalf* of a consumer service
* like out-of-process *proxy* that is co-located with the client.
*  useful for *offloading* common client connectivity tasks such as monitoring, logging, routing, security (such as TLS), and *resiliency patterns* in a *language agnostic* way
  * write these common tasks in a different language
* *resiliency patterns* are: Retry(if *idempotent*), Circuit breaking, routing, etc
* So the *main service* could focus on the main business logic.
* The same code of  *Ambassador* could probably services more than one applications.
* *cons*: extra latency


### Anti-Corruption Layer pattern
* Tags: **Design**
* My tag:  **SF-Proxy-Type**,  **SF-transform**
* *Isolate* the different subsystems by placing an anti-corruption layer between them.
* The anti-corruption layer contains all of the logic necessary to translate between the two systems
* Not use: This pattern may not be suitable if there are no significant semantic differences between new and legacy systems.
* Ok, this is again a proxy type thing, but more like a translator, good if two subSystems want to communication but with different symantic, or a new version want to maintain the integration to legacy

### Availability patterns
* Tags : **Availability**
* defines the proportion of time that the system is functional and working.
* *Health Endpoint Monitoring*:
  * Implement functional checks in an application that external tools can access through exposed endpoints at regular intervals.
* *Queue-Based Load Leveling*
  * Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth intermittent heavy loads.
* *Throttling*
  * Control the consumption of resources used by an instance of an application, an individual tenant, or an entire service.

### Backends for Frontends pattern
* Tags: **Design**
* My Tags: **SF-DeCompose-type**
* Basically, *separate* backend for different frontend applications
  * so the backend can be specifically optimized for a given frontend.
  * and could be less complex and smaller
* *me*: kind of not in favor for this, very likely would repeat lots of code, and need to deploy and maintain double number of backend services. Why not make the frontends talk with backend in same language?

### Bulkhead pattern
* Tags: **Design**
* MY Tags: **SF-RiskSplit**
* *Isolate elements* of an application into *pools* so that if one fails, the others will continue to function.
  * like in boat, we have different separate spaces, one fails, the others are ok
* Partition service instances into different groups
* example: we have different connection pools for different workloads
  * like if  *update* connection has problem, and not release, and eventually exhausted the *update* pool, but the *retrieve* operations can still work well, if they have a different connection pool

### Cache aside
* Tags: **DataManagement**
* My Tags: **SF-PerfImp**
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

### Circuit Breaker
* A kind of *anti-retry*, do repeating retry to failed services, and which would not auto-healing in a short time
  * and so avoid cascading failures due to this (keeping on retrying)
* Make a *Proxy* over the services, monitor the number of recent failures and decide whether to allow the operation to proceed, or return an exception immediately.
* *note*: basically another wrapper things, monitoring the health of the underlay service, and decide what the strategy to use in different situation, like
  * switch between status of : OPEN, CLOSE, Half-OPEN (control the traffic, or filtering by whitelist, prorityies)

### Claim Check  
* Split a large message into a *claim check* and a *payload*
  * Send the claim check to the messaging platform
  * and store the payload to an external service ( like a cheaper storage service, a database system, etc)
  * **NOTE**: sounds like the idea of using block-chain to protect actual data
* Consideration
  * consider delete the payload after a period of time
*  For example, Event Hubs currently has a limit of 256 KB (Basic Tier), while Event Grid supports only 64-KB messages.

### CQRS
* Tags: **DataManagement**
* MyTages: **READ-WRITE-Separate**
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
  * resiliency and *eventual consistency*, how to make sure the READ replica is reflecting the latest writes
  * Consider applying CQRS to limited sections of your system where it will be most valuable
  * A typical approach to deploying eventual consistency is to use *event sourcing* in conjunction with CQRS so that the write model is an append-only stream of events driven by execution of commands.
* **MyNote**:
  * main idea, using different representation for *READ* and *WRITE* operations
  * usually, the *WRITE* part could be *event-driven, append-only*, and so *multiple versioning*
  * *WRITE* becomes a *log-like* thing, while *READ* is reading *View* of a given version
    * like in MDS, the read is always talking to  a given "Tx-sequence-num"

### Compensating Transaction
* **MyNOTE**:
  * ok, this is talking about the situtation that, a cloud application wants to do a work in multiple steps. which is common, e.g. use a workflow of a series tasks
  * but, each failure of the tasks, may lead to problem not only for this tasks, but previous tasks
  * sometimes it is untrivial to roll-back, since you are in a concurrent environment
  * so we need a smart *compensation logic* to do the cancelling
    * it may including triggering the *compensation logic* of a previous tasks, or giving user a chance to redo a task, but with a different option
  * it could be the "cancel" operation of a task in a workflow
  * cancelling of previous tasks may happen in exact the reverse order or not, or even in parallel, if possible

### Competing Consumers
* An application running in the cloud is expected to handle a large number of requests.
* Rather than process each request synchronously, a common technique is for the application to pass them *through a messaging system* to another service (a *consumer service*) that handles them *asynchronously*.
* At peak hours a system might need to process many hundreds of requests per second, while at other times the number could be very small
  * To handle this fluctuating workload, the system can run *multiple instances* of the consumer service
  *  However, these consumers must be *coordinated* to ensure that each message is only delivered to a single consumer
* *Solution*: just use a messsage in-between pool of applicaiton servers, and pool of *consumer services*
* *Issue, considerations*:
  * Message ordering
    * Design the system to ensure that message processing is *idempotent* because this will help to eliminate any dependency on the order in which messages are handled.
      * **idempotent : ->  order does not matter**
  * *resiliency*: **OK to failed and restart (a task)**
  * Detecting poison messages: **and block it from exec, and write to log**
  * How to pass the result back to application services? How to make sure it is completed  instead a in-progress middle state
  * consider the *Scaling* and the *reliability* of the message system, since your system is fully relying on it

### Compute Resource Consolidation
* Tags: **Design**
* Idea: Consolidate multiple tasks or operations into a single computational unit
  * Tasks can be grouped according to criteria based on the features provided by the environment and the costs associated with these features
  * A common approach is to look for tasks that have a similar profile concerning their scalability, lifetime, and processing requirements
* **MyNote**: sounds like a typical resource allocation problem, how to group tasks to the right compute, and then save cost, but may increase management burden, or risks to resilient to failures, and (irregular) resource consumming tasks, or malicious tasks (impact to others)
* This idea is trying to reduce the complexity of compute node management, increase utilization on under-utilized node, however, there multiple concerns over it:
* *concerns*
  * Security/isolation: since you put multiple tasks into single execution unit, there is a risk of security leak or one crash would lead to other crash
  * contention:  resource contention between tasks
  * release cadence: if one of the tasks has a more frequent release schedule, than you have a dillema
  * scalability and elasticity :
    * Many cloud solutions implement scalability and elasticity at the level of the computational unit by *starting and stopping* instances of units.
    * so Avoid grouping tasks that have conflicting scalability requirements in the same computational unit.


### Event Sourcing pattern
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


### External configuration store
* Move configuration information out of the application deployment package to a centralized location.
* and provide a query interface (for the configuration)
* Pros:
  * easy deployment
  * easy to share configuration between different applications
* Recommendations:
  * Ensure that the configuration interface can expose the configuration data in the required formats such as *typed values*, collections, key/value pairs, or property bags
  * How to make sure the configurations can only be accessed by only the appropriate users and applications
* **MyNote**: actualy there are many examples in real world for external/dedicated configuration store
  * FB has such a services, *ConfigDb* or something

### Federated Identity
* MyTags: **DelegateSideWork**
* Delegate *authentication to an external identity* provider. (IdP)
  * also known as "security token services" (*STS*)
  * external:  Login as "Facebook", "Google", etc
* Like the GDot thing we have
  0. a *service* register itself (or/and its tenant) to the Identity provider
  0. The *consumer* of the service send request to the Identity service, and receive a token
  0. The *consumer* uses this token when talking with *service*
* This model is often called *claims-based access control*.
* Concerns
  * The *IdP* could be single point failure --> need multiple copy
  * Extra perf cost --> consider co-located with Service (in the same data center)

### Gatekeeper
* Tags: **Security**
* MyTags: **Proxy-Broker**
* Protect applications and services by using a dedicated host instance that acts as a broker between clients and the application
  *  an additional layer of security, and limit the attack surface of the system.
* *GateKeeper* exposes Endpoints to end users
  *  The gatekeeper validates all requests, and rejects those that don't meet validation requirements
  * Use a secure communication channel (HTTPS, SSL, or TLS) between the gatekeeper and the trusted hosts

### Gateway aggregation
* Use a `gateway` to aggregate *multiple individual requests* into a *single* request
* The so-called `gateway` works like the DFS
* Issues and considerations
  * should not introduce service coupling
  * gateway should be located near the backend services (reduce latency)
  * Gateway could be the perf bottleneck, make sure to optimizing it greatly
  * must be *resilient*
  * use *Async*, so slow in back service would be make user angry
  * tracing! : distributed tracing using correlation IDs to track each individual call.
  * consider using *Cache*   

### Gateway Offloading
* *Offload* some features into an API gateway
  * particularly cross-cutting concerns such as:
    * certificate management, authentication, SSL termination, monitoring, protocol translation, or throttling.

### Gateway Routing
* Route requests to *multiple services* using a *single endpoint*
* Cons:
  * The gateway service may introduce a single point of failure, or perf bottleneck
  * ensure you don't introduce cascading failures for services
  * Gateway routing is *level 7*. It can be based on IP, port, header, or URL

### Health Endpoint Monitoring
* external service (health monitor services) can access through exposed endpoints (of your service) at regular intervals.
  *  health monitoring by sending requests to an endpoint on the application
  * Secure the endpoint by requiring authentication
* can be extended to *online diagnosis/testing* tool


### Index table
* Tags: **DataManagement**
* basically, create your own table to mimic the *secondary indexes* in RDBMS
* *Three* strategies are commonly used for structuring an index table
  0. complete denormalization:  duplicate the data in each index table but organize it by different keys
    * ppropriate if the data is relatively static compared to the number of times it's queried
  0. index to primiary key: create normalized index tables organized by different keys and reference the original data by using the primary key rather than duplicating it
    * index table as side-table, point to the main fact table
    * kind of like a standard snow-flake schema design
  0. create partially normalized index tables organized by different keys that duplicate frequently retrieved fields.
    * more or less like a *included index*, keep the most frequently used field with the index, and then point to fact table (primiary key) if you need other information
* use case for *shard key*
  * if you are using a hash-based sharding, you can use an index table with shard key (point to the fact table with shard key) to save the recomputation of the hashed shard key
* *Overhead* of maintaining index is huge, *not good* for where data keep on changing
  * including maintaining *consistency* between different indexing tables
* The balance of the data values for a field selected as the secondary key for an index table are highly *skewed* : when scanning (fact table) is too expensive


### Leader Election
* Implementing one of the common leader election algorithms such as the *Bully Algorithm* or the *Ring Algorithm*.

**NEXT** https://docs.microsoft.com/en-us/azure/architecture/patterns/leader-election

### Data Management patterns
*  the *key element* of cloud applications



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





#### Materialized View pattern
* pregenerating *result* for queries, or easy to build result for the queries *quickly*
* A key point is that a *materialized view* and the data it contains is *completely disposable* because it can be *entirely rebuilt* from the *source data* stores
* A materialized view is *never updated* directly by an *application*, (*READ-ONLY*) and so it's a *specialized cache*.
  *  consider using a *scheduled task*, an external *trigger*, or a *manual action* to regenerate the view.
  * such as when using the *Event Sourcing* pattern to maintain a store of only the events that modified the data, materialized views are necessary

#### Sharding
* horizontal partitions
* strategies
  0. Lookup strategy : implements a map that routes a request for data to the shard (likely purely ad-hoc mapping)
    * *mapping* between the *shard key* and the *physical storage*, or *virtual partitioning* ( more than virtual partition could map to one physical node)
    * fully controlled, easy to rebalance
    * permits scaling and data movement operations to be carried out at the user level, either online or offline ( could do something within tenant boundaries, or a group of given tenants)
  0. range strategy
    * groups related items together in the same shard, and *orders* them by shard key
    * but rebalance would be hard
  0. Hash strategy. (kind of more popular, I think)
    *  a better chance of more even data and load distribution
    * no need to maintain a map, just a hash function
    * but hash computation is an overhead, and rebalance is not easy too, makes scaling and data movement operations more complex
      * Isn't that we have *consistent hashing*?
  0. specials
    * based on attributes of *tenant*, you can have flexible strategies
    * like separate *highly active*, *very important*, *requring high security* tenants with other tenants

#### Static Content Hosting
* basically, put the *static part* (resources, static pages, etc) of the services to given storage
* issues, requirements:
  * The hosted storage service must expose an HTTP endpoint that users can access to download the static resources
  * consider using a content delivery network (CDN) to cache the contents of the storage container in multiple datacenters around the world.
  * IP address (of the storage device) might change, but the URL will remain the same.
  * Consider using a valet key or token to control access to resources that shouldn't be available anonymously.

#### Valet Key
* Use a token that provides clients with restricted direct access to a specific resource
* in the case, where your service serves untrusted client to access storage resource. In order not let the application itself to do data transfer, you would like to let the client to upload/download the data directly to the storage, which mean the client need have direct access to the storage
* But then application would lose the access control, which is not good
* One typical solution is to restrict access to the data store’s public connection and *provide the client with a key or token* that the *data store* can *validate*
* a valet key:  It provides time-limited access to specific resources, like *jwt*
  * and allows only predefined operations

### Design










#### Pipes and Filters
* Decompose a complex task into a series of separate elements that can be reused
  * Break down the processing required for each stream into a set of separate components (or *filters*), each performing a single task
  * The filters don't even have to be in the same datacenter or geographic location
* **NOTE**: kind of like how we break the big task into reusable tasks and assemblize into workflow (pipeline of tasks)
* Challenges
  * Complexity, Reliability, Idempotency a must, Repeated messages, Context and state

#### sidecar
* put some "side" functionality in a side application,  co-located on the same host as the main service, and have the same life-cycle  
* simplify main service, out-sourcing works like *monitoring, logging, configuration, cron-job, platform abstraction* to the side-car application
* can be written in different languages, and scale independently

#### Strangler
* how to retire an old system?
* Incrementally replace specific pieces of functionality with new applications and services
* Create a façade that intercepts requests going to the backend legacy system.
* The *strangler façade* routes these requests either to the legacy application or the new services
* **NOTE**: like the different service dispatcher we designed for DBS (whether go to V1.0 or V2.0)
*  Existing features can be migrated to the new system gradually, and consumers can continue using the same interface, unaware that any migration has taken place
* when the migration is complete, the strangler façade will either
  * go away or
  * evolve into an adaptor for legacy clients

#### TBD marker :https://docs.microsoft.com/en-us/azure/architecture/patterns/category/messaging
 * /category/design-implementation

#### Cache-Aside
* For caches, if not provide built-in *read-Through* or *write-through* abilities   




## Cloud best practices
* resource: https://docs.microsoft.com/en-us/azure/architecture/best-practices
