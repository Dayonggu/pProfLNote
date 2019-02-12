# Learning Note of Go
* following
  * *The Go Programming Language*
    * tbc: c4.4

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
type Point struct {    X, Y int}
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


### Lib-Type tips
* frequently used packages:
  * `os`, `fmt`, `bufio`, `strings`, `math`, `net/http`
  * `io/ioutil`, `image`
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
