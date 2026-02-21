# What is WebAssembly

WebAssembly (Wasm) is a binary instruction format for a stack-based virtual machine. It is designed to be fast to decode, close to native execution speed, and completely sandboxed.

The name is slightly misleading: WebAssembly is not assembly, and it is not just for the web. It is a portable bytecode format that any host can embed and execute — browsers, servers, edge functions, plugin systems, and standalone runtimes all use it.

---

## The core model

A WebAssembly module is a `.wasm` file containing type definitions, function bytecode, memory declarations, imports, and exports. When a host loads a module, it creates an **instance**: a running copy with its own linear memory and mutable state.

```
[C source]          [Go source]
    ↓ clang               ↓ go build
[.wasm module]      [.wasm module]
        ↘               ↙
         [host runtime]
               ↕
          [instance]
               ↕
        [linear memory]
```

The host provides **imports** — functions the module can call — and the module provides **exports** — functions and memory the host can call or access.

---

## Stack machine, not register machine

Most real CPUs are register machines: instructions name specific registers (`mov rax, rbx`). WebAssembly is a **stack machine**: instructions implicitly pop their inputs and push their outputs onto an operand stack.

```wat
;; Compute (a + b) * c
local.get $a    ;; stack: [a]
local.get $b    ;; stack: [a, b]
i32.add         ;; pops a, b → pushes (a+b)    stack: [a+b]
local.get $c    ;; stack: [a+b, c]
i32.mul         ;; pops a+b, c → pushes result  stack: [(a+b)*c]
```

Stack machines are simple to specify and validate, which is why Wasm can guarantee type-safe execution without sacrificing performance.

---

## The sandbox

A WebAssembly instance is strictly isolated:

- It can only access its own **linear memory** — no pointers into host memory
- It can only call **imported functions** explicitly provided by the host
- It cannot spawn threads, open files, or make syscalls on its own
- All control flow is **structured** — no arbitrary jumps

This is why Wasm is used for untrusted plugins and edge compute: you can run arbitrary third-party code safely.

---

## Why it exists

| Goal | How Wasm achieves it |
|------|---------------------|
| Near-native speed | Simple instruction set, straightforward compilation to machine code |
| Portable | Binary format is OS- and architecture-independent |
| Safe | Sandboxed memory, no undefined behavior by design |
| Compact | Binary encoding is typically smaller than equivalent native binaries |
| Fast to load | Linear format, no relocation, validates in a single forward pass |

---

## Not just for browsers

Originally Wasm ran only in browsers (as a safer, faster replacement for asm.js). WASI (WebAssembly System Interface) extended it to server-side by defining a set of import functions that provide OS capabilities — file I/O, clocks, sockets. Today Wasm runs in:

- **Browsers** — via the JS `WebAssembly` API
- **Standalone runtimes** — wasmtime, wasmer, wazero (Go-native)
- **Edge runtimes** — Cloudflare Workers, Fastly Compute
- **Plugin systems** — Envoy proxy filters, Extism, any host that embeds a runtime

Both C and Go compile to Wasm. The rest of this course explains how.

---

Next: [02 - Modules](02-Modules)
