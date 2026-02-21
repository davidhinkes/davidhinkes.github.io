# Imports and Exports

The boundary between a WebAssembly module and the outside world is defined entirely by **imports** (what the module needs) and **exports** (what the module provides). Everything that crosses this boundary is explicit and declared in the binary.

---

## Imports

At instantiation time, the host provides an **import object**: a map from (module name, field name) pairs to values. The module declares exactly what it needs in its import section.

```wat
(module
  ;; Import a function "log_i32" from the host "env" namespace
  (import "env" "log_i32" (func $log (param i32)))

  ;; Import memory from the host
  (import "env" "memory" (memory 1))

  (func $main
    i32.const 42
    call $log))
```

The host must supply every declared import or instantiation fails. Importable things:

| Kind | Declaration |
|------|-------------|
| Function | `(import "ns" "name" (func (param ...) (result ...)))` |
| Memory | `(import "ns" "name" (memory N))` |
| Table | `(import "ns" "name" (table N funcref))` |
| Global | `(import "ns" "name" (global i32))` |

---

## Exports

Exports make module internals visible to the host by name.

```wat
(module
  (func $add (param i32 i32) (result i32)
    local.get 0
    local.get 1
    i32.add)
  (export "add" (func $add))

  (memory 1)
  (export "memory" (memory 0)))
```

The host calls `"add"` by name and can read/write `"memory"` directly. Anything not exported is invisible to the host — a form of encapsulation enforced at the binary level.

---

## The namespace convention

Import names have two parts: module name and field name. The conventions vary by runtime:

| Runtime | Module name | What provides it |
|---------|------------|-----------------|
| WASI runtimes | `"wasi_snapshot_preview1"` | Runtime's WASI implementation |
| Browsers (JS) | `"imports"` (or any name) | JS object passed at instantiation |
| wazero (Go) | custom | Go code registered with `r.NewHostModuleBuilder` |
| Emscripten | `"env"` | Emscripten runtime functions |

---

## WASI imports

A WASI-compliant module imports all OS capabilities from `"wasi_snapshot_preview1"`:

```wat
(import "wasi_snapshot_preview1" "fd_write"  (func ...))
(import "wasi_snapshot_preview1" "proc_exit" (func (param i32)))
```

The runtime (wasmtime, wasmer, wazero) provides these. The module doesn't know or care whether it runs on Linux, macOS, or Windows.

---

## Start function

A module can declare a **start function** — executed automatically at instantiation, before the host calls any exports. Analogous to a C `__attribute__((constructor))` or a Go `init`.

```wat
(start $init)
```

WASI programs use `_start` as their entry point by convention. The runtime calls the exported `"_start"` function explicitly, which then calls `main`.

---

## Thinking in terms of ABI

Imports and exports together define the module's **ABI** — the contract between module and host. Because the only values that cross the boundary are Wasm value types (i32, i64, f32, f64, and reference types), any richer data (strings, structs, byte slices) must be passed through shared linear memory. This is covered in [11 - Passing Data Across the Boundary](11-Passing-Data).

---

Previous: [06 - Linear Memory](06-Linear-Memory) | Next: [08 - C to WebAssembly](08-C-to-WebAssembly)
