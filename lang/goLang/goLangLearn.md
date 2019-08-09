# Learning Note of Go
* following
  * *The Go Programming Language* : **DONE**

  https://github.com/Dayonggu/pProfLNote/invitations

## Fundamentals
### Language overall specifics
* **GO** has no class hierarchies, no class, use *composition* of simpler ones
* communicating sequential processes (*CSP*), embodied by goroutines and channels   
* With a GC

### Basic programming concepts
* Slices: like Array, subarray can be access as sliceA[m:n]
  * cmd argument is one: `os.Args[xxx]`, like `os.Args[0]` is the name of the cmd, and `os.Args[1]` is the first argument
* `range` : create a <idx, val> returns, can be assigned to two var, eg:
  * `idx, arg := range os.Args[1:]`
  * if you don't need the `idx`, you can just write `_, arg := range os.Args[1:]`
  * somewhat like Python did
* other examples of return 2 values (*Tuple assignment*):
  * `file, err := os.Open(filePath)`
  * `resp, err := http.Get(url)` : get response of url, or error
    * `b, err := ioutil.ReadAll(resp.Body)`: read the body or error from response
* *Tuple assignment* : can assign multiple vars
  * can assign an array: `medals := []string{"gold", "silver", "bronze"}`
* concurrency
  * make a channel: `ch := make(chan string)`
  * start goroutines: `go fetch(url, ch)` : here `fetch(url, ch)` is a function, declared as
    * `func fetch(url string, ch chan<- string)` in which `ch <- xxx` means something send to channel
  * receive from channel:  `fmt.Println(<-ch)`
* Http Server
~~~
 http.HandleFunc("/", handler) // each request calls handler
 // you can add more handler ( to different url, like /count)
 // (http.ListenAndServe: starting listen and serve
 log.Fatal(http.ListenAndServe("localhost:8000", nil))
~~~
* named types: user defined types
~~~
type Point struct { X, Y int}
~~~

#### Data Type
* `&^` : bit clear
* even `int16` and `int32` cannot be convert automatically
* type conversion: *T(x)*, eg int(xVar), f=1.31  y = int(f)  => y = 1
* *octal number* starts with "0", hex starts with "0x",  c-like ;
* *Complex number* : `var x complex128 = complex(1, 2)` for 1+2i
  * import `math/cmplx`: The math/cmplx package provides library functions for working with complex numbers, such as the complex square root and exponentiation functions
  * *real(complexNum)* got the real part *imag(complexNum)* got virtual part  
* utf: utf8.RuneCountInString(s), count unicode char as "1", although it may take 3 bytes
* ` strconv package` does the string -- number conversion
  * strconv.Atoi("123")
  * strconv.ParseInt("123", 10, 64) // base 10, up to 64 bits
* *const* : const pi = 3.14159
* Array:
  * `var a [3]int`; `var q [3]int = [3]int{1, 2, 3}`
  * `q := [...]int{1, 2, 3}` : array length is decided by numbers initialized
* **slice**
  * variable-length sequences whose elements all have the same type: `[]Type`
  * back by a underlying array, but three components: a pointer, a length (`len(x)`), and a capacity(`cap(x)`);
  * `slice operator s[i:j],where 0 ≤ i ≤ j ≤ cap(s),`
  * Slicing beyond *cap(s)* causes a `panic`, but slicing beyond `len(s)` extends the slice
  * Unlike arrays, slices are *not comparable*, so we cannot use == to test whether two slices contain the same elements
  * built-in function `make` creates a slice of a specified element type, length, and capacity
    * `make([]T, len, cap) `
  * built-in `append` function appends items to slice
* *map*
  * `map[string]int{}` : empty string -> int map;  or use  `make(map[string]int)`, you *allocate* a map
  * remove: `delete(mapName, key)`
* *struct*
~~~
type tree struct {    
  value       int    
  left, right *tree
}
type address struct {
    hostname string
    port     int
}
// init a address struct : address{"golang.org", 443}
~~~
  * The == operation compares the correspondingfields of the two structs in order
  * *anonymous fields* : declare a field with a type but no name, access by typename directly
* json support
  * `data, err := json.Marshal(movies)` : where *movies* is an array of struct, so make it pretty well suit for translating to json ( the *data*)
  * *unmarshal* : `json.Unmarshal(data, &titles)`, where titles is `[]struct{ Title string }` : this would get only the "Title" field in the json data to the struct array "titles"
* template
  * text or html
  * A template is a string or file containing one or more portions enclosed in double braces, {{...}}, called actions.
   * actions trigger other behaviors. Action are data field name with `|` (pipeline) to statement/fuction call which process the field
* end

#### Functions
* declare
~~~
func name(parameter-list) (result-list) {    
     body
}
~~~
  * Note: the result could be a list: eg `func Parse(r io.Reader) (*Node, error)`
  * Parameter name does not matter, order matter
  * Arguments are passed by *value*
* *function value* :
  * function can be assigned to variable like a "value" in function type
  * a variable f's function type is decided by the first time it was assigned to a function, and thus you cannot assign other functions with *different signature* to it later
* *Anonymous Functions* or *lambda*
  * *Named* functions can be declared only at the *package level*
  * define a function at its point of use, and can access variable in the parent function (same scope at the define point), eg: `strings.Map(func(r rune) rune { return r + 1 }, "HAL-9000")`
  * Anonymous function can be the result type of a function :
    * define  `func squares() func() int `, which could return `return func() int { body}`
    ~~~
    func squares() func() int {  
        var x int // note: this var state would be kept at each call   
        return func() int {  
                x++       // this lead the outter "x" value be increased
                 return x * x    
                 }
               }
    ~~~
    * The squares example demonstrates that function values are notjust code but can have state.
  * *variadic function* : variable number of arguments
    * `func sum(vals ...int) int `
  * Deferred Function Calls
    * *defer* mechanism:
      * with a `defer` statement, the function or expression,  are evaluated when the statement is executed, but the actual call is deferred until the function that contains the defer statement has finished
    * A defer statement is often used with paired operations like open and close,connect and disconnect, or lock and unlock to ensure that resourcesare released in all cases
    * put `defer resp.Body.Close()` right after got the resp. And it works like you put it into `finally`, would be executed at the end of the parent scope, eg:
    ~~~
    func lookup(key string) int {
         mu.Lock()
         defer mu.Unlock()
         return m[key]
    }
    ~~~     
    * and even be put in front of an inner function, so it would execute at the end of the parent function
  * *panic* and *recover*
    * like throw and catch exception

#### Method
* example of method: `func (p Point) ScaleBy(factor float64) `
  * *P* is the receiver, somewhat like the *this* in java,
  * you can do:  `p Point;  p.ScaleBy(0.1);`
  * you can also do `func (p *Point) ScaleBy(factor float64)`, now the *receiver* is a *pointer* to *Point*
    * attach the *method* to the *pointer* type of type `Point`
    * `r := &Point{1, 2}; r.ScaleBy(0.1)`
    * and if the variable is type of Point not pointer to it, like ` p = Point{1,2}`
      * you can still do `p.ScaleBy(2)`, not necessarily do `(&p).scale(2)`, because the **compile implicitly does the conversion**
      * while which is actually a little confusing (sacrifice the concept uniform)
  * `Nil` Is a Valid Receiver Value; specially if nil is a meaningful *zerovalue* of the type, like an empty list or tree
  * *method* of a type is defined out of the type {}, but by writing the type as `Receiver`

#### Interface
* *Implicit* satisfying
  * You don't need to clear say type A implements Interface I
  * if type A actually implement all methods required by Interface I, then type A satisfy interface I
* Interface values:
  * An interface has a
    * *type* : a type descriptor  *T*
    * *value* : pointer to dynamic value with type *T*
  * eg: `var x interface{} = time.Now()`
     * we got a interface variable *x*, with type is *time.Time* and value point to *Time* struct instance
* *sort* by interface
  * you could implement an sort-helper var for a Type T, that implement the `Len, Less, Swap` methods required by the `sort` interface, and then sort the slice of T
  * in other word, write the comparator for the Type, eg.
  * We have a Type `Track`, and a slice of pointers to it: `tracks []*Track`
  * we then define a new var `byArtist` for it which satifies the *sort* interface, as
    * `var byArtist []*Track`, and implement the `Less(i, j int)` method, to give comparison of by the *artist* field of a *Track* object
  * Then we can sort the slice `tracks` by `sort.Sort(byArtist(tracks))` or `sort.Sort(sort.Reverse(byArtist(tracks)))` in the reverse order
* The `http.Handler` Interface
  * A http handler need to have `ServeHTTP(w ResponseWriter, r *Request)`
  * thus this handler could be put in : `http.ListenAndServe("localhost:8000", handlerVar)` then start the serving
  * or use `mux := http.NewServeMux()` to create a ServeMux which wrapper out different path for the http server and do the work:
    * mux.Handle("/list", http.HandlerFunc(db.list))
    * mux.Handle("/price", http.HandlerFunc(db.price))
    * http.ListenAndServe("localhost:8000", mux)
    * where *db* is the actual handler, implement the separate function for different API calls
* type switch : `switch x.(type)  {}`

#### **Goroutines and Channels**
* `go func()`
* *channels* are the connections between goroutines, sending values
  * Each channel is a conduit for values of a *particular type*, called the channel’s *element type*.
  * `ch := make(chan int) // ch has type 'chan int'` : *make* is built-in function
    * `ch = make(chan int, 3) // buffered channel with capacity 3` **buffered** channel
  * A channel has two principal operations, *send* and *receive*
    * `ch <- x  // a send statement`, `x = <-ch // a receive expression `
  * `close(ch)` to close a channel:
    * After a channel has been closed, any further send operations on it will *panic*
* *channel pipeline* : a way to breakup process to sub-processes
* *unidirectional channel*: channel can only send or receive, eg
  * `func squarer(out chan<- int, in <-chan int)`: first is send-only, second is receive-only
  * Conversions from bidirectional to unidirectional channel types are permitted in any assignment.There is no going back
* Channel return value could be having no receiver, in which we just use this to *sync*, make sure the goroutines finish.
* `sync.WaitGroup` : cross goroutines counters
  * in parent function: `var wg sync.WaitGroup; wg.Add(1);`
  *  and in gorountine: `wg.Done()` ; reduce one
  * then` wg.Wait()` to wait for the counter to zero
* *cancellation of rountines* : broadcasting cancellation by `close` a `done` channel; and each goroutines running need to check the channel is cancel or not.
~~~
func cancelled() bool {
      select
      {    
        case <-done:        return true    // "done" is the channel
        default:        return false    
      }
}
~~~

#### Concurrency with shared var
* *Race Condition*
  * *confinement* : make only one gorountine access a given variable
    * banking example: every deposit is to send deposit value to a channel, and a special rountine read from the the channel and change the real account balance
      * like make the real balanace changing operation sequential.
    * it is also common to share a variable between gorountines in a *pipeline* by passing its *address* from one stage to the next over a *channel*
  * *sync.Mutex* and *sync.RWMutex*
  ~~~
  var (   
     mu      sync.Mutex // guards balance   
    balance int
    )
    later do: mu.Lock() --> real operations --> mu.UnLock()
    or defer mu.Unlock() to make sure UnLock happen in any cases
    // for readWRite
    var mu sync.RWMutex
    mu.RLock()
    defer mu.RUnlock()
  ~~~
  * *sync.Once* : lazy initialization
    * `var loadIconsOnce sync.Once` : a *Once* consists of a `mutex` and a `boolean variable` that records whether initialization has taken place
    * `loadIconsOnce.Do(loadIcons)` :
      * Each call to `Do(loadIcons)` locks the mutex and checks the boolean variable, only load once when the boolean is still `false`
      * we can avoid sharing variables with other goroutines until they have been properly constructed
* *Race detector*
  * Add the `-race` flag to your `go build, go run, or go test`
  * The code would be instrumented and generate a stream of  of sync-related events
  * The race detector reports all data races that were *actually executed*
* *Gorountines vs OS Threads*
  * unlike an OSthread, a goroutine’s stack is not fixed; it grows and shrinks as needed
  * OS Thread shecduled by Kernel, thus there is a context-switch cost
  * The Go runtime contains its own scheduler that uses a technique known as m:n scheduling
    * m goroutines on n OS threads
  * `runtime.GOMAXPROCS`: how many OS-threads are used for scheduling Go code, configurable  
  * *gorountine* has no identity, you cannot do thread-local-like things

#### Package and tool
* every go source, starts with a definition of *package*
  * For example, every file of the math/rand package starts with `package rand`
  * Conventionally, the package name is the last segment of the import path
  * a package defining a command (an executable Go program) always has the name *main*
    * so `go` would know where to start your program
  * some files in the directory may have the suffix `_test` on their package name if the file name ends with `_test.go`
    * so `go test` know where to start the test
  * some tools for dependency management append *version number* suffixes to package import paths,
* *blank import*: import a package and rename to `_`; If you just want to trigger the initialization of that package, and would not directly invoke anything from that package (explicitly)
* *go* tool
  * `go help`, `go env`; `go get`; `go doc`
  * `go list`: reports information about available packages. `go list -json [packageName]`

#### go Testing tool
* must import `testing`
* looking for:  In a package directory, files whosenames end with `_test.go`: *Test*, *Benchmark* and *Example*
* `Test` functions: start with `Test`: `func TestSOMEFEATURE(t *testing.T)`
  * `go test -v`: run all tests
  * `go test -v -run="French|Canal"` : run all tests where names start with "French" or "Canal" after the prefix `Test`
  * Test failure messages are usually of the form "f(x) = y, want z": y is actual result
  * `t.Errorf` (will continue to next test case ) and `t.Fatalf` (which stop all test)
  * sometimes, we need external test package to avoid import dependency conflict, so called `XTestGoFiles`
    * `go list -f={{.XTestGoFiles}} [packageName]`
  * *Coverage* :
    * `go tool cover`
    * `go test -v -run=Coverage -coverprofile=c.out [packageName]`
    * `go tool cover -html=c.out`
* `Benchmark` function : perf measure
  * `func Benchmark[ThingToMeasure](b *testing.B)` :  Note `*testing.B` provides lots of methods:
    * `testing.B` parameter provides methods to stop, resume, and reset the timer, but these are rarely needed.
  * run it: `go test -bench=.`
  * *profiling*
    * `go test -cpuprofile=cpu.out`
    * `go test -blockprofile=block.out`
    * `go test -memprofile=mem.out`
    * Once we’ve gathered a profile, we need to analyze it using the `pprof` tool
* `Example` functions

#### Reflection in Go
* `relect` pacakge: *reflect.Type* and *reflect.Value*
  * `reflect.TypeOf`: `t := reflect.TypeOf(3)  // a reflect.Type` which is the *type descriptor*
  * `v := reflect.ValueOf(3) // a reflect.Value`
  * ` switch v.Kind()  case reflect.Int, etc`
  * `b := reflect.valueOf(x)  b.canAddr() // change where b type is addressable`
  * if x is slice type? `reflect.valueOf(x).Index(i)`
* reflect can be used to *change*
  * eg:
  ~~~
  x := 2   d := reflect.ValueOf(&x).Elem()     // d refers to the variable x
  px := d.Addr().Interface().(*int)   // px := &x; get d address, and consider as a int pointer
  *px = 3                             // x = 3, here we change the value of x
  fmt.Println(x)                      // "3"
  ~~~
  * or you can use the `set`: like `d.Set(reflect.ValueOf(4))`
  * or though the element
  ~~~
  d := reflect.ValueOf(&x).Elem()
  d.SetInt(3)
  fmt.Println(x) // "3"
  ~~~
* Display the methods of a type
  * `v = reflect.ValueOf(x)`
  * get the type `t=v.Type()`
  * get num of methods: `numMethod = v.NumMethod()`
  * for each method: `v.Method(i).Type`
* Cautions!!
  * Reflect code could be *fragile*, problem only be exposed as panic while running (not at compile time)
  * reflection cannot be subject to static type checking,heavily reflective code is often *hard to understand*. --> Better with good document, and encapsulation
  * reflection-based functions may be one or two orders of magnitude *slower* than code specialized for a particular type.  

#### Low-Level programming
*  Go scheduler freely moves `goroutines` from one thread to another.
* pointers to variable are transparently updated while GC
* But sometimes, you want to do some *unsafe* thing, so  the `unsafe` package is here to help
* `unsafe` pacakge
  * `unsafe.Sizeof, Alignof, and Offsetof`
  * *unsafe pointer*: `unsafe.Pointer(&x)`; `unsafe.Pointer(x.unsafeAddr())`
    *  can hold address of any type, not a given type as the normal pointers
* Calling C code, through *foregin function interface (FFI)* like
  * *cgo*, a binding tool for C functions ; `import "C"`: go build tool see the "C" here, and preprocess this file with *cgo*, before normal Go building process.
    * A temp package will be generated, contains *Go declaration* for all C functions/types used
  * *SWIG*
* end

### Misc tips
* frequently used packages:
  * `os`, `fmt`, `bufio`, `strings`, `math`, `net/http`, `testing`, `log`
  * `io/ioutil`, `image`, `html/template`, `text/template`
  * `errors`, `sort`, `syscall`, `sync`, `reflect`, `runtime`
  * `database/sql`, `encoding/xml`
* `strings.join(A_slice, B_slice)` : can also be just String or a slice of string
* `make(map[string]int)` : this would give you a {String -> Int} map
* variable names *case* matter, and first letter *Upper* case means cross-package visible, *lower-case* means within package, if declared with in *func*, it is local.
* variable init can be done one step for multiple, eg.
  * `var b, f, s = true, 2.3, "four" // bool, float64, string`
* `:=` is a declaration, whereas `=` is anassignment

### install/cmd/resources
* go `https://golang.org/doc/install`
* or `sudo snap install go`
* check
  * `go version`
  * `go build` : in the root of project src
  * `go run helloworld.go` : run directly
* ssample code from goLang book: `:~/go/src/gopl.io`
* Run Goprograms from the web pages: https://play.golang.org/

## Concurrency programming
