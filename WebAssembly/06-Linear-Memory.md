# Linear Memory

A WebAssembly instance has a **linear memory**: a contiguous, byte-addressable array of bytes. This is where all heap data, strings, stack-allocated arrays passed by pointer, and global variables that require addresses live.

---

## Pages

Memory is measured in **pages** of 64 KB each. A module declares the initial page count and an optional maximum:

```wat
(memory 1)          ;; 1 page = 65,536 bytes, no maximum
(memory 1 16)       ;; 1 page initial, 16 pages maximum
```

Memory can grow at runtime with `memory.grow`:

```wat
i32.const 1         ;; number of pages to add
memory.grow         ;; pops count, pushes old page count (or -1 on failure)
drop
```

In C, `malloc` calls `memory.grow` (or a sbrk-equivalent) when it needs more heap from the system. In Wasm, there is no OS — `memory.grow` is the only primitive for expanding available memory.

---

## Load and store

All memory access goes through explicit load/store instructions. Each instruction specifies the type, width, and alignment:

```wat
;; Store i32 value 42 at byte address 100
i32.const 100       ;; address
i32.const 42        ;; value
i32.store

;; Load i32 from byte address 100
i32.const 100
i32.load            ;; pushes 42
```

The full syntax is `i32.load offset=N align=M`:

- `offset` is a static constant added to the runtime address (useful for struct field access)
- `align` is a power-of-two hint (1, 2, 4, or 8 bytes); misaligned access still works but may be slower

Load/store variants:

| Instruction | What it does |
|-------------|-------------|
| `i32.load` | Load 4 bytes as i32 |
| `i32.load8_s` | Load 1 byte, sign-extend to i32 |
| `i32.load8_u` | Load 1 byte, zero-extend to i32 |
| `i32.load16_s` | Load 2 bytes, sign-extend to i32 |
| `i64.load32_s` | Load 4 bytes, sign-extend to i64 |
| `i32.store` | Store 4 bytes |
| `i32.store8` | Store low byte only |
| `i32.store16` | Store low 2 bytes |

---

## Bounds checking

Every memory access is bounds-checked. Accessing an address ≥ the current memory size **traps**. There is no undefined behavior for out-of-bounds access — it is always a clean trap, not silent corruption.

This is fundamentally different from native C, where an out-of-bounds access is undefined behavior. In Wasm, the memory is bounded and the hardware cannot reach outside it.

---

## Memory layout from C

When clang compiles C to Wasm, linear memory is laid out as:

```
Low addresses                                        High addresses
┌──────────────┬────────────────┬──────────────────────────────┐
│  data/bss    │  heap (malloc) │  stack (grows down)          │
│  (from data  │  (grows up)    │                              │
│   section)   │                │                              │
└──────────────┴────────────────┴──────────────────────────────┘
0            data_end        heap_top                   memory_end
```

The C stack in Wasm is a **software stack** in linear memory, distinct from the Wasm operand stack. Variables that have their address taken (`&x`), variable-length arrays, and large structs all live in this software stack.

---

## Shared memory

The host can access the same memory object. If the module exports its memory:

```wat
(memory (export "memory") 1)
```

The host can read and write bytes directly — this is how you pass complex data (strings, structs) between host and module. See [11 - Passing Data Across the Boundary](11-Passing-Data).

---

## Multiple memories

The multi-memory proposal (part of Wasm 2.0) allows more than one memory per module. Older toolchains target single-memory Wasm; check your runtime's support before relying on multiple memories.

---

Previous: [05 - Control Flow](05-Control-Flow) | Next: [07 - Imports and Exports](07-Imports-and-Exports)
