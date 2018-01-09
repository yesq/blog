# `sync.Mutex` of `Golang`

Learn usage of `sync.Mutex` from [A Tour of Go](https://tour.golang.org/concurrency/9).

```golang
package main

import (
	"fmt"
	"sync"
	"time"
)

// SafeCounter is safe to use concurrently.
type SafeCounter struct {
	v   map[string]int
	mux sync.Mutex
}

// Inc increments the counter for the given key.
func (c *SafeCounter) Inc(key string) {
	c.mux.Lock()
	// Lock so only one goroutine at a time can access the map c.v.
	c.v[key]++
	c.mux.Unlock()
}

// Value returns the current value of the counter for the given key.
func (c *SafeCounter) Value(key string) int {
	c.mux.Lock()
	// Lock so only one goroutine at a time can access the map c.v.
	defer c.mux.Unlock()
	return c.v[key]
}

func main() {
	c := SafeCounter{v: make(map[string]int)}
	for i := 0; i < 1000; i++ {
		go c.Inc("somekey")
	}

	time.Sleep(time.Second)
	fmt.Println(c.Value("somekey"))
}
```

Run script above, we will get a number, `1000`. All is well.

If we comment `c.mux.Lock()` and `c.mux.Unlock()`(there are 4 lines above), we will get error `fatal error: concurrent map writes`.

If the exist struct don't have element `sync.Mutex`, could I put `sync.Mutex` in a global varite.

```golang
package main

import (
	"fmt"
	"sync"
	"time"
)

type SafeCounter struct {
	v map[string]int
}

var lock sync.Mutex

func (c *SafeCounter) Inc(key string) {
	lock.Lock()
	defer lock.Unlock()
	c.v[key]++
}

func (c *SafeCounter) Value(key string) int {
	lock.Lock()
	defer lock.Unlock()
	return c.v[key]
}

func main() {
	c := SafeCounter{v: make(map[string]int)}
	for i := 0; i < 1000; i++ {
		go c.Inc("somekey")
	}

	time.Sleep(time.Second)
	fmt.Println(c.Value("somekey"))
}
```

Everything is fine.