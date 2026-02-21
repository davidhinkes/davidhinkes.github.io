# Control Flow

WebAssembly has no `goto`. All control flow is **structured**: it uses blocks with explicit entry and exit points. This makes programs straightforward to validate and compile efficiently to native code.

---

## Blocks

A `block` is a sequence of instructions with an optional result type. `br` (break) exits a block from anywhere inside it, jumping to the instruction after `end`.

```wat
block $done (result i32)
  ;; ... instructions ...
  i32.const 42
  br $done          ;; exit block, leaving 42 on stack
  ;; unreachable after br
end
;; 42 is on the stack here
```

Labels are lexically scoped. `br 0` means "break out of the immediately enclosing block." `br 1` breaks out of the block one level up. Named labels (like `$done`) are syntactic sugar for these numeric depths.

---

## Loop

A `loop` is a block where `br` goes to the **top** (re-enters) instead of the bottom (exits).

```wat
(local $i i32)
i32.const 0
local.set $i

loop $again
  local.get $i
  i32.const 10
  i32.lt_s
  if
    ;; loop body
    local.get $i
    i32.const 1
    i32.add
    local.set $i
    br $again       ;; jump back to top of loop
  end
end
```

The pattern: `loop` provides the re-entry label; `br` to the loop label jumps back to the top; `br` to an outer `block` exits.

---

## if / else

```wat
local.get $condition    ;; push i32 (0 = false, nonzero = true)
if (result i32)
  i32.const 1
else
  i32.const 0
end
;; result (0 or 1) is on stack
```

`if` pops an i32 from the stack. Nonzero → take the if branch. The `else` clause is optional. Both branches must leave the stack in the same state.

---

## br_if

Conditional break — pops a condition, breaks if nonzero:

```wat
local.get $done
br_if $exit         ;; if $done != 0, break to $exit
;; otherwise, fall through
```

This is the standard way to implement `while` and `for` loops. The typical pattern is: `loop` + `br_if` to exit (jumping to an enclosing `block`) + `br` to continue (jumping back to the loop label).

---

## br_table

Dispatch to one of N targets based on an index — Wasm's `switch`:

```wat
local.get $idx
br_table $case0 $case1 $case2 $default
```

If `$idx` is 0 → jump to `$case0`, 1 → `$case1`, 2 → `$case2`. Out-of-range → `$default`. The last label is always the default.

---

## Traps

Some instructions terminate the instance immediately with a **trap** — an unrecoverable error:

- Integer divide by zero (`i32.div_s` or `i32.div_u` with divisor 0)
- Out-of-bounds memory access
- `unreachable` instruction (like `__builtin_unreachable` or `panic`)
- `i32.trunc_f64_s` with a value that doesn't fit in i32

A trap is not catchable from within the Wasm module. The host runtime receives it. In Go's wazero, a trap returns an error from the call. In browsers, it throws a `WebAssembly.RuntimeError`.

---

## How C control flow maps to Wasm

The C compiler transforms all control flow into structured form:

| C | Wasm |
|---|------|
| `if / else` | `if / else / end` |
| `while (cond) { ... }` | `block` + `loop` + `br_if` (to exit) + `br` (to loop) |
| `for (init; cond; post)` | same |
| `break` | `br` to enclosing `block` |
| `continue` | `br` to enclosing `loop` |
| `switch` | `br_table` (for dense cases) or chain of `if/else` |
| `return` | `return` instruction |

The structured control flow requirement is one reason function pointers and virtual dispatch require **tables** — see [12 - Tables and Indirect Calls](12-Tables-and-Indirect-Calls).

---

Previous: [04 - The Stack Machine](04-The-Stack-Machine) | Next: [06 - Linear Memory](06-Linear-Memory)
