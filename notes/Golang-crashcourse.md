# Golang

- [Golang](#golang)
- [project basics](#project-basics)
  - [env](#env)
  - [package management](#package-management)
    - [go module](#go-module)
  - [Standard Project Layout](#standard-project-layout)
- [language basics](#language-basics)
  - [types](#types)
    - [array](#array)
    - [slice](#slice)
    - [map](#map)
    - [struct](#struct)
    - [interface](#interface)
    - [channel](#channel)
  - [variable](#variable)
    - [new/make](#newmake)
  - [control flow](#control-flow)
  - [function](#function)
  - [goroutine](#goroutine)
    - [concurrency control](#concurrency-control)
  - [error handling](#error-handling)
  - [stack/heap](#stackheap)
  - [reflection](#reflection)
  - [misc tools](#misc-tools)
    - [print](#print)
    - [io](#io)
    - [race detection](#race-detection)
    - [go generate](#go-generate)
    - [text/template](#texttemplate)
    - [go test](#go-test)
    - [pprof](#pprof)
- [pitfalls \& tricks](#pitfalls--tricks)
  - [go version](#go-version)
  - [variadic params](#variadic-params)
  - [interface](#interface-1)
  - [protobuf](#protobuf)

# project basics

## env

- `go env`
    
- `go version`
    
- `GOROOT` is where Go installs
    
- `GOPATH` is where your project lives
    

## package management

- Every `go` file belongs to one package.
- Package name is usually folder name. `main` is special package which go builds it into exe.
- Declare your package at first line of code file `package xxx`
- Search other packages on [pkg.go.dev](https://pkg.go.dev/)
- Download remote packages via `go get`
- Import local packages via `import`
    - `import "github.com/xxx/xxx"`, Go will fetch it from github
    - `import _ "packageName"` only call init()
    - `import . "packageName"` omit
    - `import alias "packageName"` give a new name
- Go compiler finds packages in such order:
    - `GOROOT` (fmt...)
    - `GOPATH`

### go module

After go 1.11, a new way of managing dependency is introduced, core concept is `module` and `go.mod` file.

- `module` is a set of go packages defined in a file `go.mod` at the root folder of this module.
    - subfolder automatically included as the module's package using subfolder's path as package name.
- Your `module` can live outside `GOPATH/src`.
- Use `go mod init [module_name]` to init `go.mod`
- Use `go list -m all` to list current module's dependency
- Downloaded modules lie in `GOPATH/pkg/mod` locally
- Use `go mod tidy` to remove unused dependency
- `go build, go test, ...` adds dependency to `go.mod` automatically

A sample of go.mod

> module example.com/hello
> 
> go 1.12
> 
> require (
> golang.org/x/text v0.3.0 // indirect
> rsc.io/quote v1.5.2
> )

## Standard Project Layout

[Github:Standard Go Project Layout](https://github.com/golang-standards/project-layout/blob/master/README_zh.md)

# language basics

## types

- Value type: `built-in types, array, struct`
- Reference type: `slice, map, channel, interface, function`
- No `enum`, use `const+iota` instead
- type embedding (act like inheritance)
- type assertion `cat, ok := animal.(Cat)`, or `swtich t:= animal.(type)`
- reflect.TypeOf(x)

For type assertion, see [How to check variable type at runtime in Go language](https://stackoverflow.com/questions/6996704/how-to-check-variable-type-at-runtime-in-go-language) and [X does not implement Y (... method has a pointer receiver)](https://stackoverflow.com/questions/40823315/x-does-not-implement-y-method-has-a-pointer-receiver) for reference.

### array

- Pass by reference to avoid copy whole array

```go
var arr [5]int
var arr2 := [...]{1,2,3,4,5}
arr = arr2 // value copy
var arr2D := [2][2]int {{1,2}, {3,4}}
```

### slice

- Size = 3 pointers. a pointer to underlying array, a length, a capacity.
- Can be `nil`
- Suggest length == capacity

When capacity is not enough, Go doubles the capacity and create a new underlying array for this slice. This can cause **unintentional** effect.

```go
var s []int // a nil slice
var ps *[]int = new([]int) // points to a nil slice. uncommon

s := []int{1, 2, 3} // len(s) = 3, cap(s) = 3
s = append(s, 4) // len(s) = 4, expanded cap(s) = 6

s2 := s[2:4] // len(s2) = 2, cap(s2) = 4 (cap(s)-2)
s2 = append(s2, 44, 55, 66)  // len(s2) = 5, cap(s2) = 8 (a new underlying array)
s2 = s2[:0] // len(s2) = 0, cap(s2) = 8. This is used to clear slice but no GC
s2 = nil // len(s2) = cap(s2) = 0. This will be GC

s3 := s[2:3:4] // begin with s[2], length = 3-2, capacity = 4-2

for idx, v := range s {
    ... // v is a copy
}

s4 := [][]int {{1}, {1, 2}} // high dim
```

### map

In Golang, `map` is implemented on HashMap, and is not designed for concurrency (Only concurrent-read, see [stackoverflow discuss](https://stackoverflow.com/questions/36167200/how-safe-are-golang-maps-for-concurrent-read-write-operations)).

```go
m := map[string]int{"three": 3, "four": 4}

// add or modify
m["one"] = 1

// remove
delete(, "three")

// get
v, ok := m["two"]
if ok {
  // has
}

// random order
for k, v := range m {
  // notice: v is a copy
}
```

### struct

Use struct do define customer type. Struct can hold other structs, golang use combination rather than inheritance to reuse code and encapsulate objects.

Two language sugar are added: `anonymous field` and `promoted field`. (see [this answer](https://stackoverflow.com/questions/28014591/nameless-fields-in-go-structs)) Generally, if struct A has a anonymous field which is struct type, its members and functions are automatically promoted to A (writing as A.x), unless A already has a member with same name.

- type XX struct {...}
- type XX int64
- function with receiver to mimic member function

```go
type User struct {
  name string
  id uint64
}

// use this if you want u is a copy
func (u User) foo() {

}
// use this if you want u is a reference, COMMON USAGE
func (u *User) foo() {

}
```

### interface

- support polymorphism
- support duck-typing, just like `js`
- use empty `interface{}` as parameter to accept all types

You don't have to "implement" an interface explicitly in Go. Go acts like an pure dynamic language, aka duck=typing. Mechanism explained [here](https://research.swtch.com/interfaces), and [more about interface](https://sanyuesha.com/2017/07/22/how-to-understand-go-interface/).

An object of `interface` type has 2 pointers: a pointer to `iTable` of actual type, a pointer to object of actual type. `iTable` contains type info and a method collection.

```go
type iface struct {
  tab *itab
  data unsafe.Pointer
}
```

See more on

- https://halfrost.com/go_interface/
- https://zhaolion.com/post/golang/upgrade/interface/
- https://github.com/teh-cmc/go-internals/blob/master/chapter2_interfaces/README.md

### channel

- no buffer: Both sides should be ready at same time, or wait/blocked for other.
- buffered: Queue. Receiver waits if channel empty, sender waits if channel full.
- read-only channel `<-chan`, write-only channel `chan<-`
- channel can be used as a sync mechanism.

[Concurrency-in-Go](https://www.youtube.com/watch?v=LvgVSSpwND8)

```go
nobufferChannel := make(chan int) // `int` can be other types, including pointer
bufferedChannel := make(chan int, 10)

bufferedChannel <- 9 // send a int to channel

v, ok := <- bufferedChannel // receive a int from channel
if !ok {
  // channel is closed
}

for msg := range bufferedChannel {
  // auto break when channel close. "range" here is a sugar
}

// only sender should close. Sending to a closed channel will panic
close(bufferedChannel)

for { // inf loop
  select { // select randomly when any channel has content
  case msg1 := <- nobufferChannel:
    // do something
  case msg2 := <- bufferChannel:
    // do something
  }
}

// worker pool pattern
func worker(jobs <-chan int, results chan<- int) {
  for n:= range jobs {
    results <- n*n
  }
}
func main(){
  jobs := make(chan int, 10)
  results := make(chan int, 10)
  go worker(jobs, results)
  go worker(jobs, results)
  go worker(jobs, results)
  go worker(jobs, results)
  for i:= 0; i < 10; i++ {
    jobs <- i
  }
  close(jobs)
  for j:=0; j < 10; j++ {
    fmt.Println(<-results)
  }
}

```

## variable

- variable outside a function belongs to package
- start with lower-case to make it only visible within its package

### new/make

- `new` allocates memory without initialization (which means to call constructor in other lang), just zeroed.
- `new(T)` returns a pointer of T `*T`, rather than `T`.
- `&T{}` is equal to `new(T)`, and `var i int; &i` is equal to `new(int)`.
- `make` allocates memory and **initialize** it.
- `make` returns a instance of T.
- `make` only used with `slice, map, channel`

More see [EffeciveGo: allocation with new and make](https://go.dev/doc/effective_go#allocation_new)

```go
s := make([]int, 100) // Allocate memory of 100 ints, initialize to 0
m := make(map[string]int) // Allocate memory for an empty <string, int> map
c := make(chan int)
m2 := map[string]int {} // Same effect as m. Called composite literals
```

https://stackoverflow.com/questions/9320862/why-would-i-make-or-new
https://stackoverflow.com/questions/31064688/which-is-the-nicer-way-to-initialize-a-map

## control flow

- for
- if
- switch-case, `break` by default, explicitly `fallthrough` if don't want to break
- `select` to multiplex multiple `channel`
- `defer`

`defer` exec when exit current scope. Multiple defers in LIFO(stack) manner.

## function

- function `main()` in `main.go`
- function `init()`, executed before `main()`
- multiple return values, using `_` to omit
- **NO support for function overloading** (for compiler speed and [Go team's taste](https://go.dev/doc/faq#overloading))

## goroutine

- one thread can have multiple goroutines
- use `go build -race` to detect race condition
- use `go foo` to create a goroutine with foo, or `go func() {}` with no name
- use `atomic` package to ensure atomic operation
- use `sync.Mytex` to add lock
- use `channel` to share data between goroutines

### concurrency control

- use `channel` in simple scenario to detect if goroutine ends.

```go
func main() {
  num := 3
  channels := make([]chan int, num)
  for i:= 0; i<num; i++ {
    channels[i] = make(chan int)
    go DoJob(channels[i])
  }
  // wait
  for i, ch := channels {
    <-ch
    // goroutine i finishes
  }
  // all finishes
}
func DoJob(ch chan int) {
  // working
  ch <- 1 // any int number
}

```

- use `sync.WaitGroup`(implement via semaphore) to wait for all goroutines above.

```go
func main() {
  num := 3
  var wg sync.WaitGroup
  wg.Add(num)

  for i:= 0; i<num; i++ {
    go DoJob(&wg)
  }

  wg.Wait()
  // all finishes
}

func DoJob(wg *sync.WaitGroup) {
  // working
  wg.Done() // counter minus 1
}

```

- use [`Context`](https://go.dev/blog/context) to put more control on multi-level goroutine. Very common in handle web requests. See [official examples](https://pkg.go.dev/context)

> In Go servers, each incoming request is handled in its own goroutine. Request handlers often start additional goroutines to access backends such as databases and RPC services. The set of goroutines working on a request typically needs access to request-specific values such as the identity of the end user, authorization tokens, and the request’s deadline. When a request is canceled or times out, all the goroutines working on that request should exit quickly so the system can reclaim any resources they are using.

```go
// https://gobyexample.com/context

func hello(w http.ResponseWriter, req *http.Request) {

    ctx := req.Context()

    fmt.Println("server: hello handler started")
    defer fmt.Println("server: hello handler ended")

    select {
    case <-time.After(10 * time.Second):
        fmt.Fprintf(w, "hello\n") // after 10s write response
    case <-ctx.Done():
        err := ctx.Err()
        fmt.Println("server:", err)
        internalError := http.StatusInternalServerError
        http.Error(w, err.Error(), internalError)
    }
}

func main() {
    http.HandleFunc("/hello", hello)
    http.ListenAndServe(":8090", nil)
}

```

## error handling

- `panic`, `recover`. When panic, all predefined `defer` will run, then program `exit(2)`.
- `v, error := func()`

Go has no exception mechanism. You have to deal with error where it happens.

## stack/heap

> Sometimes we want an object lives on stack, in c# you can use `Span` but in Go it's impossible.

- GC handles heap memory, but way too expensive than stack memory.
    - More about [Go GC](https://go.dev/doc/gc-guide)
- Go automatically handles whether on heap or stack via compiler's `escape analysis`.
    - If a function returns a pointer to its local var, escape happens.
    - If a local var is too large(or unkonwn) to be on stack, escape happens.
    - If a param has unkonwn type (e.g. `...interface`), escape happens.
    - If a local var is in closure, escape happens.
- Ouput escapes via `go build -gcflags=-m`

## reflection

`reflect` package provides representation and operations of type info and value info in `interface`.

```go
func ValueOf(i interfae{}) reflect.Value
func TypeOf(i interfae{}) reflect.Type
```

Use reflection to
- get `reflection representation` on any object
- get original object from `reflection representation`
- modify value of original object from `reflection representation`. (Sometimes we need to pass pointer and retrieve by `Elem()`)

```go
var x float64 = 0.1
v := reflect.ValueOf(&x)
v.Elem().SetFloat(6.6)
fmt.Println(v.Elem().Interface()) //6.6
fmt.Println(x) //6.6
```

See more on 
- https://github.com/Ompluscator/dynamic-struct
- https://halfrost.com/go_reflection/

## misc tools

### print

```go
s := fmt.SPrintf("%#v", myStruct) // %#v with typeName, fieldName. %+v with fieldName. %v is plain value.

// customize ToString on Type MyStruct
func (this *MyStruct) String() string {
    return fmt.Sprintf("This is %v", this)
}

// using json serialization
resultString, err := json.Marshal(myStructObj)

```

`fmt.FPrintf`

### io

- Read/Write Json
```go
var jsonObject MyJsonObject

if bt, err := ioutil.ReadFile("xxx.json"); err == nil {
  if err2 := json.Unmarshal(bt, &jsonObject); err2 == nil {
    // use jsonObject
  }
}

if bt, err := json.Marshal(jsonObject); err == nil {
  if err2 := ioutil.WriteFile(filePath, bt, 0666); err2 != nil { // 0666 is permission mask
    // handle error
  }
}
```

- Read csv
```go
allRecords := make([]map[string]string, 0) // array of map
headerLines := 1 // num of rows that are headers, not data

if bt, err := ioutil.ReadFile("xxx.csv"); err == nil {
  reader := csv.NewReader(bt)
  if data, err2 := reader.ReadAll(); err == nil {
    // data is [][]string
    header := data[0]

    for i, line := range data {
      if i < headerLines {continue} //omit headers

      if len(line) < len(header) {continue} // omit invalid line

      // Your logic:
      record := map[string]string
      for j, field := range line {
        record[header[j]] = field
      }
      allRecords = append(allRecords, record)
    }
  }
}
```

### race detection

`go run -race xxx.go`

### go generate

Used for code-generation. [Official blog](https://go.dev/blog/generate). [Stackoverflow example](https://stackoverflow.com/questions/37362054/explain-go-generate-in-this-example)

Other code-generation framework.

- [`genny`, Generics for Go](https://github.com/cheekybits/genny)
- [`gen`, a code-generation tool for Go](https://github.com/clipperhouse/gen)

### text/template

Used for code-generation, too. [Official doc](https://pkg.go.dev/text/template). [Go by example](https://gobyexample.com/text-templates).

There’re two packages operating with templates — `text/template` and `html/template`. Both provide the same interface, however the html/template package is used to generate HTML output safe against code injection.

> Data + Template => Generated File

Besides simple text substitution, you can use `if/else/end, range, break, continue...` in template to be more flexible.

### go test

- create a go file, name ends with `_test`
- `import "testing"`
    - `import "net/http/httptest"` to mock http
- `func TestXXX(t *testing.T)` for unit test (UT), use `t.Fatalf(msg)` to give an error.
- `func BenchmarkXXX(b *testing.B)` for performance test (PT), user `b.ResetTimer()` to reset time now.

Use cli

- `go test -v` for all tests
- `go test -v -run="xxx"` for specific UT (xxx can be regex)
- `go test -v -run="none" -bench="xxx"` for specific PT (xxx can be regex)
- `go test -v -run="none" -bench="xxx" -benchmem` to see allocation

Or use GoLand

- for UT, just click green button on each test, or run all UT in a folder.
- for PT, click green button and choose "Profile with CPU/Mem..." to see flamegraph/calltree/etc.

### pprof

On windows, install [Graphviz](https://graphviz.org/download/) before open pprof result in browser.

# pitfalls & tricks

Go team explained why they make these choices on [FAQ](https://go.dev/doc/faq).

## go version

- Go1.10 does not support `gopls`
- Go1.10 has bad performance on interface{}, pprof shows much time on `convT2E32` (See [Go internals](https://github.com/teh-cmc/go-internals/blob/master/chapter2_interfaces/README.md), basically, it involves memory allocation when converting between concrete type and interface)

## variadic params

- How to pass variadic params to nested function?

> Do NOT foget `...`, or you get wrong param type.

```go
func main() {
  A(1,2,3)
}
func A(args ...interface{}) {
  fmt.Println(args) //3
  B(args) // output 1, which is wrong
  B(args...) //output 3
}
func B(args ...interface{}) {
  fmt.Println(args)
}

```

## interface

- How to check nil on interface{}?

> [stackoverflow](https://stackoverflow.com/questions/13476349/check-for-nil-and-nil-interface-in-go)
> If `var a interface{}`, `a == nil` is NOT enough to validate, because underlying value can still be nil while `a != nil`.

- How to cast interface{} to concrete type?

> The only right way is `v, ok := a.(T)` and check if `ok`. Otherwise you still get `false` when `a.(bool)` and `a is nil`.

- How to explicitly declare a struct implement an interface?

> [stackoverflow](https://stackoverflow.com/questions/31753282/go-how-to-explicitly-state-that-a-structure-is-implementing-an-interface)
> The only right way (trick) is adding `var _ IYourInterface = (*YourType)(nil)` where you want to validate.

- How to achieve polymorphism via interface?

> It's trickier than you think. The "type" info stored in an interface variable is NOT always real, it's just inferred from context. If context does NOT give enough information what the real type is, polimorphism will fail. The trick is, passing subclass type to baseclass type and stored as a property `internal`, now even you just got a baseclass pointer, you can still call subclass's Foo().

```go
package main

import "fmt"

type IObj interface {
  Create() IObj
  Foo()
  GetConcrete() *Base
}
type Base struct {
  internal IObj
}
func (this *Base) GetConcrete() *Base {
    return this
}
func (this *Base) Create() IObj {
    this.CreateInternal(nil)
    return this
}
func (this *Base) CreateInternal(self IObj) {
  this.internal = self
}
func (this *Base) Foo() {
  if this.internal != nil {
  	this.internal.Foo()
  }
}

type Sub struct {
  Base
}
func (this *Sub) Create() IObj {
    this.CreateInternal(this)
    return this
}
func (this *Sub) Foo() {
  fmt.Println("Sub.Foo")
}

func main() {
  // sometimes we just got *Base, not IObj
  var i *Base
  i = new(Sub).Create().GetConcrete()
  i.Foo() // Sub.Foo
}

```

## protobuf

- [Unmarshal into incorrect type does NOT result in error](https://github.com/protocolbuffers/protobuf/issues/7946). A bytes can be forcely translated into different type with unexpected value, but `err := protobuf.Unmarshal(bytes, msg)` is no error. Official says it's expected behaviour.