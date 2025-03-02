+++
title = "Golang: Block forever"
date = 2020-04-27T15:59:21+07:00
lastmod = 2020-04-27T15:59:21+07:00
tags = ["golang", "trick", "note"]
categories = []
imgs = []
cover = ""  # image show on top
readingTime = true  # show reading time after article date
toc = true
comments = true
justify = false  # text-align: justify;
single = false  # display as a single page, hide navigation on bottom, like as about page.
license = ""  # CC License
draft = false
+++

Sometimes, you want to block the current goroutine when allowing others to continue. Here is some tricks I've collected:

## 1. References

Firstly give them some credits:

1. https://blog.sgmansfield.com/2016/06/how-to-block-forever-in-go/
2. https://pliutau.com/different-ways-to-block-go-runtime-forever/

> NOTE: I run these with Golang 1.12

## 2. The original

```go
package main

import "fmt"

func show() {
    for i := 1; i < 9696969; i++ {
        time.Sleep(1000)
        fmt.Println(i)
    }
}

func main() {
    go show() // The main goroutine is exited before the show() be done.
    fmt.Println("OK we're done")
}
```

## 3. Bad - An empty infinite loop

```go
package main

import (
    "fmt"
    "time"
)

func forever() {
    for {
        // Empty, just do nothing
    }
}

func show() {
    for i := 1; i < 9696969; i++ {
        time.Sleep(5 * time.Second)
        fmt.Println(i)
    }
}

func main() {
    go show()
    forever()
    fmt.Println("OK we're done")
}
```

An infinite loop here is a busy loop that does nothing except burn CPU time.

## 4. Good - Busy blocking

```go
package main

import (
    "fmt"
    "runtime"
    "time"
)

func forever() {
    for {
        runtime.Gosched()
    }
}

func show() {
    for i := 1; i < 9696969; i++ {
        time.Sleep(5 * time.Second)
        fmt.Println(i)
    }
}

func main() {
    go show()
    forever()
    fmt.Println("OK we're done")
}
```

It will reduce your CPU usage but it isn't the preferable solution.

## 5. Good - Waiting on itself

We wait but we never done XD

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func forever() {
    wg := sync.WaitGroup{}
    wg.Add(1)
    wg.Wait()
}

func show() {
    for i := 1; i < 9696969; i++ {
        time.Sleep(5 * time.Second)
        fmt.Println(i)
    }
}

func main() {
    go show()
    forever()
    fmt.Println("OK we're done")
}
```

## 6. Good - Empty select

```go
package main

import (
    "fmt"
    "time"
)

func forever() {
    select{ }
}

func show() {
    for i := 1; i < 9696969; i++ {
        time.Sleep(5 * time.Second)
        fmt.Println(i)
    }
}

func main() {
    go show()
    forever()
    fmt.Println("OK we're done")
}
```

## 7. Good - Double locking

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func forever() {
    m := sync.Mutex{} // Same with sync.RWMutex
    m.Lock()
    m.Lock()
}

func show() {
    for i := 1; i < 9696969; i++ {
        time.Sleep(5 * time.Second)
        fmt.Println(i)
    }
}

func main() {
    go show()
    forever()
    fmt.Println("OK we're done")
}
```

## 8. Good - Reading an Empty Channel

```go
package main

import (
    "fmt"
    "time"
)

func forever() {
    c := make(chan struct{})
    <-c
}

func show() {
    for i := 1; i < 9696969; i++ {
        time.Sleep(5 * time.Second)
        fmt.Println(i)
    }
}

func main() {
    go show()
    forever()
    fmt.Println("OK we're done")
}
```

## 9. Good - Self produce-and-consume

```go
package main

import (
    "fmt"
    "time"
)

func forever() {
    c := make(chan struct{}, 1)
    for {
        select {
        case <-c:
        case c <- struct{}{}:
        }
    }
}

func show() {
    for i := 1; i < 9696969; i++ {
        time.Sleep(5 * time.Second)
        fmt.Println(i)
    }
}

func main() {
    go show()
    forever()
    fmt.Println("OK we're done")
}
```
