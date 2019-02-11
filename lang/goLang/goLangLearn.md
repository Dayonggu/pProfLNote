# Learning Note of Go
* following
  * *The Go Programming Language* 

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
* other examples of return 2 values:
  * `file, err := os.Open(filePath)`
  * `resp, err := http.Get(url)` : get response of url, or error
    * `b, err := ioutil.ReadAll(resp.Body)`: read the body or error from response
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
type Point struct {    X, Y int}
~~~
*  TBD: Charpter 2

### Lib-Type tips
* frequently used packages:
  * `os`, `fmt`, `bufio`, `strings`, `math`, `net/http`
  * `io/ioutil`, `image`
* `strings.join(A_slice, B_slice)` : can also be just String or a slice of string
* `make(map[string]int)` : this would give you a {String -> Int} map

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
