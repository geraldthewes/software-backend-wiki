# Mathematical Foundations (KA17)

> **Tier 1** | Source: SWEBOK V4, Chapter 17 | Authority: immutable

## Summary

Mathematical Foundations covers the discrete mathematics, logic, statistics, and graph theory that underpin formal software reasoning. While most software is not formally verified, these mathematical structures appear everywhere in software engineering: logic in precondition/postcondition specifications, graph theory in dependency analysis and scheduling, probability in performance analysis and A/B testing, and Boolean algebra in test coverage analysis. Understanding these foundations enables agents to reason precisely rather than informally about software behavior.

This KA is intentionally brief and practical. The goal is not mathematical mastery but mathematical literacy — knowing when a mathematical tool applies and being able to use it at a working level. Precision in problem formulation, even without full formal proof, reduces ambiguity and surfaces hidden assumptions.

## Key Concepts

### Logic

**Propositional logic**: Reasoning about true/false propositions using connectives:
- AND (∧): Both must be true
- OR (∨): At least one must be true
- NOT (¬): Negation
- IMPLICATION (→): If P then Q; equivalent to ¬P ∨ Q
- EQUIVALENCE (↔): P if and only if Q

**Application**: Preconditions and postconditions. A function's contract is a logical statement:
- Precondition: What must be true before the function is called
- Postcondition: What is guaranteed to be true after the function returns
- Invariant: What remains true throughout (loop invariant, class invariant)

Example:
```
# Precondition: n >= 0
# Postcondition: result == n!
def factorial(n: int) -> int: ...
```

**Predicate logic**: Extends propositional logic with quantifiers:
- ∀x: "for all x" — used in specifications covering all elements of a collection
- ∃x: "there exists x" — used in specifications asserting the existence of a value

**Implication chain**: If A→B and B→C, then A→C. Used in reasoning about inheritance contracts (Liskov Substitution Principle relies on implication chains).

### Set Theory

**Sets**: Unordered collections of unique elements. Foundation for type theory and database theory.
- Set membership: x ∈ S
- Subset: A ⊆ B (every element of A is also in B)
- Union: A ∪ B, Intersection: A ∩ B, Difference: A \ B

**Relations**: A relation R between sets A and B is a subset of A × B (the Cartesian product). Database foreign keys, class associations, and dependency graphs are all relations.

**Functions**: A special type of relation where each element of the domain maps to exactly one element of the codomain. Type signatures in programming languages are function type declarations.

**Application**: Database normalization (functional dependencies between attributes), API contract specification (what inputs map to what outputs), and type system reasoning.

### Graph Theory

**Basic concepts**:
- **Graph**: Nodes (vertices) and edges (connections) between them
- **Directed graph (digraph)**: Edges have direction — used for dependency graphs, call graphs, state machines
- **Weighted graph**: Edges have numeric weights — used for network routing, shortest path problems
- **Tree**: Connected acyclic graph — used for file systems, parse trees, decision trees
- **DAG (Directed Acyclic Graph)**: Directed graph with no cycles — used for dependency resolution (build systems, package managers, task scheduling)

**Key algorithms**:
- **BFS (Breadth-First Search)**: Finds shortest path in unweighted graphs; level-order traversal
- **DFS (Depth-First Search)**: Topological sort, cycle detection, strongly connected components
- **Topological sort**: Orders nodes in a DAG such that all dependencies come before dependents. Used by build systems (Make, Bazel), package managers (npm, pip), and CI/CD pipeline orchestration.
- **Shortest path**: Dijkstra's algorithm (non-negative weights), Bellman-Ford (negative weights), Floyd-Warshall (all pairs)

**Application to software**:
- **Dependency analysis**: Package dependencies form a DAG; cycles indicate circular dependencies (usually a design error)
- **Call graph**: Function call relationships; used by profilers, dead code detection, and security analysis
- **State machines**: Finite automaton is a directed graph where nodes are states and edges are transitions
- **Network topology**: Physical and logical network layouts are graphs; routing is a shortest-path problem

### Probability and Statistics

**Distributions**: Understanding data distributions prevents misinterpretation of performance results.
- **Normal distribution**: Bell curve; mean ± standard deviation. Latency distributions are usually NOT normal — they are right-skewed.
- **Log-normal**: Better model for latency; use for sizing capacity based on percentiles.

**Percentiles**: p50 (median), p95, p99, p99.9. Service Level Objectives are defined in percentiles, not averages. The average hides the tail — users experiencing the tail are the most likely to churn.

**Confidence intervals**: A range that contains the true parameter with a specified probability (e.g., 95%). Required for honest performance benchmarking — "X is 10% faster than Y" is meaningless without a confidence interval.

**Hypothesis testing**: Framework for evaluating whether an observed difference is real or due to random variation.
- **Null hypothesis (H₀)**: No difference between treatments
- **p-value**: Probability of observing the result (or more extreme) if H₀ is true. p < 0.05 is the conventional threshold for statistical significance.
- **Statistical power**: Probability of detecting a true effect. Underpowered A/B tests produce inconclusive results.

**A/B testing**: Controlled experiment assigning users to control (A) and treatment (B) groups to measure the causal effect of a change. Requires: random assignment, sufficient sample size (power analysis), single metric primary (avoid p-hacking), pre-registered hypothesis.

### Discrete Mathematics

**Combinatorics**: Counting arrangements. Used in:
- Test case enumeration (how many combinations of inputs must be tested?)
- Hash collision probability analysis
- Cryptographic key space calculation

**Number theory**: Modular arithmetic, prime factorization. Used in:
- Cryptography (RSA, Diffie-Hellman rely on prime factorization hardness)
- Hash functions (modular arithmetic for bucket placement)
- Pseudorandom number generation

### Boolean Algebra

**Logic gates and truth tables**: AND, OR, NOT, XOR operations on binary values. Foundation of digital logic but also directly applicable to:
- **Test coverage analysis**: Decision coverage requires testing each branch condition as both true and false. Multiple condition coverage (MC/DC) requires testing all combinations of compound conditions.
- **Feature flag logic**: Complex combinations of user segment conditions are Boolean expressions; truth table analysis ensures all cases are covered.
- **Access control policy**: Permission systems are Boolean expressions over user attributes and resource attributes.

**De Morgan's Laws**: ¬(A ∧ B) = ¬A ∨ ¬B; ¬(A ∨ B) = ¬A ∧ ¬B. Used when negating compound conditions (common in test case inversion and access control analysis).

## Agent Guidance

### Do
- Write formal preconditions and postconditions for non-trivial functions, using logical notation where precision matters
- Use topological sort for dependency ordering in build systems, migration scripts, and task scheduling
- Report performance results with confidence intervals and percentile distributions, not just averages
- Apply hypothesis testing to A/B test results before declaring a winner
- Use truth tables to verify completeness of test cases for compound Boolean conditions
- Apply graph analysis (cycle detection, topological sort) when designing dependency structures

### Do Not
- Report performance benchmarks using only the mean — always include p95/p99 and confidence intervals
- Launch A/B tests without a pre-registered hypothesis and statistical power analysis
- Declare A/B test winners before reaching the predetermined sample size (peeking problem)
- Ignore circular dependencies in module or package structures — use topological sort to detect them early

## Checklist
- [ ] Preconditions and postconditions documented for public APIs
- [ ] Performance benchmarks reported with percentiles and confidence intervals
- [ ] A/B tests have defined sample size (power analysis) before launch
- [ ] Dependency graphs checked for cycles before committing new dependencies
- [ ] Boolean conditions in access control and feature flags verified with truth table analysis

## See Also
- `wiki/tier1-sources/swebok-v4/ka16-computing-foundations.md`
- `wiki/tier1-sources/swebok-v4/ka05-testing.md`
- `wiki/tier1-sources/swebok-v4/ka18-engineering-foundations.md`

## Source
IEEE Computer Society. *Guide to the Software Engineering Body of Knowledge (SWEBOK), Version 4*. Chapter 17: Mathematical Foundations. IEEE Press, 2024.
