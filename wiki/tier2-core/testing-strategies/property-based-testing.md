# Property-Based Testing

> **Tier 2** | Source: Hypothesis library / QuickCheck tradition | Derives From: ka05-testing | Authority: established practice

## Summary

Instead of specifying exact inputs and expected outputs, property-based testing specifies *properties* — invariants that must hold for all valid inputs — and lets a library generate hundreds of random test cases automatically. This finds edge cases that humans consistently miss and is approximately 50 times more effective at killing mutants than traditional example-based tests.

## Key Concepts

### What vs. Why

**Example-based test**: "When I sort [3, 1, 2], I get [1, 2, 3]."

**Property-based test**: "When I sort any list, the result is the same length as the input, all elements from the input are present in the result, and each element is less than or equal to the next element."

The property-based test exercises thousands of inputs automatically, including empty lists, single-element lists, lists with duplicates, lists with negative numbers, and lists with extreme values — all cases the developer may not have written explicitly.

### The Hypothesis Library

Hypothesis is the standard property-based testing library for Python. It integrates with pytest, provides built-in strategies for generating test data, and automatically shrinks failing examples to their minimal form.

```bash
pip install hypothesis
```

### Core Strategies

| Strategy | What It Generates | Example |
|----------|------------------|---------|
| `st.integers()` | Integer values, including MIN/MAX | `st.integers(min_value=0, max_value=1_000_000)` |
| `st.text()` | Unicode strings, including empty, long, emoji | `st.text(alphabet=st.characters(whitelist_categories=("Lu", "Ll")))` |
| `st.floats()` | Floats including NaN, Inf, -0.0 | `st.floats(allow_nan=False, allow_infinity=False)` |
| `st.booleans()` | `True` and `False` | |
| `st.lists()` | Lists of a given strategy | `st.lists(st.integers(), min_size=1, max_size=100)` |
| `st.dictionaries()` | Dicts with given key/value strategies | `st.dictionaries(st.text(), st.integers())` |
| `st.from_type()` | Values based on a Python type annotation | `st.from_type(MyDataclass)` |
| `st.builds()` | Instances of a class | `st.builds(Order, customer_id=st.integers(min_value=1))` |
| `st.one_of()` | One of multiple strategies | `st.one_of(st.none(), st.integers())` |

### Composite Strategies

For complex domain objects, use `@st.composite`:

```python
from hypothesis import strategies as st

@st.composite
def order_strategy(draw) -> dict:
    customer_id = draw(st.integers(min_value=1, max_value=999_999))
    amount = draw(st.floats(min_value=0.01, max_value=10_000.00, allow_nan=False))
    currency = draw(st.sampled_from(["USD", "EUR", "GBP"]))
    return {"customer_id": customer_id, "amount": amount, "currency": currency}
```

### Settings

```python
from hypothesis import given, settings, HealthCheck
from hypothesis import strategies as st

@settings(max_examples=500, suppress_health_check=[HealthCheck.too_slow])
@given(st.text())
def test_url_encode_decode_roundtrip(text: str) -> None:
    from urllib.parse import quote, unquote
    assert unquote(quote(text)) == text
```

- `max_examples`: number of test cases generated (default: 100; use 500+ for thorough testing of critical functions)
- `suppress_health_check`: silence Hypothesis warnings about slow strategies when justified

### Shrinking

When Hypothesis finds a failing input, it automatically **shrinks** it to the minimal failing case. Instead of seeing a 200-character string that causes a failure, you see the 3-character string that triggers the same bug. This dramatically reduces debugging time.

---

## Property Identification Guide

| Code Pattern | Natural Property | Hypothesis Example |
|-------------|------------------|--------------------|
| `encode(x)` / `decode(y)` | **Roundtrip**: `decode(encode(x)) == x` | `@given(st.text())` then assert roundtrip |
| `sort(x)` | **Idempotency**: `sort(sort(x)) == sort(x)`; **Content**: same elements | `@given(st.lists(st.integers()))` |
| `compress(x)` / `decompress(y)` | **Roundtrip** + size reduction | `@given(st.binary())` |
| `parse(x)` / `serialize(y)` | **Roundtrip**: `parse(serialize(x)) == x` | `@given(st.builds(MyModel, ...))` |
| Mathematical operation | **Associativity**, **commutativity**, **identity element** | `@given(st.integers(), st.integers())` |
| `validate(x)` | **Safety**: never raises unexpected exceptions; only `ValueError` | `@given(st.text())` |
| Container (add item) | **Size monotonically increases** | `@given(st.lists(st.integers()))` |
| Idempotent operation | Applying twice = applying once | `@given(st.text())` |

---

## Full Working Example — JSON Serializer Roundtrip

```python
import json
from hypothesis import given, settings
from hypothesis import strategies as st

# The function under test
def serialize_record(record: dict) -> str:
    return json.dumps(record, ensure_ascii=False, sort_keys=True)

def deserialize_record(serialized: str) -> dict:
    return json.loads(serialized)

# Property: serialize then deserialize must return the original value
@given(
    st.dictionaries(
        keys=st.text(min_size=1),
        values=st.one_of(
            st.integers(),
            st.floats(allow_nan=False, allow_infinity=False),
            st.text(),
            st.booleans(),
            st.none(),
        ),
        min_size=0,
        max_size=10,
    )
)
@settings(max_examples=500)
def test_json_roundtrip(record: dict) -> None:
    serialized = serialize_record(record)
    recovered = deserialize_record(serialized)
    assert recovered == record, f"Roundtrip failed: {record!r} -> {serialized!r} -> {recovered!r}"
```

This single test generates 500 random dictionaries and verifies the roundtrip property for all of them. A developer writing example-based tests would never manually write 500 test cases.

---

## Second Example — Validation Safety Property

```python
from hypothesis import given
from hypothesis import strategies as st

def validate_email(email: str) -> bool:
    """Should never crash; should return bool."""
    import re
    pattern = r'^[a-zA-Z0-9._%+\-]+@[a-zA-Z0-9.\-]+\.[a-zA-Z]{2,}$'
    return bool(re.match(pattern, email))

@given(st.text())
def test_validate_email_never_crashes(email: str) -> None:
    # Property: validate_email must return a bool for any input; never raise
    result = validate_email(email)
    assert isinstance(result, bool)
```

---

## When to Use Property-Based Testing

| Use Case | Use Property-Based Testing? | Why |
|----------|-----------------------------|-----|
| Serialization / deserialization | Yes | Roundtrip property covers all edge cases |
| Encoding / decoding | Yes | Roundtrip property |
| Input validation | Yes | Safety property: never crashes unexpectedly |
| Mathematical operations | Yes | Algebraic properties (associativity, commutativity) |
| Data transformations / pipelines | Yes | Input/output invariants |
| Sorting, filtering, grouping | Yes | Content, ordering, size invariants |
| UI rendering | No | Use snapshot tests |
| Database integration | No | Use integration tests with real data |
| Non-deterministic operations | No | Properties cannot be asserted if output varies |

## Agent Guidance

### Do
- Write property-based tests for every serialization, encoding, validation, and transformation function
- Use the property identification table to find the right property for each function
- Set `max_examples=500` for business-critical or complex functions
- Run Hypothesis tests in the same pytest session as unit tests — they run fast

### Do Not
- Do not use property-based testing for UI tests, database integration, or non-deterministic operations
- Do not suppress all health checks permanently — investigate the underlying cause
- Do not stop at one property per function — identify and test multiple properties

## Checklist
- [ ] Roundtrip property tested for all encode/decode and serialize/parse pairs
- [ ] Safety property tested for all validation functions
- [ ] Algebraic properties tested for mathematical operations
- [ ] `max_examples` is set to 500+ for business-critical modules
- [ ] Hypothesis is integrated into the standard pytest run (not a separate CI stage)

## See Also
- `wiki/tier2-core/testing-strategies/overview.md`
- `wiki/tier2-core/testing-strategies/test-pyramid.md`
- `wiki/tier2-core/testing-strategies/mutation-testing.md`
- `wiki/tier1-sources/swebok-v4/ka05-testing.md`

## Source

Hypothesis documentation (hypothesis.works); John Hughes, "QuickCheck: A Lightweight Tool for Random Testing of Haskell Programs" (2000). Synthesized from *Software Development Best Practices for Agent* reference document — property-based tests are ~50× more effective at mutation killing than example-based tests.
