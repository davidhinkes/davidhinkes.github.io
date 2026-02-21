# Modules

A WebAssembly module is a `.wasm` binary file. It is the unit of compilation and deployment — analogous to a shared library (`.so` or `.dll`), but portable and sandboxed.

---

## Sections

A module is a sequence of typed sections. Each section has an ID byte and a byte length, so parsers can skip unknown sections. Sections you will encounter most often:

| Section | Contents |
|---------|----------|
| Type | Function signatures: parameter types and result types |
| Import | External things the module needs from the host |
| Function | Maps each local function index to its type signature |
| Table | Tables of function references |
| Memory | Declares linear memory (initial and max page counts) |
| Global | Module-level mutable or immutable variables |
| Export | What the module exposes to the host, by name |
| Code | The actual bytecode for each function |
| Data | Initial byte contents copied into linear memory at startup |

The binary format is designed to be validated and compiled in a single forward pass — no section references anything declared later.

---

## Binary format vs. text format

`.wasm` files are binary. There is also a text representation called **WAT** (WebAssembly Text format) that uses S-expressions and is human-readable. `wat2wasm` and `wasm2wat` convert between them.

A minimal module in WAT:

```wat
(module
  (func $add (param $a i32) (param $b i32) (result i32)
    local.get $a
    local.get $b
    i32.add)
  (export "add" (func $add)))
```

This module exports one function named `"add"`. The host calls it by that string name.

In binary, the same module is around 30 bytes. The encoding uses LEB128 for integers (variable-length, compact for small values).

---

## Instantiation

Loading a `.wasm` file gives you a **module**: a parsed, validated, stateless description. **Instantiation** takes a module plus the host's provided imports and creates a live **instance** with:

- Its own copy of linear memory (initialized from the data section)
- Its own mutable globals
- Function bodies shared (immutable) across all instances of the same module

```
module (stateless) + imports → instance (stateful)
```

You can instantiate the same module multiple times. Each instance is independent.

---

## The magic bytes

Every `.wasm` file starts with the 4-byte magic `\0asm` followed by the 4-byte version `\x01\x00\x00\x00`. A file that doesn't start with these 8 bytes is not valid Wasm.

```
00 61 73 6d  01 00 00 00  ...
 \0  a  s  m  version=1
```

---

## Inspecting modules

```bash
# Convert binary to readable text
wasm2wat app.wasm

# Show imports, exports, and section sizes
wasm-objdump -x app.wasm

# Validate a module
wasm-validate app.wasm
```

These tools ship with the [WebAssembly Binary Toolkit (wabt)](https://github.com/WebAssembly/wabt).

---

Previous: [01 - What is WebAssembly](01-What-is-WebAssembly) | Next: [03 - Value Types](03-Value-Types)
