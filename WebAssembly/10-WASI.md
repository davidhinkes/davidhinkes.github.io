# WASI

WASI (WebAssembly System Interface) is a set of import functions that give Wasm modules access to OS capabilities: file I/O, environment variables, clocks, and sockets. It is the mechanism that makes Wasm useful outside the browser.

---

## The model

Without WASI, a Wasm module is completely isolated — it cannot read a file, print to stdout, or get the current time. WASI defines standard import names that runtimes implement:

```
Wasm module  calls "wasi_snapshot_preview1" "fd_write"
       ↓
Runtime (wasmtime / wazero / wasmer)
  implements fd_write using the host OS
       ↓
Host OS (Linux / macOS / Windows)
```

The module is portable. The runtime adapts WASI calls to the host OS. The module never knows which OS it runs on.

---

## wasip1 — the current standard

**wasip1** (also called `wasi_snapshot_preview1`) is the first stable WASI specification. It is what `GOOS=wasip1` targets in Go and what wasi-sdk targets in C.

Key functions:

| Function | What it does |
|----------|-------------|
| `fd_write` | Write bytes to a file descriptor |
| `fd_read` | Read bytes from a file descriptor |
| `fd_close` | Close a file descriptor |
| `path_open` | Open a file (requires a preopened directory) |
| `path_create_directory` | Create a directory |
| `clock_time_get` | Get current time |
| `environ_get` | Get environment variables |
| `args_get` | Get command-line arguments |
| `proc_exit` | Exit with a status code |

File descriptors 0, 1, 2 are stdin, stdout, stderr — same as POSIX.

---

## Preopened directories

WASI is capability-based: a module can only access directories that the host explicitly grants. There is no "open any path" — the module must be given a **preopened directory** at startup.

```bash
# Grant access to /tmp
wasmtime --dir /tmp myapp.wasm

# Grant access to current directory, mapped as "."
wasmtime --dir . myapp.wasm

# Grant access to /data, but the module sees it as "/data"
wasmtime --dir /data myapp.wasm
```

In wazero (Go):

```go
wazero.NewModuleConfig().
    WithFS(os.DirFS("/data"))
```

A module that tries to open a path outside its granted directories receives a permission error — the capability is not available, not just denied.

---

## fd_write in detail

`fd_write` is how `printf` / `fmt.Println` write to stdout. Its WAT signature:

```wat
(import "wasi_snapshot_preview1" "fd_write"
  (func (param i32 i32 i32 i32) (result i32)))
```

Parameters:
1. `fd` (i32) — file descriptor (1 = stdout, 2 = stderr)
2. `iovs` (i32) — pointer to an array of `{buf_ptr i32, buf_len i32}` structs in memory
3. `iovs_len` (i32) — number of iov pairs
4. `nwritten` (i32) — pointer where the number of bytes written is stored

Returns 0 on success, or an errno value. The iovec design (scatter-gather I/O) mirrors POSIX `writev`.

---

## WASI in C

With wasi-sdk, standard C I/O functions work without changes. The libc translates them to WASI calls:

```c
#include <stdio.h>
#include <time.h>

int main(void) {
    // These use WASI internally
    printf("hello\n");               // → fd_write

    time_t t = time(NULL);           // → clock_time_get
    printf("time: %ld\n", t);

    FILE *f = fopen("data.txt", "r"); // → path_open (needs preopened dir)
    return 0;
}
```

---

## WASI in Go

With `GOOS=wasip1`, the Go runtime implements WASI:

```go
package main

import (
    "fmt"
    "os"
    "time"
)

func main() {
    fmt.Println("hello")          // → fd_write
    fmt.Println(time.Now())       // → clock_time_get
    f, _ := os.Open("data.txt")  // → path_open (needs --dir)
    _ = f
}
```

---

## wasip2 and the Component Model

**wasip2** is the next generation, built on the **Component Model** — a higher-level type system that supports strings, records, variants, and resource handles natively. It eliminates manual pointer-passing for complex types and provides language-agnostic interface definitions via WIT (WebAssembly Interface Types).

wasip2 is not yet widely supported by language toolchains (as of 2025), but wasmtime supports it and Go, Rust, and C toolchains are adding support. It is the direction the ecosystem is heading.

---

## Runtimes

| Runtime | Language | Best for |
|---------|----------|----------|
| wasmtime | Rust | Reference implementation; Rust, C, and Python embedding |
| wasmer | Rust | Multi-backend (Cranelift, LLVM, Singlepass) |
| wazero | Go | Embedding in Go programs; pure Go, no CGo |
| WasmEdge | C++ | Cloud-native and edge deployments |

For Go programs that embed a Wasm runtime, **wazero** is the idiomatic choice: pure Go, no CGo dependency, MIT license.

---

Previous: [09 - Go to WebAssembly](09-Go-to-WebAssembly) | Next: [11 - Passing Data Across the Boundary](11-Passing-Data)
