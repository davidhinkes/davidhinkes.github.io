# Passing Data Across the Boundary

WebAssembly functions can only pass value types (i32, i64, f32, f64) across the host/module boundary. To pass richer data — strings, arrays, structs — both sides must agree on a layout in **linear memory**.

---

## The fundamental pattern

1. Host writes input data into the module's linear memory
2. Host calls an exported function, passing a pointer (i32) and length
3. Module reads the data, processes it, writes the result back
4. Host reads the result from memory

```
Host                             Module linear memory
                                 ┌────────────────────────────┐
"hello world"  ──memcopy──►      │ 68 65 6c 6c 6f 20 77 6f ...│
                                 └────────────────────────────┘
                                               ↑ ptr=1024

host calls: process(ptr=1024, len=11)
```

The host accesses the module's memory through the exported `memory` object.

---

## Making memory accessible

The module must export its memory for the host to access it:

```wat
(memory (export "memory") 1)
```

Or the host may provide memory as an import. Either way, both sides see the same bytes at the same addresses.

---

## C example: passing a string

**Module side (C with wasi-sdk):**

```c
#include <stdint.h>
#include <string.h>

static char input_buf[4096];
static char output_buf[4096];

// Host calls this to get the input buffer address
__attribute__((export_name("get_input_ptr")))
uint32_t get_input_ptr(void) {
    return (uint32_t)(uintptr_t)input_buf;
}

// Host writes the string into input_buf, then calls this
__attribute__((export_name("reverse")))
uint32_t reverse(uint32_t len) {
    for (uint32_t i = 0, j = len - 1; i < j; i++, j--) {
        char tmp = input_buf[i];
        input_buf[i] = input_buf[j];
        input_buf[j] = tmp;
    }
    memcpy(output_buf, input_buf, len);
    return len;
}

__attribute__((export_name("get_output_ptr")))
uint32_t get_output_ptr(void) {
    return (uint32_t)(uintptr_t)output_buf;
}
```

**Host side (Go with wazero):**

```go
ctx := context.Background()
mem := mod.Memory()

// Get buffer addresses
inputPtrResult, _ := mod.ExportedFunction("get_input_ptr").Call(ctx)
outputPtrResult, _ := mod.ExportedFunction("get_output_ptr").Call(ctx)
inputPtr := uint32(inputPtrResult[0])
outputPtr := uint32(outputPtrResult[0])

// Write string into module memory
input := "hello world"
mem.Write(inputPtr, []byte(input))

// Call the function
mod.ExportedFunction("reverse").Call(ctx, uint64(len(input)))

// Read result
result, _ := mem.Read(outputPtr, uint32(len(input)))
fmt.Println(string(result)) // "dlrow olleh"
```

---

## Go example: passing data from a Go module

When a Go Wasm module (wasip1) needs to hand data back to the host, use `unsafe.Pointer` to get the address:

```go
//go:build wasip1

package main

import "unsafe"

var buf [4096]byte

//go:wasmexport get_buf_ptr
func getBufPtr() uint32 {
    return uint32(uintptr(unsafe.Pointer(&buf[0])))
}

//go:wasmexport fill
func fill(n uint32) {
    for i := uint32(0); i < n; i++ {
        buf[i] = byte(i % 256)
    }
}
```

The host reads `buf` via the exported memory object, using the address returned by `getBufPtr`.

---

## Allocator-based pattern

A more flexible approach: export `malloc` and `free` so the host can allocate memory dynamically in the module's address space.

**C module:**

```c
#include <stdlib.h>

__attribute__((export_name("alloc")))
void* alloc(size_t n) { return malloc(n); }

__attribute__((export_name("dealloc")))
void dealloc(void* p) { free(p); }
```

**Go host:**

```go
allocFn := mod.ExportedFunction("alloc")
deallocFn := mod.ExportedFunction("dealloc")

// Allocate space in the module for our input
ptrResult, _ := allocFn.Call(ctx, uint64(len(input)))
ptr := uint32(ptrResult[0])

mem.Write(ptr, []byte(input))
// ... call processing function ...

// Free when done
deallocFn.Call(ctx, uint64(ptr))
```

This pattern is common when the module is a library and the host controls the data lifecycle. The module's own allocator manages the memory, so there is no risk of the host and module having conflicting views of which bytes are "owned."

---

## Struct layout

C structs have a well-defined layout on wasm32 (same rules as a 32-bit platform ABI):

```c
// Both host and module must agree on this layout
typedef struct {
    uint32_t x;      // offset 0, 4 bytes
    uint32_t y;      // offset 4, 4 bytes
    float    value;  // offset 8, 4 bytes
} Point;             // 12 bytes total, no padding on wasm32
```

The host writes the struct at the known pointer using the same field offsets. In Go, use `encoding/binary` for portability or `unsafe.Pointer` for zero-copy access.

---

## What the Component Model solves

The manual pointer-passing pattern is tedious and error-prone. The **Component Model** (wasip2) defines a type system that handles strings, lists, records, and variants natively, with generated language bindings via `wit-bindgen`. For wasip1, the pointer pattern described here is what those generators produce under the hood.

---

Previous: [10 - WASI](10-WASI) | Next: [12 - Tables and Indirect Calls](12-Tables-and-Indirect-Calls)
