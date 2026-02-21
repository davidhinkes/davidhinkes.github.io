# Tables and Indirect Calls

A **table** is an indexed array of references — most commonly function references. Tables enable **indirect calls**: calling a function via an index rather than a direct name. This is how Wasm implements function pointers, virtual dispatch, and callbacks from a host.

---

## Why tables exist

In Wasm, the code section is not addressable. You cannot take the address of a function and store it in memory as an i32, because that would allow jumping to arbitrary code offsets — breaking the structured control flow guarantee.

Instead, functions that need to be called indirectly are stored in a **table**. A C function pointer becomes an index into the table. The runtime validates the call at the table boundary: it checks that the slot is not null and that the function's signature matches what the call site expects.

---

## Declaring a table

```wat
(table 10 funcref)          ;; 10 slots, holding function references
```

Tables can be imported:

```wat
(import "env" "table" (table 0 funcref))
```

And exported for the host to inspect:

```wat
(table (export "indirect_fn_table") 10 funcref)
```

---

## Populating a table

The `elem` section fills table slots with function references at instantiation:

```wat
(table 3 funcref)
(elem (i32.const 0) $add $sub $mul)
;; slot 0 = $add, slot 1 = $sub, slot 2 = $mul
```

You can also fill slots at runtime using `table.set` (part of the reference types proposal).

---

## call_indirect

`call_indirect` pops an i32 index from the stack, looks up the function in the table, type-checks it, and calls it:

```wat
(type $binop (func (param i32 i32) (result i32)))

local.get $a
local.get $b
local.get $fn_index      ;; which slot in the table?
call_indirect (type $binop)
```

If the slot is null, or if the function's actual signature doesn't match `$binop`, a **trap** occurs. This is the safety guarantee: indirect calls can only succeed with the correct signature.

---

## C function pointers

When clang compiles C with function pointers, it translates them to table indices automatically:

```c
typedef int (*BinopFn)(int, int);

int add(int a, int b) { return a + b; }
int sub(int a, int b) { return a - b; }

int apply(BinopFn fn, int a, int b) {
    return fn(a, b);  // compiles to: call_indirect
}

int main(void) {
    return apply(add, 10, 3);  // passes table index of "add"
}
```

In the compiled Wasm:

- `add` and `sub` are placed in the `__indirect_function_table` at specific indices
- Passing `add` as a function pointer means passing its table index (an i32)
- Calling through the pointer uses `call_indirect` with a type check

To see this in action: `wasm2wat app.wasm | grep call_indirect`

---

## Callbacks from host to module

If the host needs to provide a function for the module to call back — a common plugin pattern — it imports a table and populates it, or the module exposes named exports and the host calls them directly.

More commonly with wazero, the host registers functions as imports:

```go
// Go host registers a callback the module can call
r.NewHostModuleBuilder("env").
    NewFunctionBuilder().
    WithFunc(func(ctx context.Context, x int32) {
        fmt.Println("module called back with", x)
    }).
    Export("on_event").
    Instantiate(ctx)
```

---

## Virtual dispatch

C++ virtual functions compile to:
1. A vtable in the data section — an array of function table indices
2. A load from the vtable to get the index
3. `call_indirect` with the index

Go interfaces compiled with TinyGo follow the same pattern: the `itab` becomes a data structure holding table indices for the method implementations.

---

## Reference types and externref

With the reference types proposal, tables can hold `externref` — opaque values from the host (DOM nodes, JS objects, host resources). The host can pass complex objects to the module as table entries without ever serializing them to linear memory.

```wat
(table 10 externref)
```

This is foundational to the Component Model's resource handle concept, where host-managed objects are represented as typed references rather than integer handles.

---

## Summary: how code and data cross the boundary

| What | Mechanism |
|------|-----------|
| Scalar values (int, float) | Function parameters and results |
| Strings, arrays, structs | Linear memory (pointer + length) |
| Function pointers (C) | Table index |
| Host objects | externref in a table |
| OS capabilities | WASI imports |

---

Previous: [11 - Passing Data Across the Boundary](11-Passing-Data) | Course index: [00 - WebAssembly Course Index](00-WebAssembly-Course-Index)
