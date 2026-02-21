# C to WebAssembly

C is the most natural language for WebAssembly. C's memory model — flat address space, explicit allocation, no GC — maps directly to Wasm's linear memory, and clang has supported Wasm targets since 2017.

---

## Targets

Two main clang targets for C to Wasm:

| Target triple | Use case | Notes |
|---------------|----------|-------|
| `wasm32-wasi` | Server-side, CLI, embedded | Full libc via wasi-sdk |
| `wasm32-unknown-emscripten` | Browser with JS glue | Emscripten toolchain |
| `wasm32-unknown-unknown` | Freestanding / no OS | No libc by default |

For server-side and general use, `wasm32-wasi` via **wasi-sdk** is the standard choice.

---

## Compiling with wasi-sdk

[wasi-sdk](https://github.com/WebAssembly/wasi-sdk) provides a clang configured for `wasm32-wasi` with a complete libc (based on musl). Set `WASI_SDK_PATH` to the install directory.

```bash
# Simple compilation
$WASI_SDK_PATH/bin/clang -o hello.wasm hello.c

# Optimize for size
$WASI_SDK_PATH/bin/clang -Os -o hello.wasm hello.c

# Multiple source files
$WASI_SDK_PATH/bin/clang -o myapp.wasm main.c utils.c -lm
```

A basic program compiles unchanged:

```c
#include <stdio.h>

int main(void) {
    printf("hello from wasm\n");
    return 0;
}
```

---

## Exporting functions

By default, only `main` (via `_start`) is exported. To export additional functions:

```c
// Mark as exported with a specific name
__attribute__((export_name("add")))
int add(int a, int b) {
    return a + b;
}
```

Or use a linker flag:

```bash
clang -Wl,--export=add -o lib.wasm lib.c
```

To build a library with no `main` (just exported functions):

```bash
$WASI_SDK_PATH/bin/clang \
    -nostdlib \
    -Wl,--no-entry \
    -Wl,--export=add \
    -o lib.wasm lib.c
```

---

## Importing host functions

Declare the import with visibility attributes:

```c
// Declare a host-provided function
__attribute__((import_module("env"), import_name("log_value")))
extern void log_value(int x);

void do_work(void) {
    log_value(42);
}
```

In the compiled Wasm, `log_value` appears as an import that the host must satisfy.

---

## The C ABI for wasm32

C types map to Wasm types:

| C type | Wasm type |
|--------|-----------|
| `int`, `unsigned int` | `i32` |
| `long long`, `uint64_t` | `i64` |
| `float` | `f32` |
| `double` | `f64` |
| Any pointer (`void*`, `char*`, ...) | `i32` (32-bit address space) |
| Small struct (≤ 2 scalar fields) | packed into i32/i64 |
| Large struct | pointer to caller-allocated space |

Pointers are `i32` offsets into the module's linear memory. A pointer that crosses the host/module boundary is just an integer — the host uses it to read or write the module's memory at that offset.

---

## Emscripten

Emscripten targets `wasm32-unknown-emscripten` and generates JavaScript glue alongside the `.wasm`. It is the standard choice for browser deployment.

```bash
# Generates hello.html, hello.js, hello.wasm
emcc -o hello.html hello.c

# Just the JS and wasm
emcc -o hello.js hello.c

# Export specific functions
emcc -s EXPORTED_FUNCTIONS='["_add", "_malloc", "_free"]' -o lib.js lib.c
```

Emscripten provides a full POSIX-like environment: `printf` goes to `console.log`, `fopen` is emulated with JS file APIs. For non-browser targets, use wasi-sdk instead.

---

## Inspecting the output

```bash
# Show WAT (text format)
wasm2wat hello.wasm

# Show imports and exports
wasm-objdump -x hello.wasm

# Optimize after compilation (binaryen)
wasm-opt -Os -o hello.opt.wasm hello.wasm
```

`wasm-opt` from [binaryen](https://github.com/WebAssembly/binaryen) often reduces binary size and improves performance beyond what clang produces.

---

Previous: [07 - Imports and Exports](07-Imports-and-Exports) | Next: [09 - Go to WebAssembly](09-Go-to-WebAssembly)
