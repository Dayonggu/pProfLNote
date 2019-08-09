# Things about programming

## Multi-threading
* Assembly line style:
  * *Actor* make up `DAG`
  * Reactive, event driven
    * `Akka, Node.js Vert.x`
  * Actor and Channel
* `Volatile` keyword
  * block compiler to do too smart *reordering*  , with *Happen - before* guarantee
  * Reads and Writes to other variables cannot reordered to occur after a *WRITE* to `Volatile` variable
  * Reads and Writes to other variables, cannot reordered to occur before a *READ* of `Volatile` variable
