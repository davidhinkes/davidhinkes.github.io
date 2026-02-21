# Go to WebAssembly

Go has supported WebAssembly since 1.11. There are two distinct targets with different use cases, plus TinyGo as an alternative compiler with different tradeoffs.

---

## Two Go Wasm targets

| `GOOS` / `GOARCH` | Use case | Notes |
|-------------------|----------|-------|
| `GOOS=wasip1 GOARCH=wasm` | WASI runtimes (server-side) | Stable since Go 1.21 |
| `GOOS=js GOARCH=wasm` | Browser | Requires `wasm_exec.js` |

---

## GOOS=wasip1 (server-side)

The `wasip1` target compiles Go programs to WASI-compatible Wasm. The Go runtime implements WASI syscalls, so the standard library (`fmt`, `os`, `net/http` basics) mostly works without modification.

```bash
GOOS=wasip1 GOARCH=wasm go build -o hello.wasm ./cmd/hello

# Run with wasmtime
wasmtime hello.wasm

# Run Go tests as Wasm
GOARCH=wasm GOOS=wasip1 go test ./...
```

A standard Go program compiles unchanged:

```go
package main

import "fmt"

func main() {
    fmt.Println("hello from wasm")
}
```

---

## Exporting functions with wasip1

The `//go:wasmexport` directive (added in Go 1.24) exports a Go function so the host can call it directly:

```go
//go:wasmexport add
func add(a, b int32) int32 {
    return a + b
}
```

The function must use Wasm-compatible types: `int32`, `uint32`, `int64`, `uint64`, `float32`, `float64`, and pointers/unsafe.Pointer.

Prior to Go 1.24, use TinyGo for library-style Wasm (many exported functions, called from a host).

---

## GOOS=js (browser)

The `js` target integrates with the browser's WebAssembly API via `syscall/js`. It provides callbacks, DOM access, and JS interop.

```go
package main

import "syscall/js"

func add(this js.Value, args []js.Value) any {
    return args[0].Int() + args[1].Int()
}

func main() {
    js.Global().Set("goAdd", js.FuncOf(add))
    select {} // keep goroutines alive
}
```

```bash
GOOS=js GOARCH=wasm go build -o main.wasm .

# Copy the JS support file (ships with Go)
cp "$(go env GOROOT)/misc/wasm/wasm_exec.js" .
```

Include `wasm_exec.js` before instantiating the Wasm module. It initializes the Go runtime and bridges Go/JS.

---

## TinyGo

TinyGo is an alternative Go compiler targeting embedded systems and Wasm. It produces much smaller binaries — often 10-100× smaller — because it does not include the full Go runtime.

```bash
tinygo build -o hello.wasm -target wasi ./hello.go
tinygo build -o lib.wasm -target wasm-unknown -no-debug ./lib.go
```

Tradeoffs:

| Feature | Go toolchain | TinyGo |
|---------|-------------|--------|
| Binary size | Large (1+ MB typically) | Small (tens of KB) |
| Goroutines | Full support | Cooperative (limited) |
| Reflection | Full | Limited |
| `//export` | Go 1.24+ (`//go:wasmexport`) | Always supported |
| Standard library | Complete | Partial |
| GC | Full | Conservative (or none) |

For library use (many exported functions called from a host), TinyGo is often the practical choice, especially when binary size matters.

---

## Running wasip1 locally

```bash
# wasmtime (Rust-based, widely used)
wasmtime hello.wasm

# wazero (Go-native, embeddable, no CGo)
go run github.com/tetratelabs/wazero/cmd/wazero@latest run hello.wasm
```

wazero is particularly useful when embedding a Wasm runtime inside a Go program — it is pure Go with no CGo dependency.

---

## Embedding wazero in a Go host

```go
import (
    "context"
    "os"

    "github.com/tetratelabs/wazero"
    "github.com/tetratelabs/wazero/imports/wasi_snapshot_preview1"
)

func main() {
    ctx := context.Background()
    r := wazero.NewRuntime(ctx)
    defer r.Close(ctx)

    // Provide WASI imports
    wasi_snapshot_preview1.MustInstantiate(ctx, r)

    wasmBytes, _ := os.ReadFile("hello.wasm")
    mod, _ := r.InstantiateWithConfig(ctx, wasmBytes,
        wazero.NewModuleConfig().WithStdout(os.Stdout))
    defer mod.Close(ctx)

    // Call an exported function
    add := mod.ExportedFunction("add")
    result, _ := add.Call(ctx, 2, 3)
    fmt.Println(result[0]) // 5
}
```

---

Previous: [08 - C to WebAssembly](08-C-to-WebAssembly) | Next: [10 - WASI](10-WASI)
