# Python Method Resolution Order (MRO) (Tier 3)

> **Tier 3** | Source: Python MRO HOWTO, docs.python.org/3/howto/mro.html | Enforces/Derives From: wiki/tier3-working/python/overview.md, wiki/tier2-core/solid-principles/lsp.md, wiki/tier2-core/solid-principles/overview.md

## Summary

Python uses the C3 linearization algorithm to determine the Method Resolution Order (MRO) — the sequence in which base classes are searched when looking up an attribute or method. Understanding MRO is essential for correct use of multiple inheritance and `super()`. Python raises `TypeError` at class definition time when the MRO is inconsistent, preventing silent ordering bugs. In practice: prefer single inheritance or mixins; use `super()` consistently in cooperative hierarchies.

## Key Concepts

### Accessing the MRO

```python
class A: pass
class B(A): pass
class C(A): pass
class D(B, C): pass

D.__mro__
# (<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>)

D.mro()
# [<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>]
```

Python searches this list left-to-right when resolving any name on an instance or class of `D`.

### C3 Linearization

The MRO for class `C(B1, B2, ..., BN)` is:

```
L[C] = C + merge(L[B1], L[B2], ..., L[BN], [B1, B2, ..., BN])
```

**Merge rules**: Take the head of the first list if it does not appear in the tail of any other list. Remove that head from all lists. Repeat. Raise `TypeError` if no valid head can be found.

**Properties of C3:**
- Monotonic: class order from parent hierarchies is preserved in all subclasses
- Local precedence: left-to-right order of bases is respected
- Consistent: no class can be linearized in conflicting ways

### Diamond Inheritance (C3 Handles It)

```python
class Base:
    def method(self) -> str:
        return "Base"

class Left(Base):
    def method(self) -> str:
        return f"Left → {super().method()}"

class Right(Base):
    def method(self) -> str:
        return f"Right → {super().method()}"

class Diamond(Left, Right):
    def method(self) -> str:
        return f"Diamond → {super().method()}"

d = Diamond()
d.method()
# "Diamond → Left → Right → Base"

Diamond.mro()
# [Diamond, Left, Right, Base, object]
```

`Base.method()` is called exactly once because each class calls `super()` and the MRO routes through `Right` before `Base`.

### When MRO Fails

Python raises `TypeError` at class definition when the hierarchy is inconsistent:

```python
class X: pass
class Y: pass
class A(X, Y): pass
class B(Y, X): pass

class C(A, B): pass
# TypeError: Cannot create a consistent method resolution order (MRO)
# for bases X, Y — X precedes Y in A but Y precedes X in B
```

This is caught at class creation — it cannot happen at runtime unexpectedly.

### Cooperative Multiple Inheritance with super()

`super()` does not call "the parent class" — it calls the **next class in the MRO**. All classes in a cooperative hierarchy must call `super()`:

```python
class LoggingMixin:
    def save(self) -> None:
        import logging
        logging.getLogger(__name__).info("Saving %r", self)
        super().save()   # Must call super() even though LoggingMixin has no parent save()

class TimestampMixin:
    def save(self) -> None:
        import datetime
        self.updated_at = datetime.datetime.utcnow()
        super().save()

class Base:
    def save(self) -> None:
        # actual persistence
        ...

class Model(LoggingMixin, TimestampMixin, Base):
    pass

# MRO: Model → LoggingMixin → TimestampMixin → Base → object
# Calling Model().save() invokes each mixin in order before Base.save()
```

**If one class in the chain omits `super().save()`, all subsequent classes in the MRO are silently skipped.**

### Inspecting MRO for Debugging

```python
# Show each class's contribution
for cls in MyClass.mro():
    if 'method_name' in cls.__dict__:
        print(cls.__name__, "defines method_name")
```

## Agent Guidance

### Do

- Use `super()` without arguments (Python 3 form) in cooperative multiple inheritance.
- Call `super()` in every class that participates in a cooperative hierarchy — even in the class that "ends" the chain, unless it is `object`.
- List more specialized (concrete) classes before more general (mixin) classes in the bases list.
- Inspect `ClassName.mro()` when debugging unexpected attribute resolution behavior.
- Raise `TypeError` deliberately by testing inconsistent hierarchies in CI — Python's MRO validation runs at class definition.

### Do Not

- Do not create inheritance hierarchies where base class order conflicts across the hierarchy (`A(X, Y)` with `B(Y, X)`) — Python will reject the combined subclass.
- Do not mix `super()` and direct parent calls (`Parent.method(self, ...)`) in the same cooperative hierarchy — direct calls break the chain.
- Do not rely on MRO to select between two conflicting implementations — that is an LSP violation; use composition or protocols instead.
- Do not duplicate base classes in the inheritance list: `class A(B, B): ...` raises `TypeError`.

## Checklist

- [ ] All classes in a cooperative mixin chain call `super()` at the appropriate point
- [ ] `super()` used without arguments (Python 3 form)
- [ ] Base class ordering puts more specific classes before more general ones
- [ ] `ClassName.mro()` checked when debugging surprising attribute resolution
- [ ] No conflicting ordering across related base classes

## See Also

- wiki/tier3-working/python/descriptors.md
- wiki/tier2-core/solid-principles/lsp.md
- wiki/tier2-core/solid-principles/overview.md
- wiki/tier3-working/python/type-system.md
- wiki/tier3-working/python/overview.md

## Source

Python MRO HOWTO, docs.python.org/3/howto/mro.html. Samuele Pedroni and Guido van Rossum, "The Python 2.3 Method Resolution Order." Original C3 algorithm: Kim Barrett et al., "A Monotonic Superclass Linearization for Dylan" (OOPSLA 1996).
