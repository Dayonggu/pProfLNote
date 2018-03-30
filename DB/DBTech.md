# Database Techniques
## Paper selections
* **Tapir** : consistent tx with inconsistent replication
  * https://syslab.cs.washington.edu/papers/tapir-tr14.pdf
  * *key idea*
    * claim using both "distributed tx protocol" and "replciation protocol" to maintain consistency is *redundant*
    * invent a *IR* : inconsistent replication, operations through IR in two modes    
      * *inconsistent* mode: operations can run in any order
      * *consensus* mode: operations run in any order, but return a *single consensus* result
        * allow the application (using distributed tx protocol) to decide outcome (resolve) of conflicts ( sounds like B-C idea?)
      * Client side: invokeInconsist(op);  invokeConsensus(op, decide(results))->result : `decide()` to resolve conflict (app-side)
      * replica side: execInconsist(op);  execConsensus(op) -> result (of this replica)  ...  Sync(R) Merge(d,u) -> record
  * full path sounds like:
    * Client.invokeInconsist(op)
      * ==> Rep1.invokeInconsist(op) || Rep2.invokeInconsist(op) || ...
    * client side get a list of results,  invokeConsensus(op, decide(results))->result. make a decision on the result
      * ==> Rep1.execConsensus(op) ->result || Rep2.execConsensus(op) ->result || ...
    *  When need recovery : Rep1.Sync(R) Merge(d,u) -> record,  Rep2.Sync(R) Merge(d,u) -> record, ...
      * keep Replicas eventually converge  
  * **IR** protocol, use 4 sub-protocol
    * Operation Processing 
