# Golang

![](E:\Github\tech-crashcourse\res\Golang.png)


### entry
- function `main()` in `main.go`
- function `init()`, executed before `main()`

### package
- Go builds `package main` into executables.
- Go searches local packages in `$GOPATH`
- Install remote packages via `go get`
- `import "github.com/xxx/xxx"`
- `import _ "packageName"` only call init()
- `import . "packageName"` omit
- `import alias "packageName"` give a new name

### type
- Value type: `built-in types, array, struct`
- Reference type: `slice, map, channel, interface, function`
- reflect.TypeOf(x)
- type embedding (act like inheritance)
- type assertion `cat, ok := animal.(Cat)`, or `swtich t:= animal.(type)`


#### array
- Pass by reference to avoid copy whole array
##### code
```go
var arr [5]int
var arr2 := [...]{1,2,3,4,5}
arr = arr2 // value copy
var arr2D := [2][2]int {{1,2}, {3,4}}
```
#### slice
- Size = 3 pointers. a pointer to underlying array, a length, a capacity.
- Can be `nil`
- Suggest length == capacity

When capacity is not enough, Go doubles the capacity and create a new underlying array for this slice. This can cause **unintentional** effect.

##### code
```go
var s []int // a nil slice
var ps *[]int = new([]int) // points to a nil slice. uncommon

s := []int{1, 2, 3} // len(s) == 3, cap(s) == 3
s = append(s, 4) // len(s) == 4, cap(s) == 6

s2 := s[2:4] // len(s2) == 2, cap(s2) == 4 (cap(s)-2)
s2 = append(s2, 44, 55, 66)  // len(s2) == 5, cap(s2) == 8 (a new underlying array)

s3 := s[2:3:4] // begin with s[2], length = 3-2, capacity = 4-2

for idx, v := range s {
	... // v is a copy
}

s4 := [][]int {{1}, {1, 2}} // high dim
```

#### map
##### code
```go
m := map[string]int{"three": 3, "four": 4}
m["one"] = 1
delete(, "three")

v, ok := m["two"]
if ok {
  // has key
}

for k, v := range m {
  // v is a copy
}
```
#### struct
Use struct do define customer type
- type XX struct {...}
- type XX int64
- function with receiver to mimic member function

##### code
```go
type User struct {
  name string
  id uint64
}

// use this if you want u is a copy
func (u User) foo() {

}
// use this if you want u is a reference
func (u *User) foo() {

}

u := &User{"jojo", uint64(999)}
fmt.Println(u) // &{jojo 999}
u.foo()
```

#### interface
- support polymorphism
- support duck-typing, just like `js`
- use empty `interface{}` as parameter to accept all types

You don't have to "implement" an interface explicitly in Go. Go acts like an pure dynamic language, aka duck=typing. Mechanism explained [here](https://research.swtch.com/interfaces), and [more about interface](https://sanyuesha.com/2017/07/22/how-to-understand-go-interface/).


An object of `interface` type has 2 pointers: a pointer to `iTable` of actual type, a pointer to object of actual type.

`iTable` contains type info and a method collection.



#### channel
- no buffer: Both sides should be ready at same time, or wait/blocked for other.
- buffered: Queue. Receiver waits if channel empty, sender waits if channel full.

##### code
```go
nobuffer := make(chan int)
buffered := make(chan int, 10)

buffered <- 9 // send a int to channel

v, ok := <- buffered // receive a int from channel
if !ok {
    // channel is empty and closed
}

close(buffered)
```

### new/make
- `new` allocates memory without initialization (call constructor in other lang), just zeroed.
- `new` returns a `pointer` of T, rather than T.
- `&T{}`== `new(T)`, `var i int; &i` == `new(int)`.
- `make` allocates memory and initialize it.
- `make` returns a instance of T.
- `make` only used with `slice, map, channel`

#### code
```go
s := make([]int, 100) // Allocate memory of 100 ints, initialize to 0
m := make(map[string]int) // Allocate memory for an empty <string, int> map
c := make(chan int)
m2 := map[string]int {} // Same effect as m. Called composite literals
```

https://stackoverflow.com/questions/9320862/why-would-i-make-or-new
https://stackoverflow.com/questions/31064688/which-is-the-nicer-way-to-initialize-a-map

### stack/heap
- Go automatically handles whether on heap or stack.
- Compile-time `escape analysis`. (escape to heap)
- GC handles those on heap.

### control flow
- for
- if
- switch, `fallthrough`
- `select` to monitor multiple `channel` state
- `defer`

`defer` exec when exit current scope. Multiple defers in LIFO(stack) manner.

Go use `break` automatically in `switch`, otherwise you need `fallthrough`.

### goroutine
- one thread, multiple goroutines
- use `go build -race` to detect race condition
- use `go foo` to create a goroutine with foo, or `go func() {}` with no name
- use `sync.WaitGroup` to wait for all goroutines above
- use `atomic` package to ensure atomic operation
- use `sync.Mytex` to add lock
- use `channel` to share data between goroutines

### error handling
- `panic`, `recover`
- `v, error := func()`

Go has no exception mechanism. You have to deal with error where it happens.

### misc


#### log tools

#### test tools
- file name ends with `_test`
- `import "testing"`
- `import "net/http/httptest"` to mock http
- `func TestXXX(t *testing.T)` for unit test (UT)
- `func BenchmarkXXX(b *testing.B)` for performance test (PT)
- `go test -v` for all tests
- `go test -v -run="xxx"` for specific UT (xxx can be regex)
- `go test -v -run="none" -bench="xxx"` for specific PT (xxx can be regex)
- `go test -v -run="none" -bench="xxx" -benchmem` to see allocation

#### doc tools
- go doc [packageName]
#### format tool
- gofmt