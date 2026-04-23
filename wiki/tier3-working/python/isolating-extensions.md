# Python Isolating Extension Modules (Tier 3)

> **Tier 3** | Source: Python Isolating Extensions HOWTO, docs.python.org/3/howto/isolating-extensions.html | Enforces/Derives From: wiki/tier3-working/python/free-threading.md, wiki/tier3-working/python/overview.md, wiki/tier1-sources/swebok-v4/ka04-construction.md

## Summary

C extension modules traditionally store state in process-global `static` variables. This works when there is exactly one Python interpreter per process but breaks with subinterpreters, multiple module reloads, and free-threaded Python. This HOWTO describes how to convert extension modules to store state per-module-object using multi-phase initialization and heap types, enabling safe use with `importlib` reloading, `Py_NewInterpreter()`, and free-threading. This page is primarily relevant to authors of C extension modules for Python.

## Key Concepts

### The Core Problem: Global vs Per-Module State

```c
/* LEGACY — global state breaks multiple interpreters */
static PyObject *my_error;          /* shared across all interpreters */
static int initialized = 0;

/* MODERN — per-module state */
typedef struct {
    PyObject *my_error;
    int       counter;
} MyModuleState;
```

When multiple interpreters exist in one process (e.g., `Py_NewInterpreter()`, embedded Python, `importlib` subinterpreters), each gets its own module object but would share global `static` state — leading to races and cross-interpreter contamination.

### Multi-Phase Initialization

The mechanism that enables per-module state:

```c
static int exec_module(PyObject *module) {
    MyModuleState *state = (MyModuleState *)PyModule_GetState(module);
    if (state == NULL) return -1;

    /* Initialize exception class per-module */
    state->my_error = PyErr_NewException("mymodule.MyError", NULL, NULL);
    if (state->my_error == NULL) return -1;
    if (PyModule_AddObject(module, "MyError", state->my_error) < 0) return -1;
    return 0;
}

static PyModuleDef_Slot module_slots[] = {
    {Py_mod_exec, exec_module},
    {Py_mod_multiple_interpreters, Py_MOD_PER_INTERPRETER_GIL_SUPPORTED},
    {0, NULL}
};

static struct PyModuleDef module_def = {
    .m_base    = PyModuleDef_HEAD_INIT,
    .m_name    = "mymodule",
    .m_size    = sizeof(MyModuleState),   /* Request state storage */
    .m_slots   = module_slots,
    .m_traverse = module_traverse,
    .m_clear    = module_clear,
    .m_free     = module_free,
};
```

### Accessing State from C Functions

```c
static PyObject *my_function(PyObject *module, PyObject *args) {
    MyModuleState *state = (MyModuleState *)PyModule_GetState(module);
    if (state == NULL) return NULL;

    /* Use state->my_error, state->counter, etc. */
    PyErr_SetString(state->my_error, "Something went wrong");
    return NULL;
}
```

### Heap Types vs Static Types

**Static types** (`static PyTypeObject`) are process-wide singletons — they cannot hold per-module state and are shared across all interpreters.

**Heap types** are allocated dynamically via `PyType_FromModuleAndSpec()` and can access their module's state:

```c
static PyType_Slot mytype_slots[] = {
    {Py_tp_new,    mytype_new},
    {Py_tp_dealloc, mytype_dealloc},
    {Py_tp_traverse, mytype_traverse},
    {0, NULL}
};

static PyType_Spec mytype_spec = {
    .name      = "mymodule.MyType",
    .basicsize = sizeof(MyTypeObject),
    .flags     = Py_TPFLAGS_DEFAULT | Py_TPFLAGS_HAVE_GC,
    .slots     = mytype_slots,
};

/* In exec_module(): */
PyObject *type = PyType_FromModuleAndSpec(module, &mytype_spec, NULL);
state->MyType = (PyTypeObject *)type;
```

### Accessing Module State from Type Methods

```c
/* From instance methods (Python 3.9+) — use defining class */
static PyObject *instance_method(
    PyObject *self,
    PyTypeObject *defining_class,
    PyObject *const *args,
    Py_ssize_t nargs,
    PyObject *kwnames)
{
    MyModuleState *state = (MyModuleState *)PyType_GetModuleState(defining_class);
    if (state == NULL) return NULL;
    /* use state */
}

/* From slot methods (Python 3.11+) */
static PyObject *tp_new_impl(PyTypeObject *type, PyObject *args, PyObject *kwargs) {
    PyObject *module = PyType_GetModuleByDef(type, &module_def);
    MyModuleState *state = (MyModuleState *)PyModule_GetState(module);
    if (state == NULL) return NULL;
    /* use state */
}
```

### Garbage Collection for Heap Type Instances

Heap type instances hold a reference to their type, which may create cycles. GC support is required:

```c
static int mytype_traverse(PyObject *self, visitproc visit, void *arg) {
    Py_VISIT(Py_TYPE(self));   /* visit the type object (Python 3.9+) */
    /* visit other gc-tracked members */
    return 0;
}

static void mytype_dealloc(PyObject *self) {
    PyObject_GC_UnTrack(self);
    /* cleanup instance fields */
    PyTypeObject *type = Py_TYPE(self);
    type->tp_free(self);
    Py_DECREF(type);   /* must come after tp_free */
}
```

### Process-Wide Global State (Legitimate Uses)

Global state is acceptable only for truly process-wide resources:

```c
/* Legitimate: controlling the terminal is process-global */
static int terminal_initialized = 0;
static PyMutex init_mutex = {0};

static int exec_module(PyObject *module) {
    PyMutex_Lock(&init_mutex);
    if (terminal_initialized) {
        PyMutex_Unlock(&init_mutex);
        PyErr_SetString(PyExc_ImportError, "cannot load module more than once");
        return -1;
    }
    terminal_initialized = 1;
    PyMutex_Unlock(&init_mutex);
    return 0;
}
```

## Agent Guidance

### Do

- Use `m_size = sizeof(MyState)` in `PyModuleDef` to request per-module state storage.
- Use multi-phase initialization (`Py_mod_exec` slot) instead of single-phase `PyInit_*`.
- Use heap types (`PyType_FromModuleAndSpec`) for types that need access to module state.
- Implement `m_traverse`, `m_clear`, and `m_free` for modules with heap-type members.
- Store exception classes, type objects, and configuration in per-module state, not `static` variables.
- Declare `Py_mod_multiple_interpreters: Py_MOD_PER_INTERPRETER_GIL_SUPPORTED` when the module is subinterpreter-safe.

### Do Not

- Do not use `static` variables for exception classes, type objects, or any state that should be per-module.
- Do not use static types (`static PyTypeObject`) when the type needs to access module state.
- Do not forget to call `Py_DECREF(type)` after `tp_free(self)` in the dealloc of heap type instances.
- Do not implement `tp_traverse` without visiting `Py_TYPE(self)` for heap types (required since Python 3.9).

## Checklist

- [ ] `m_size = sizeof(ModuleState)` set in `PyModuleDef`
- [ ] Module uses multi-phase initialization (exec slot, not single-phase)
- [ ] No exception classes or type objects stored in `static` variables
- [ ] Heap types used for classes requiring module state access
- [ ] `m_traverse`, `m_clear`, `m_free` implemented for cleanup
- [ ] `Py_TYPE(self)` visited in instance `tp_traverse`
- [ ] `Py_DECREF(type)` called after `tp_free` in heap type dealloc

## See Also

- wiki/tier3-working/python/free-threading.md
- wiki/tier3-working/python/overview.md
- wiki/tier1-sources/swebok-v4/ka04-construction.md
- wiki/tier3-working/python/mro.md

## Source

Python Isolating Extension Modules HOWTO, docs.python.org/3/howto/isolating-extensions.html. CPython C API documentation. Example: cpython/Modules/xxlimited.c.
