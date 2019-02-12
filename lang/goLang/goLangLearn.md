# Learning Note of Go
* following
  * *The Go Programming Language*
    * tbc: c6.4

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
    * `r := &Point{1, 2}; r.ScaleBy(0.1)`
  * `Nil` Is a Valid Receiver Value; specially if nil is a meaningful *zerovalue* of the type, like an empty list or tree
* end

### Lib-Type tips
* frequently used packages:
  * `os`, `fmt`, `bufio`, `strings`, `math`, `net/http`
  * `io/ioutil`, `image`, `html/template`, `text/template`
  * `errors`
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
