# Comparison with Similar Technologies

WebAssembly is not the first portable bytecode format, and understanding how it differs from predecessors clarifies why it was designed the way it was.

---

## Java Bytecode (JVM)

Java bytecode is the closest historical parallel. Both are typed, portable bytecode formats for stack machines.

| Property | JVM bytecode | WebAssembly |
|----------|-------------|-------------|
| Stack machine | Yes | Yes |
| Type system | Rich (classes, interfaces, generics erased) | Minimal (i32, i64, f32, f64, v128, funcref, externref) |
| Object model | Built-in (heap, GC, vtables) | None — memory is a flat byte array |
| Garbage collection | Mandatory (part of the spec) | Optional (WasmGC proposal; off by default) |
| Null references | Yes (historically `NullPointerException`) | No — references are explicit proposals |
| Sandbox | Class loader boundaries; weaker isolation | Hard sandbox; no implicit host access |
| Host interop | JNI (complex, unsafe) | Import/export (simple, explicit) |
| Verification | Single-pass type check | Single-pass validation (faster) |
| Target | Originally browser-via-applets, then server | Browser + server + edge + embedded |

The deepest difference is the object model. The JVM bakes in a heap with garbage collection, object identity, and class hierarchies. Wasm has none of this — it is closer to a typed, portable assembly language. A JVM program cannot exist without a GC; a Wasm module can be compiled from C with `malloc`/`free` and have no GC at all.

JVM bytecode also carries significantly more metadata: constant pools, class hierarchies, debug info. A `.wasm` file is leaner and faster to validate.

---

## CLR / CIL (.NET)

.NET's Common Intermediate Language (CIL) is another typed stack-based bytecode, standardized as ECMA-335.

| Property | CIL | WebAssembly |
|----------|-----|-------------|
| Type system | Very rich (generics, value types, boxing) | Minimal |
| Object model | Built-in (managed heap, GC) | None by default |
| Pointer arithmetic | Unsafe allowed (unsafe keyword) | Linear memory; no host pointers |
| Sandbox | AppDomain (deprecated) / limited trust | Hard sandbox by design |
| Interop | P/Invoke, COM interop | Import/export only |
| Primary runtime | CoreCLR / Mono | wasmtime, wazero, browser, etc. |

CIL and JVM bytecode share the same fundamental tradeoff: they bake in a rich type system and runtime model, which makes them productive for managed languages but heavyweight for systems languages and sandboxing.

Wasm's minimalism is intentional. By leaving GC, object models, and exception handling out of the core spec, it becomes a stable target that many runtimes can implement without requiring a large, fixed runtime.

---

## LLVM IR

LLVM IR is the intermediate representation used inside the LLVM compiler infrastructure. It is what clang produces from C before emitting machine code.

| Property | LLVM IR | WebAssembly |
|----------|---------|-------------|
| Purpose | Compiler IR, not a deployment format | Deployment format |
| Portability | Not portable across versions | Stable, versioned spec |
| Type system | SSA-form, pointer types, LLVM-specific | Simple value types |
| Execution | JIT or AOT via LLVM backends | JIT or AOT via any runtime |
| Human readable | Yes (`.ll` files) | Yes (`.wat` files) |
| Designed for distribution | No | Yes |

LLVM IR is designed to be manipulated by compiler passes, not shipped to end users. It has no stability guarantee across LLVM versions, exposes pointer arithmetic directly, and has no sandboxing properties. Wasm is what you distribute; LLVM IR is how you produce it. Clang compiles C → LLVM IR → WebAssembly.

---

## asm.js

asm.js was Wasm's direct predecessor. It is a strict subset of JavaScript that Mozilla designed in 2013 to allow ahead-of-time compilation in browsers.

| Property | asm.js | WebAssembly |
|----------|--------|-------------|
| Format | Text (JavaScript) | Binary |
| Parse speed | Slow (JS parser) | Fast (compact binary) |
| Type annotations | Implicit (coercions like `x|0`) | Explicit |
| Validation | Optional (engines could ignore it) | Mandatory |
| Toolchain | Emscripten | Emscripten, clang, Go, Rust, etc. |
| SIMD / threads | No | Yes (proposals) |

asm.js demonstrated that near-native performance was achievable in browsers, but it was awkward: type information was encoded as arithmetic coercions, files were large, and parse time was significant. WebAssembly replaced it with a proper binary format that decodes faster than asm.js can parse, with explicit types and mandatory validation. Emscripten can still emit asm.js for legacy targets but defaults to Wasm.

---

## Google Native Client (NaCl / PNaCl)

Google Native Client (2011) was an earlier attempt to run native code safely in the browser. Portable Native Client (PNaCl) used LLVM bitcode as the distribution format.

| Property | PNaCl | WebAssembly |
|----------|-------|-------------|
| Format | LLVM bitcode | Custom binary |
| Portability | Tied to LLVM ABI | Independent |
| Validation | Complex (NaCl's SFI model) | Simple single-pass |
| Browser support | Chrome only | All major browsers |
| Standard | Proprietary Google spec | W3C standard |
| Status | Deprecated (2017) | Active |

NaCl's sandbox relied on Software Fault Isolation, which required a complex validator and was difficult to implement correctly. WebAssembly's structured control flow and explicit type system make validation straightforward and the specification simple enough for multiple independent implementations.

---

## Summary

| Technology | Era | GC required | Sandbox | Primary use today |
|------------|-----|-------------|---------|-------------------|
| JVM bytecode | 1995 | Yes | Moderate | Server, Android |
| CLR / CIL | 2002 | Yes | Moderate | Server, desktop, mobile |
| LLVM IR | 2003 | No | None | Compiler pipeline |
| asm.js | 2013 | No | JS sandbox | Legacy (superseded) |
| NaCl / PNaCl | 2011 | No | SFI | Deprecated |
| **WebAssembly** | **2017** | **Optional** | **Hard** | **Browser, server, edge, plugins** |

WebAssembly occupies a niche that nothing before it filled well: a **portable, sandboxed bytecode with no mandatory runtime model**, suitable for both systems languages (C, C++, Rust) and managed languages (Go, Swift, Kotlin via WasmGC). Its strict sandbox and simple validation make it viable for untrusted plugins — something the JVM and CLR were never designed for.

---

Previous: [12 - Tables and Indirect Calls](12-Tables-and-Indirect-Calls)
