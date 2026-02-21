# The Stack Machine

Every WebAssembly function executes against an implicit **operand stack**. Instructions pop their inputs from the stack and push their outputs. There are no named registers.

---

## Instruction execution

```
i32.add pops two i32 values, pushes their sum.

Before:   [ ... | 3 | 7 ]
                      ↑ top
After:    [ ... | 10 ]
```

The validator tracks the stack state at every instruction statically. If an instruction would pop from an empty stack, or types don't match, validation fails before the module runs. A valid module cannot crash due to a type error at runtime.

---

## Locals

Each function declares **locals**: typed, mutable, zero-initialized variables. Parameters are locals — the first N locals are the parameters.

```wat
(func $example (param $x i32) (param $y i32) (result i32)
  (local $tmp i32)       ;; extra local, initialized to 0

  local.get $x           ;; push x
  local.get $y           ;; push y
  i32.add                ;; push x+y
  local.tee $tmp         ;; store in $tmp, keep on stack
  drop                   ;; discard (just showing tee)
  local.get $tmp         ;; push $tmp
)
```

| Instruction | Effect |
|-------------|--------|
| `local.get $n` | Push the value of local n onto the stack |
| `local.set $n` | Pop a value, store in local n |
| `local.tee $n` | Store in local n AND leave the value on the stack |

Locals are **not** memory. They live in the activation frame and are not addressable — you cannot take a pointer to a local. In C, a local variable that has its address taken (`&x`) ends up in linear memory, not in a Wasm local.

---

## Globals

Module-level variables, accessible from any function. Can be mutable or immutable.

```wat
(global $counter (mut i32) (i32.const 0))

(func $increment
  global.get $counter
  i32.const 1
  i32.add
  global.set $counter)
```

Globals can be imported from or exported to the host.

---

## Common instructions

```
Constants:
  i32.const 42        push i32 value 42
  f64.const 3.14      push f64 value 3.14

Arithmetic:
  i32.add / i32.sub / i32.mul / i32.div_s / i32.rem_s
  i64.add / i64.mul / ...
  f32.add / f32.sqrt / f32.floor / ...
  f64.add / f64.sqrt / ...

Comparison (push i32 result: 0 = false, 1 = true):
  i32.eq / i32.ne / i32.lt_s / i32.gt_u / i32.le_s / ...
  f64.lt / f64.ge / ...

Conversion:
  i32.wrap_i64        truncate i64 to i32
  i64.extend_i32_s    sign-extend i32 to i64
  f64.convert_i32_s   signed i32 → f64
  i32.trunc_f64_s     f64 → i32 (traps if out of range or NaN)

Stack manipulation:
  drop                discard top of stack
  select              pop condition, pop two values, push one of them
```

---

## Call and return

```wat
call $fn            ;; pop args from stack matching $fn's signature, push results
return              ;; exit function; top of stack must match declared result types
```

At function entry, the operand stack (from the caller's perspective) contains the arguments. At function exit, the stack must contain exactly the declared result types.

---

## Validation guarantees

The static stack discipline means every instruction can be type-checked before execution. After a module passes validation:

- No stack underflow can ever occur
- No type mismatches can occur at runtime
- The operand stack depth is bounded at every point

This is why Wasm can be compiled ahead-of-time to native machine code and trusted to run correctly on any conforming runtime.

---

Previous: [03 - Value Types](03-Value-Types) | Next: [05 - Control Flow](05-Control-Flow)
