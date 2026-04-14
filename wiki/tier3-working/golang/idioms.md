# Go Idioms and Patterns

> **Tier 3** | Source: Effective Go, Go blog | Enforces/Derives From: wiki/tier1-sources/swebok-v4/ka04-construction.md

## Summary

Idiomatic Go emphasizes simplicity, explicit error handling, and composition over inheritance. This page documents the patterns that distinguish professional Go from ad hoc code.

## Error Handling — The Central Go Idiom

Error handling is not optional in Go. Every fallible function returns an `error` as its last return value. Ignoring it with `_` is a serious bug.

### Basic pattern

```go
result, err := doSomething()
if err != nil {
    return nil, fmt.Errorf("doSomething failed: %w", err)
}
```

### Error wrapping with `%w`

Use `fmt.Errorf("context: %w", err)` to add context while preserving the original error for `errors.Is()` and `errors.As()`.

```go
func loadConfig(path string) (*Config, error) {
    data, err := os.ReadFile(path)
    if err != nil {
        return nil, fmt.Errorf("loadConfig: reading %q: %w", path, err)
    }
    // ...
}

// Callers can still check the root cause
if errors.Is(err, os.ErrNotExist) {
    // file was not found
}
```

### Sentinel errors

```go
var (
    ErrNotFound   = errors.New("not found")
    ErrUnauthorized = errors.New("unauthorized")
)

func GetUser(id int) (*User, error) {
    if id == 0 {
        return nil, ErrNotFound
    }
    // ...
}

// Check with errors.Is — works through wrapping chains
if errors.Is(err, ErrNotFound) {
    // handle not found
}
```

### Custom error types

```go
type ValidationError struct {
    Field   string
    Message string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation error on field %q: %s", e.Field, e.Message)
}

// Check with errors.As
var ve *ValidationError
if errors.As(err, &ve) {
    fmt.Printf("Invalid field: %s\n", ve.Field)
}
```

### Handle errors at the appropriate level

Pick one: log OR return. Don't do both — it leads to duplicate log entries and confusion about where the error originated.

```go
// BAD: logs and returns — causes duplicate log output up the call chain
func processUser(id int) error {
    user, err := getUser(id)
    if err != nil {
        log.Printf("getUser failed: %v", err)  // logged here
        return fmt.Errorf("processUser: %w", err)  // and propagated
    }
    // ...
}

// GOOD: either log (terminal handler) or return (intermediate handler)
func processUser(id int) error {
    user, err := getUser(id)
    if err != nil {
        return fmt.Errorf("processUser: %w", err)  // wrap and return
    }
    // ...
}
```

## Interfaces — Implicit and Small

Go interfaces are satisfied implicitly. A type need not declare that it implements an interface.

```go
// Small, focused interfaces — compose for more capability
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

type ReadWriter interface {
    Reader
    Writer
}
```

**Accept interfaces, return structs**: functions that accept interfaces are flexible; functions that return interfaces hide type information unnecessarily.

```go
// Good: accept the interface, return the concrete type
func NewFileLogger(w io.Writer) *FileLogger {
    return &FileLogger{w: w}
}

// Bad: returning an interface prevents callers from accessing concrete methods
func NewFileLogger(w io.Writer) io.Writer {
    return &FileLogger{w: w}
}
```

**Interface naming**: single-method interfaces use the method name + `-er` suffix (`io.Reader`, `io.Writer`, `fmt.Stringer`).

## Named Return Values

Use only for documentation in short functions. Never use naked returns in functions longer than 3 lines — they make control flow invisible.

```go
// OK: short function, named returns as documentation
func divide(a, b float64) (result float64, err error) {
    if b == 0 {
        err = errors.New("division by zero")
        return  // naked return — acceptable here (2 lines)
    }
    result = a / b
    return
}

// BAD: naked return in long function — unreadable
func processOrder(id int) (order *Order, invoiceID int, err error) {
    // ... 30 lines of code ...
    return  // what are the values here?
}
```

## defer — Resource Cleanup

Always defer cleanup immediately after acquisition. Multiple defers execute in LIFO order (last deferred, first executed).

```go
func writeFile(path string, data []byte) error {
    f, err := os.Create(path)
    if err != nil {
        return fmt.Errorf("creating file: %w", err)
    }
    defer f.Close()   // immediately after successful open

    _, err = f.Write(data)
    return err
}

// Multiple defers — LIFO
func example() {
    defer fmt.Println("third")   // prints last
    defer fmt.Println("second")  // prints second
    defer fmt.Println("first")   // prints first
}
```

## Zero Values — Design for Validity

Design types so their zero value is immediately usable.

```go
// sync.Mutex zero value is an unlocked mutex — ready to use
var mu sync.Mutex
mu.Lock()
defer mu.Unlock()

// bytes.Buffer zero value is an empty buffer — ready to use
var buf bytes.Buffer
buf.WriteString("hello")
```

When designing your own types, strive for a meaningful zero value. If it is impossible, document the required initialization clearly.

## Struct Embedding — Composition

Use embedding to reuse behavior. Embedding is not inheritance — the outer type does not become the inner type.

```go
type Logger struct {
    prefix string
}

func (l *Logger) Log(msg string) {
    fmt.Printf("[%s] %s\n", l.prefix, msg)
}

type Service struct {
    Logger           // embedded — Service.Log() is promoted
    endpoint string
}

svc := Service{Logger: Logger{prefix: "svc"}, endpoint: "/api"}
svc.Log("started")  // calls Logger.Log via promotion
```

## Pointer vs Value Receivers — Be Consistent

- Use **pointer receivers** when: the method mutates the struct, the struct is large (copying is expensive), or consistency with other methods on the type requires it.
- Use **value receivers** when: the struct is small, immutable, and copying is trivially cheap.
- **Be consistent within a type**: if any method uses pointer receiver, all methods should.

```go
type Counter struct{ count int }

func (c *Counter) Increment() { c.count++ }  // pointer — mutates
func (c *Counter) Value() int { return c.count }  // pointer — consistent with Increment

type Point struct{ X, Y float64 }

func (p Point) Distance() float64 {  // value — small, doesn't mutate
    return math.Sqrt(p.X*p.X + p.Y*p.Y)
}
```

## Naming Conventions

| Item | Convention | Example |
|------|-----------|---------|
| Exported names | PascalCase | `UserService`, `ErrNotFound` |
| Unexported names | camelCase | `parseHeader`, `maxRetries` |
| Interfaces (single method) | Method name + `-er` | `Reader`, `Stringer`, `Closer` |
| Package names | lowercase, single word | `http`, `encoding`, `myservice` |
| Loop index | `i`, `j` | short is fine for tight loops |
| Error variable | `err` | always `err`, not `e` or `error` |
| Short-lived local | `r` for reader, `w` for writer | conventional in stdlib |

## init Functions

Avoid `init()` where possible. It runs implicitly, is hard to test, and can have surprising ordering. Use explicit initialization functions instead.

```go
// BAD: implicit init
func init() {
    loadConfig()  // when does this run? hard to control
}

// GOOD: explicit initialization
func NewApp(configPath string) (*App, error) {
    cfg, err := loadConfig(configPath)
    if err != nil {
        return nil, err
    }
    return &App{cfg: cfg}, nil
}
```

## Anti-Patterns

### Naked panic

```go
// BAD: panic should not cross API boundaries
func GetUser(id int) *User {
    user, err := db.Find(id)
    if err != nil {
        panic(err)  // crashes the server on DB error
    }
    return user
}

// GOOD: return error
func GetUser(id int) (*User, error) {
    return db.Find(id)
}
```

### Goroutine leaks

Every goroutine must have a defined exit condition. A goroutine that blocks forever without cleanup leaks memory and goroutines.

```go
// BAD: goroutine leaks if nobody reads from ch
func leaky() chan int {
    ch := make(chan int)
    go func() {
        ch <- expensiveComputation()  // blocks forever if caller ignores ch
    }()
    return ch
}

// GOOD: use context for cancellation
func safe(ctx context.Context) chan int {
    ch := make(chan int, 1)
    go func() {
        select {
        case ch <- expensiveComputation():
        case <-ctx.Done():
        }
    }()
    return ch
}
```

### `interface{}` overuse

```go
// BAD: loses all type safety
func Process(data interface{}) interface{} { ... }

// GOOD: use generics (Go 1.18+) or specific types
func Process[T any](data T) T { ... }
```

## See Also

- wiki/tier3-working/golang/concurrency.md
- wiki/tier3-working/golang/toolchain.md
- wiki/tier1-sources/swebok-v4/ka04-construction.md

## Source

Effective Go (golang.org/doc/effective_go). "The Go Programming Language" (Donovan & Kernighan, 2015). Go blog: Error handling and Go (go.dev/blog). CodeReviewComments (github.com/golang/go/wiki/CodeReviewComments).
