# Value Types

WebAssembly has exactly four numeric value types in the baseline spec:

| Type | Description | C equivalent | Go equivalent |
|------|-------------|-------------|---------------|
| `i32` | 32-bit integer | `int32_t`, `uint32_t` | `int32`, `uint32` |
| `i64` | 64-bit integer | `int64_t`, `uint64_t` | `int64`, `uint64` |
| `f32` | 32-bit float | `float` | `float32` |
| `f64` | 64-bit float | `double` | `float64` |

That is the entire set for core Wasm. No booleans (use i32), no bytes (use i32), no pointers (use i32 or i64 depending on address width).

---

## Signedness

Wasm integers have no intrinsic sign — the bits are just bits. Sign is determined by the instruction:

```wat
i32.div_s   ;; signed division
i32.div_u   ;; unsigned division
i32.lt_s    ;; signed less-than
i32.lt_u    ;; unsigned less-than
```

The `_s` / `_u` suffix appears on any instruction where sign matters: division, remainder, comparison, and the extend/convert instructions. Storage is always the same bit pattern; interpretation differs.

---

## Function types

A function type (also called a signature) lists parameter types and result types:

```wat
;; No parameters, no result
(func)

;; Two i32 parameters, one i32 result
(func (param i32 i32) (result i32))

;; i32 and i64 parameters, f64 result
(func (param i32 i64) (result f64))
```

Multi-value returns are supported:

```wat
;; Returns two i32 values (quotient and remainder)
(func (param i32 i32) (result i32 i32))
```

In C, multi-value returns are usually expressed as out-parameters or a struct. In Go, multiple return values naturally map to this.

---

## Reference types

Two reference types exist for use with tables and host interop:

| Type | Description |
|------|-------------|
| `funcref` | Reference to a function (for tables, indirect calls) |
| `externref` | Opaque reference provided by the host |

These cannot be stored in linear memory — only in locals, globals, and tables. You will encounter them in [12 - Tables and Indirect Calls](12-Tables-and-Indirect-Calls).

---

## SIMD (v128)

The SIMD proposal adds `v128` — a 128-bit type representing multiple packed integers or floats. Instructions like `i32x4.add` operate on four i32 lanes simultaneously. Clang emits SIMD when targeting `wasm32` with `-msimd128`. Most major runtimes support it.

---

## What about structs and strings?

There are none at this level. Structs and strings live in **linear memory** as raw bytes and are accessed through load/store instructions. A pointer is just an `i32` offset into memory. This is exactly how C works at the ABI level, which is why C compiles so naturally to Wasm.

See [06 - Linear Memory](06-Linear-Memory) and [11 - Passing Data Across the Boundary](11-Passing-Data).

---

Previous: [02 - Modules](02-Modules) | Next: [04 - The Stack Machine](04-The-Stack-Machine)
