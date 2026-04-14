# Go Concurrency Patterns

> **Tier 3** | Source: Effective Go, "The Go Programming Language" | Enforces/Derives From: wiki/tier1-sources/swebok-v4/ka04-construction.md

## Summary

Go's concurrency model is built on goroutines (lightweight threads) and channels (typed communication pipes). The scheduler multiplexes goroutines onto OS threads transparently. This page covers safe concurrency patterns and the tools to detect race conditions.

## Goroutines

Goroutines are cheap — approximately 2KB initial stack vs 1MB for an OS thread. The runtime grows stacks as needed.

```go
// Start a goroutine
go func() {
    fmt.Println("running concurrently")
}()

// Always track goroutines — ensure they can exit
go func() {
    for {
        select {
        case work := <-workCh:
            process(work)
        case <-ctx.Done():
            return  // clean exit on cancellation
        }
    }
}()
```

## Channels

Channels are typed communication pipes between goroutines.

```go
// Unbuffered channel — sender blocks until receiver is ready
ch := make(chan int)

// Buffered channel — sender blocks only when buffer is full
ch := make(chan int, 10)

// Send
ch <- 42

// Receive
value := <-ch

// Close — signals no more values will be sent
close(ch)

// Range over channel — exits when channel is closed
for value := range ch {
    process(value)
}

// Directional channels — enforce send/receive roles
func producer(out chan<- int) { out <- 1 }    // send-only
func consumer(in <-chan int) { v := <-in; _ = v }   // receive-only
```

## select — Wait on Multiple Channels

```go
select {
case v := <-dataCh:
    process(v)
case err := <-errCh:
    log.Printf("error: %v", err)
case <-ctx.Done():
    return ctx.Err()
case <-time.After(5 * time.Second):
    return errors.New("timed out")
}

// Non-blocking select with default
select {
case v := <-ch:
    use(v)
default:
    // channel was empty — continue without blocking
}
```

## sync.WaitGroup — Wait for a Group of Goroutines

```go
import "sync"

var wg sync.WaitGroup

for _, item := range items {
    wg.Add(1)
    go func(item string) {
        defer wg.Done()
        process(item)
    }(item)  // pass item as argument to avoid closure capture bug
}

wg.Wait()  // blocks until all goroutines call Done()
```

## sync.Mutex / sync.RWMutex — Protect Shared State

Embed mutexes in the struct they protect. Always defer `Unlock`.

```go
type SafeCounter struct {
    mu    sync.Mutex
    count map[string]int
}

func (c *SafeCounter) Increment(key string) {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.count[key]++
}

// RWMutex — multiple concurrent readers, exclusive writer
type Cache struct {
    mu   sync.RWMutex
    data map[string]string
}

func (c *Cache) Get(key string) (string, bool) {
    c.mu.RLock()
    defer c.mu.RUnlock()
    v, ok := c.data[key]
    return v, ok
}

func (c *Cache) Set(key, value string) {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.data[key] = value
}
```

## Context — Cancellation and Deadlines

Pass `context.Context` as the first argument to every function that performs I/O or long-running work. Always respect `ctx.Done()`.

```go
func fetchUser(ctx context.Context, id int) (*User, error) {
    // Pass context to all downstream calls
    row := db.QueryRowContext(ctx, "SELECT * FROM users WHERE id = $1", id)
    // ...
}

// Create contexts with deadlines
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()  // always cancel to release resources

user, err := fetchUser(ctx, 42)

// Propagate cancellation
ctx, cancel := context.WithCancel(parentCtx)
go worker(ctx)
// ... later, when done:
cancel()  // signals worker to stop
```

## Common Concurrency Patterns

### Fan-out: Distribute Work to N Workers

```go
func fanOut(ctx context.Context, in <-chan Job, workers int) []<-chan Result {
    outputs := make([]<-chan Result, workers)
    for i := 0; i < workers; i++ {
        out := make(chan Result)
        outputs[i] = out
        go func() {
            defer close(out)
            for job := range in {
                select {
                case out <- process(job):
                case <-ctx.Done():
                    return
                }
            }
        }()
    }
    return outputs
}
```

### Fan-in: Merge N Channels into One

```go
func fanIn(ctx context.Context, channels ...<-chan Result) <-chan Result {
    merged := make(chan Result)
    var wg sync.WaitGroup
    for _, ch := range channels {
        wg.Add(1)
        go func(c <-chan Result) {
            defer wg.Done()
            for v := range c {
                select {
                case merged <- v:
                case <-ctx.Done():
                    return
                }
            }
        }(ch)
    }
    go func() {
        wg.Wait()
        close(merged)
    }()
    return merged
}
```

### Worker Pool — Fixed Pool Consuming a Work Channel

```go
func workerPool(ctx context.Context, jobs <-chan Job, numWorkers int) <-chan Result {
    results := make(chan Result)
    var wg sync.WaitGroup

    for i := 0; i < numWorkers; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for job := range jobs {
                select {
                case <-ctx.Done():
                    return
                default:
                    results <- process(job)
                }
            }
        }()
    }

    go func() {
        wg.Wait()
        close(results)
    }()
    return results
}
```

### Pipeline — Chain of Stages

```go
func generate(nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for _, n := range nums {
            out <- n
        }
    }()
    return out
}

func square(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            out <- n * n
        }
    }()
    return out
}

// Usage: pipeline
c := generate(2, 3, 4)
out := square(c)
for result := range out {
    fmt.Println(result)
}
```

## Goroutine Leaks

A goroutine that blocks permanently without being cleaned up is a leak. Common causes:

- Sending to an unbuffered channel with no receiver
- Blocking on a channel receive with no sender and no done signal
- Goroutine waits on a lock that is never released

Always ensure every goroutine has a defined exit path via `ctx.Done()`, channel close, or a done signal.

## Race Conditions

The Go race detector catches concurrent reads and writes to shared memory:

```bash
go test -race ./...
go run -race main.go
```

Rules to avoid races:
- Never share memory without a mutex or channel.
- Use `sync/atomic` only for simple integer counters.
- Run `go test -race` in CI — always.

## errgroup — Goroutine Groups with Error Collection

```go
import "golang.org/x/sync/errgroup"

g, ctx := errgroup.WithContext(context.Background())

for _, url := range urls {
    url := url  // capture loop variable
    g.Go(func() error {
        return fetch(ctx, url)
    })
}

if err := g.Wait(); err != nil {
    log.Printf("one or more fetches failed: %v", err)
}
```

`errgroup` is the Go equivalent of Python's `asyncio.TaskGroup`: first error cancels the context; `Wait()` returns the first non-nil error.

## See Also

- wiki/tier3-working/golang/idioms.md
- wiki/tier3-working/golang/toolchain.md
- wiki/tier3-working/python/async-patterns.md
- wiki/tier2-core/distributed-systems/resilience-patterns.md

## Source

Effective Go (golang.org). "The Go Programming Language" (Donovan & Kernighan, 2015). Go blog: "Pipelines and cancellation" (blog.golang.org). golang.org/x/sync/errgroup.
