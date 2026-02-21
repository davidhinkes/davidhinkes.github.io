# WebAssembly

A course on how WebAssembly works, from binary format through running C and Go programs. Each lesson is self-contained but builds on earlier material.

---

## Part 1: What WebAssembly Is

1. [01 - What is WebAssembly](01-What-is-WebAssembly) — Portable bytecode, stack machine, sandbox. Why it exists.
2. [02 - Modules](02-Modules) — The .wasm file: sections, binary format, text format (WAT).
3. [03 - Value Types](03-Value-Types) — i32, i64, f32, f64. Function signatures. Everything is a number.

## Part 2: The Core Machine

4. [04 - The Stack Machine](04-The-Stack-Machine) — Operand stack, instructions, locals, globals.
5. [05 - Control Flow](05-Control-Flow) — block, loop, if/else, br, br_if. Structured control.
6. [06 - Linear Memory](06-Linear-Memory) — One flat byte array. Pages, load/store, bounds checking.
7. [07 - Imports and Exports](07-Imports-and-Exports) — The host/module boundary. What flows in and out.

## Part 3: C and Go to WebAssembly

8. [08 - C to WebAssembly](08-C-to-WebAssembly) — clang targets, wasi-sdk, Emscripten, the ABI.
9. [09 - Go to WebAssembly](09-Go-to-WebAssembly) — GOOS=wasip1, GOOS=js, TinyGo, wazero.
10. [10 - WASI](10-WASI) — The system interface. fd_write, preopened dirs, runtimes.

## Part 4: The Host Boundary

11. [11 - Passing Data Across the Boundary](11-Passing-Data) — Strings, structs, and shared memory.
12. [12 - Tables and Indirect Calls](12-Tables-and-Indirect-Calls) — Function pointers, callbacks, dynamic dispatch.

---

## Quick reference

| Concept | What it is |
|---------|-----------|
| Module | A .wasm file: the unit of compilation and deployment |
| Instance | A running module with its own linear memory and mutable state |
| Linear memory | A flat byte array shared between host and module |
| Operand stack | The implicit stack that instructions push/pop |
| WASI | Import functions that give modules OS-like capabilities |
| WAT | WebAssembly Text format — human-readable .wat files |
| wasi-sdk | Clang + musl libc configured for wasm32-wasi |
| wazero | Pure-Go Wasm runtime, no CGo required |

## Toolchain cheat sheet

```
# C → Wasm (WASI)
$WASI_SDK_PATH/bin/clang -o app.wasm main.c

# Go → Wasm (WASI)
GOOS=wasip1 GOARCH=wasm go build -o app.wasm .

# Go → Wasm (browser)
GOOS=js GOARCH=wasm go build -o main.wasm .

# Run with wasmtime
wasmtime app.wasm

# Run Go tests as Wasm
GOARCH=wasm GOOS=wasip1 go test ./...

# Inspect a .wasm file
wasm2wat app.wasm
wasm-objdump -x app.wasm
```
