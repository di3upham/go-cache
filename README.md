# go-cache

go-cache is an in-memory key:value store/cache similar to memcached that is
suitable for applications running on a single machine. Its major advantage is
that, being essentially a thread-safe `map[string]interface{}` with expiration
times, it doesn't need to serialize or transmit its contents over the network.

Any object can be stored, for a given duration or forever, and the cache can be
safely used by multiple goroutines.

Although go-cache isn't meant to be used as a persistent datastore, the entire
cache can be saved to and loaded from a file (using `c.Items()` to retrieve the
items map to serialize, and `NewFrom()` to create a cache from a deserialized
one) to recover from downtime quickly. (See the docs for `NewFrom()` for caveats.)

### Installation

`go get github.com/di3upham/go-cache`

### Usage

```go
import (
	"fmt"
	"github.com/di3upham/go-cache"
	"time"
)

func main() {
	// Create a cache with a default expiration time of 5 minutes
	c := cache.New(5*time.Minute)

	// Set the value of the key "foo" to "bar", with the default expiration time
	c.Set("foo", "bar")

	// Set the value of the key "baz" to 42, with the expiration time
	// is 1 minute. (the item won't be removed until it is expired, re-set, or removed using
	// c.Delete("baz")
	c.SetWithExpire("baz", 42, 1 * time.Minute)

	// Get the string associated with the key "foo" from the cache
	foo, found := c.Get("foo")
	if found {
		fmt.Println(foo)
	}

	// Since Go is statically typed, and cache values can be anything, type
	// assertion is needed when values are being passed to functions that don't
	// take arbitrary types, (i.e. interface{}). The simplest way to do this for
	// values which will only be used once--e.g. for passing to another
	// function--is:
	foo, found := c.Get("foo")
	if found {
		MyFunction(foo.(string))
	}

	// This gets tedious if the value is used several times in the same function.
	// You might do either of the following instead:
	if x, found := c.Get("foo"); found {
		foo := x.(string)
		// ...
	}
	// or
	var foo string
	if x, found := c.Get("foo"); found {
		foo = x.(string)
	}
	// ...
	// foo can then be passed around freely as a string

	// Want performance? Store pointers!
	c.Set("foo", &MyStruct, cache.DefaultExpiration)
	if x, found := c.Get("foo"); found {
		foo := x.(*MyStruct)
			// ...
	}

	// Set multiple items
	entries := make(map[string]interface{})
	entries["k1"] = "v1"
	entries["k2"] = "v2"
	entries["k3"] = "v3"
	c.SetMultiple(entries)

	itemm := c.GetMultiple([]string{"k1", "k3"})
	for k, x := range itemm {
		foo := x.(string)
		// ...
	}
}
```

### Reference

`godoc` or [https://godoc.org/github.com/di3upham/go-cache](https://godoc.org/github.com/di3upham/go-cache)
