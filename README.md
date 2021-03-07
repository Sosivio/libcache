[![PkgGoDev](https://pkg.go.dev/badge/github.com/Sosivio/libcache@v1.0.0)](https://pkg.go.dev/github.com/Sosivio/libcache@v1.0.0)
[![Go Report Card](https://goreportcard.com/badge/github.com/Sosivio/libcache)](https://goreportcard.com/report/github.com/Sosivio/libcache)
[![Coverage Status](https://coveralls.io/repos/github/Sosivio/libcache/badge.svg?branch=master)](https://coveralls.io/github/Sosivio/libcache?branch=master)
[![CircleCI](https://circleci.com/gh/Sosivio/libcache/tree/master.svg?style=svg)](https://circleci.com/gh/Sosivio/libcache/tree/master)

# Libcache
A Lightweight in-memory key:value cache library for Go. 

## Introduction 
Caches are tremendously useful in a wide variety of use cases.<br>
you should consider using caches when a value is expensive to compute or retrieve,<br>
and you will need its value on a certain input more than once.<br>
libcache is here to help with that.

Libcache are local to a single run of your application.<br>
They do not store data in files, or on outside servers.

Libcache previously an [go-guardian](https://github.com/Sosivio/go-guardian) package and designed to be a companion with it.<br>
While both can operate completely independently.<br>


## Features
- Rich [caching API](https://pkg.go.dev/github.com/Sosivio/libcache@v1.0.0#Cache)
- Maximum cache size enforcement
- Default cache TTL (time-to-live) as well as custom TTLs per cache entry
- Thread safe as well as non-thread safe
- Event-Driven callbacks ([OnExpired](https://pkg.go.dev/github.com/Sosivio/libcache@v1.0.0#Cache),[OnEvicted](https://pkg.go.dev/github.com/Sosivio/libcache@v1.0.0#Cache))
- Dynamic cache creation
- Multiple cache replacement policies:
  - FIFO (First In, First Out)
  - LIFO (Last In, First Out)
  - LRU (Least Recently Used)
  - MRU (Most Recently Used)
  - LFU (Least Frequently Used)
  - ARC (Adaptive Replacement Cache)

## Quickstart 
### Installing 
Using libcache is easy. First, use go get to install the latest version of the library.

```sh
go get github.com/Sosivio/libcache
```
Next, include libcache in your application:
```go
import (
    _ "github.com/Sosivio/libcache/<desired-replacement-policy>"
    "github.com/Sosivio/libcache"
)
```

### Examples
**Note:** All examples use the LRU cache replacement policy for simplicity, any other cache replacement policy can be applied to them.
#### Basic 
```go
package main 
import (
    "fmt" 

    "github.com/Sosivio/libcache"
    _ "github.com/Sosivio/libcache/lru"
)

func main() {
    size := 10
    cache := libcache.LRU.NewUnsafe(size)
    for i:= 0 ; i < 10 ; i++ {
        cache.Store(i, i)
    }
    fmt.Println(cache.Load(0)) // nil, false  
    fmt.Println(cache.Load(1)) // 1, true
}
```

#### Thread Safe 
```go
package main

import (
	"fmt"

	"github.com/Sosivio/libcache"
	_ "github.com/Sosivio/libcache/lru"
)

func main() {
	done := make(chan struct{})

	f := func(c libcache.Cache) {
		for !c.Contains(5) {
		}
		fmt.Println(c.Load(5)) // 5, true
		done <- struct{}{}
	}

	size := 10
	cache := libcache.LRU.New(size)
	go f(cache)

	for i := 0; i < 10; i++ {
		cache.Store(i, i)
	}

	<-done
}
```
#### Unlimited Size
zero capacity means cache has no limit and replacement policy turned off.
```go
package main 
import (
    "fmt" 

    "github.com/Sosivio/libcache"
    _ "github.com/Sosivio/libcache/lru"
)

func main() {
	cache := libcache.LRU.New(0)
    for i:= 0 ; i < 100000 ; i++ {
        cache.Store(i, i)
    }
	fmt.Println(cache.Load(55555))
}
```
#### TTL
```go
package main 
import (
	"fmt"
	"time"

	"github.com/Sosivio/libcache"
	_ "github.com/Sosivio/libcache/lru"
)

func main() {
	cache := libcache.LRU.New(10)
	cache.SetTTL(time.Second) // default TTL 
	
	for i:= 0 ; i < 10 ; i++ {
        cache.Store(i, i)
	}
	fmt.Println(cache.Expiry(1))

	cache.StoreWithTTL("mykey", "value", time.Hour) // TTL per cache entry 
	fmt.Println(cache.Expiry("mykey"))

}
```

#### Events 
Timed expiration by default is performed with lazy maintenance during reads operations,<br>
Evict an expired entry immediately can be done using on expired callback.<br>
You may also specify a eviction listener for your cache to perform some operation when an entry is evicted.<br>
**Note:** Expiration events relying on golang runtime timers heap to call on expired callback when an entry TTL elapsed.
```go
package main 
import (
	"fmt"
	"time"

	"github.com/Sosivio/libcache"
	_ "github.com/Sosivio/libcache/lru"
)

func main() {
	cache := libcache.LRU.New(10)
	cache.RegisterOnEvicted(func(key, value interface{}) {
		fmt.Printf("Cache Key %v Evicted\n", key)
	})

	cache.RegisterOnExpired(func(key, value interface{}) {
		fmt.Printf("Cache Key %v Expired, Removing it from cache\n", key)
		// use delete directly when your application 
		// guarantee no other goroutine can store items with the same key.
		// Peek also invoke lazy expiry. 
		// 
		// Note this should done only with safe cache.
		cache.Peek(key) 
	})	

	for i:= 0 ; i < 10 ; i++ {
        cache.StoreWithTTL(i, i, time.Microsecond)
	}

	time.Sleep(time.Second)
	fmt.Println(cache.Len())
}
```

# Contributing
1. Fork it
2. Download your fork to your PC (`git clone https://github.com/your_username/libcache && cd libcache`)
3. Create your feature branch (`git checkout -b my-new-feature`)
4. Make changes and add them (`git add .`)
5. Commit your changes (`git commit -m 'Add some feature'`)
6. Push to the branch (`git push origin my-new-feature`)
7. Create new pull request

# License
Libcache is released under the MIT license. See [LICENSE](https://github.com/Sosivio/libcache/blob/master/LICENSE)