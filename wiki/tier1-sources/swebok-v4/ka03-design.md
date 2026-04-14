# SWEBOK V4 — KA 03: Software Design

> **Tier 1** | Source: IEEE SWEBOK V4, KA 03; SOLID principles; Design Patterns | Authority: immutable

## Summary

Software Design is the Knowledge Area concerned with transforming requirements into a representation of the software system that can guide Construction (KA 04). Design operates at two levels: architectural design (the subject of KA 02) defines the system-level structure of components and connectors, while detailed design (the subject of this KA) defines the internal structure of individual modules — their classes, interfaces, algorithms, and data structures.

For a coding agent, Design is the bridge between "what the system must do" (requirements) and "how to build it" (construction). Good detailed design produces code that is testable, extensible, and maintainable. The primary measurable goals of detailed design are low coupling (components depend on as few others as possible) and high cohesion (everything within a component is strongly related). SWEBOK V4 integrates Agile incremental design — "just enough design upfront, emerge the rest through refactoring" — and explicitly endorses SOLID principles as the canonical detailed design framework.

## Key Concepts

### Design Levels

| Level | Scope | KA | Output |
|-------|-------|-----|--------|
| **Architectural Design** | System-wide components, connectors, constraints | KA 02 | Architecture diagrams, ADRs |
| **Detailed Design** | Modules, classes, interfaces, algorithms | KA 03 (this KA) | Class diagrams, interface definitions, design docs |

This KA covers detailed design only. Architectural decisions belong in KA 02 and must be documented in ADRs before detailed design begins.

---

### Core Design Concepts

#### Abstraction
Abstraction is the suppression of implementation detail to focus on essential properties. Good abstraction allows a caller to use a component without knowing how it works. In Python, abstraction is achieved via:
- Functions (abstract the algorithm)
- Classes (abstract data + operations)
- Abstract Base Classes / Protocols (abstract the interface)
- Module boundaries (abstract the implementation)

**Principle:** Every abstraction should reveal what something does, not how it does it.

#### Encapsulation
Encapsulation bundles data and the methods that operate on that data, and restricts direct access to the internal state. It prevents external code from depending on internal implementation details, making internal changes safe.

In Python: use properties instead of direct attribute access for fields that have invariants; use `_prefix` for implementation-private attributes; never expose raw collections that external code could mutate.

#### Modularity
Modularity is the decomposition of a system into discrete, independently modifiable units (modules). A module has:
- A well-defined interface (what it offers)
- A hidden implementation (how it does it)
- A single primary responsibility

**Criteria for a good module boundary:** If you can describe the module's purpose in one sentence without using "and", the boundary is likely correct.

#### Information Hiding (Parnas)
Each module hides a design decision that is likely to change. Users of the module depend only on the stable interface, not the volatile implementation. When the decision changes, only the module must change.

**Practical application:** Hide the choice of database (users call a repository interface, not SQL directly), the serialization format (users call an API, not `json.dumps` directly), and the external service used (users call a gateway, not `httpx.get` directly).

#### Coupling
Coupling is the degree of interdependence between modules. **Low coupling is required.** Types of coupling (worst to best):

| Coupling Type | Description | Example |
|--------------|-------------|---------|
| **Content coupling** | Module A directly modifies Module B's internals | Module accesses private `_state` of another |
| **Common coupling** | Modules share mutable global state | Two services sharing a global dict |
| **Control coupling** | Module passes a flag that controls another module's behavior | `process(data, mode="fast")` |
| **Stamp coupling** | Modules share a composite data structure | Passing an entire `User` object when only `user.id` is needed |
| **Data coupling** | Modules communicate via simple data parameters | Pass only what is needed; return only what is produced |

Target data coupling everywhere possible.

#### Cohesion
Cohesion is the degree to which elements within a module belong together. **High cohesion is required.** Types of cohesion (worst to best):

| Cohesion Type | Description | Example |
|--------------|-------------|---------|
| **Coincidental** | Elements grouped arbitrarily | `utils.py` with unrelated functions |
| **Logical** | Elements perform similar operations of different types | `input_handling.py` for files, DB, and HTTP |
| **Temporal** | Elements execute at the same time | `startup.py` that initializes unrelated subsystems |
| **Procedural** | Elements must execute in a specific order | `process_order()` calling unrelated steps in sequence |
| **Communicational** | Elements operate on the same data | `order_service.py` — all methods operate on `Order` |
| **Sequential** | Output of one element is input to next | Pipeline stage functions |
| **Functional** | Elements all contribute to a single well-defined task | `jwt_validator.py` — only JWT validation |

Target functional or sequential cohesion. Coincidental and logical cohesion are always wrong.

---

### Low Coupling, High Cohesion: The Primary Design Goal

These two properties are the primary measurable outcome of detailed design. They directly predict:
- **Testability:** Low coupling means dependencies can be replaced with test doubles
- **Changeability:** Low coupling means changes in one module do not cascade
- **Reusability:** High cohesion means modules do their one thing well and can be reused

**How to measure:**
- **Afferent coupling (Ca):** How many modules import this module (fan-in). High Ca = this module is stable; changes are costly.
- **Efferent coupling (Ce):** How many modules this module imports (fan-out). High Ce = this module depends on many others; it is fragile.
- **Instability (I):** Ce / (Ca + Ce). Range 0–1. Core stable abstractions should have I near 0.

---

### Design by Contract

Design by Contract (DbC), from Bertrand Meyer, formalizes the relationship between a function and its callers as a contract with three components:

| Component | Definition | Who is responsible |
|-----------|-----------|-------------------|
| **Preconditions** | Conditions the caller must ensure before invoking the function | Caller |
| **Postconditions** | Conditions the function guarantees upon return | Function |
| **Invariants** | Conditions that must hold throughout the object's lifetime | Class |

**Python enforcement during development:**
```python
def withdraw(self, amount: float) -> None:
    # Precondition — caller's responsibility
    assert amount > 0, f"Amount must be positive, got {amount}"
    assert amount <= self.balance, f"Insufficient funds: {amount} > {self.balance}"

    # Operation
    self.balance -= amount

    # Postcondition — function's responsibility
    assert self.balance >= 0, "Balance cannot be negative after withdrawal"
```

**Note:** Use `assert` for DbC development-time checks only. For production validation of external input, use explicit `if/raise` (pyscg-0037).

**Type narrowing as production contract enforcement:**
```python
from typing import assert_never

def process(value: int | str | None) -> str:
    if isinstance(value, int):
        return str(value)
    elif isinstance(value, str):
        return value.upper()
    elif value is None:
        raise ValueError("value cannot be None")
    else:
        assert_never(value)  # mypy will catch unhandled cases
```

---

### Design for Testability

Testability is not an afterthought — it is a design requirement. A design that is difficult to test is a design with excessive coupling or insufficient abstraction.

**Key techniques:**

1. **Dependency Injection:** Pass dependencies in rather than creating them inside the function/class. This allows tests to inject test doubles.

```python
# Hard to test — creates its own dependency
class UserService:
    def __init__(self) -> None:
        self.db = PostgresDatabase()  # cannot be replaced in tests

# Testable — dependency injected
class UserService:
    def __init__(self, db: DatabaseProtocol) -> None:
        self.db = db  # test can inject a mock
```

2. **Seams:** Points in the code where behavior can be changed without modifying the code. Seams are created by abstractions (protocols, ABCs) and dependency injection.

3. **Interfaces before implementations:** Define the interface (Protocol or ABC) before writing the implementation. This forces the design to be driven by how the component will be used, not how it will work.

---

### SOLID Alignment

| Principle | Core Rule | Python Idiom | Design Metric |
|-----------|----------|--------------|---------------|
| **S** — Single Responsibility | A class has only one reason to change | Separate data model (dataclass) from persistence (repository) | High functional cohesion |
| **O** — Open/Closed | Open for extension; closed for modification | Use `ABC` or `Protocol`; add new implementations, don't modify existing | Afferent coupling of stable abstractions |
| **L** — Liskov Substitution | Subclass is a behavioral substitute for the base class | Never override a method to raise `NotImplementedError` unless the base class raises it | No behavioral narrowing in subclasses |
| **I** — Interface Segregation | Clients should not depend on methods they don't use | Use `Protocol` with ≤5 methods; split large ABCs | Efferent coupling from clients |
| **D** — Dependency Inversion | Depend on abstractions, not concretions | Inject `Protocol`/`ABC` types; no direct I/O instantiation in domain layer | Instability metric of domain layer ≈ 0 |

**SRP in Python:**
```python
# Violation: class does business logic AND persistence
class UserService:
    def create_user(self, name: str) -> User:
        user = User(name=name)
        cursor.execute("INSERT INTO users ...")  # persistence mixed in
        return user

# Correct: separate classes
class UserFactory:
    def create(self, name: str) -> User:
        return User(name=name)

class UserRepository:
    def __init__(self, connection: Connection) -> None:
        self.conn = connection

    def save(self, user: User) -> None:
        with self.conn.cursor() as cur:
            cur.execute("INSERT INTO users ...", (user.name,))
```

**DIP in Python (using Protocol):**
```python
from typing import Protocol

class UserStore(Protocol):
    def find(self, user_id: int) -> User | None: ...
    def save(self, user: User) -> None: ...

class UserService:
    def __init__(self, store: UserStore) -> None:  # depends on abstraction
        self.store = store

    def deactivate(self, user_id: int) -> None:
        user = self.store.find(user_id)
        if user is None:
            raise ValueError(f"User {user_id} not found")
        user.active = False
        self.store.save(user)
```

---

### Design Metrics

| Metric | Description | Target |
|--------|-------------|--------|
| **Cyclomatic complexity** | Number of independent paths through a function | ≤10 per function |
| **Afferent coupling (Ca)** | Number of external classes that depend on this class | Higher is better for stable abstractions |
| **Efferent coupling (Ce)** | Number of classes this class depends on | Lower is better; high Ce = fragile class |
| **Lack of Cohesion in Methods (LCOM)** | Fraction of method pairs that share no instance variables | Lower is better; LCOM → 0 = high cohesion |
| **Depth of inheritance** | Length of longest inheritance chain | ≤3 levels (prefer composition) |

---

### Incremental Design in Agile

SWEBOK V4 endorses "just enough design upfront" — enough to avoid costly rework, but not a full waterfall design phase:

- **Design in spikes:** Time-box architectural and design experiments before committing to an approach
- **Emergent design through refactoring:** Start with the simplest design that works; improve it as understanding grows
- **Continuous design review:** Design is part of every sprint's Definition of Done, not a one-time phase
- **Pattern application as refactoring:** Apply design patterns when complexity makes them necessary, not preemptively

---

### Anti-Patterns

| Anti-Pattern | Description | Consequence | Remedy |
|-------------|-------------|------------|--------|
| **God Class** | One class that knows and does too much | Untestable; change cascades everywhere; team bottleneck | Apply SRP; split along responsibility lines |
| **Spaghetti Code** | Complex, tangled control flow with no structure | Impossible to reason about; test coverage misleading | Refactor to smaller functions with clear structure |
| **Blob (Object Orgy)** | Objects expose all internal state; everything depends on everything | No encapsulation; any change breaks everything | Enforce information hiding; use private members |
| **Lava Flow** | Dead code, deprecated designs left in place "because we're afraid to remove them" | Cognitive load; misunderstanding of codebase; accidental use | Remove with tests as safety net; document in ADR if preserved intentionally |
| **Premature Optimization** | Optimizing before profiling proves the bottleneck | Complex code for negligible benefit; obscured intent | Profile first; optimize only proven bottlenecks |
| **Anemic Domain Model** | Domain objects are data holders with no behavior; all logic in service classes | Business rules scattered; hard to find; inconsistent | Move behavior into domain objects; apply DDD |

---

## Agent Guidance

### Do
- Apply SRP to every new class: state the single reason to change before writing the class
- Use `Protocol` for all dependency abstractions; inject dependencies, never instantiate I/O inside domain classes
- Design for testability first: if writing a unit test for the class requires >5 lines of setup, the design has excessive coupling
- Keep interfaces small: Protocol definitions with ≤5 methods; split if more are needed (ISP)
- Measure coupling before finalizing a design: high efferent coupling (Ce) is a warning sign
- Prefer composition over inheritance; limit inheritance depth to ≤3 levels
- Define the interface (Protocol/ABC) before writing the implementation
- Apply SOLID principles in order: SRP first, then OCP, then DIP — not all at once prematurely

### Do Not
- Create "God classes" that combine business logic, persistence, I/O, and presentation
- Violate LSP by overriding methods to raise `NotImplementedError` where the base class does not
- Use large inheritance hierarchies (>3 levels) — prefer composition and Protocol
- Create `utils.py` catch-all modules — this is coincidental cohesion and violates SRP
- Design in a vacuum: design must be validated by trying to write a test for it before implementation
- Apply all design patterns preemptively — introduce complexity only when the problem demands it
- Leave anti-patterns (God Class, Lava Flow, Blob) in place — address them with refactoring before adding new features on top

## Checklist
- [ ] All classes have a single identifiable reason to change (SRP)
- [ ] All external dependencies are injected (DIP); no direct I/O instantiation in domain layer
- [ ] Protocol/ABC interfaces defined before implementations
- [ ] Protocol definitions have ≤5 methods (ISP)
- [ ] No LSP violations: subclasses do not narrow behavior of base class
- [ ] Cyclomatic complexity ≤10 for all new functions
- [ ] No God classes, Blob, or Spaghetti code in modified files
- [ ] Design reviewed for testability: can a unit test be written without >5 lines of setup?
- [ ] Coupling measured: no classes with excessive efferent coupling (Ce > 7)

## See Also
- wiki/tier1-sources/swebok-v4/overview.md
- wiki/tier1-sources/swebok-v4/ka02-architecture.md
- wiki/tier1-sources/swebok-v4/ka04-construction.md
- wiki/tier1-sources/swebok-v4/ka05-testing.md
- wiki/tier2-core/solid-principles/overview.md
- wiki/tier2-core/design-patterns/overview.md
- wiki/tier3-working/python/protocols-and-abc.md
- wiki/tier3-working/python/dataclasses.md

## Source

IEEE Computer Society. *Guide to the Software Engineering Body of Knowledge (SWEBOK), Version 4.0*, KA 03: Software Design. IEEE Press, 2024.

Martin, R.C. *Clean Architecture: A Craftsman's Guide to Software Structure and Design*. Prentice Hall, 2017.

Meyer, B. *Object-Oriented Software Construction, 2nd Edition*. Prentice Hall, 1997.

Gamma, E., Helm, R., Johnson, R., Vlissides, J. *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley, 1994.
