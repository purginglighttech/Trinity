# MK-Ultra — Error Handling Model

**Status:** draft v0.1 · **Decision locked 2026-09-24:** Zig-style error unions.
**Applies to:** Maia and Koda identically (seamless interop).

## The three categories

Failure in MK-Ultra is three distinct things with three distinct mechanisms.
They do not mix.

| Category | Meaning | Mechanism |
|---|---|---|
| **Absence** | a value may not exist | `?T` (Option): `??`, `if x \|v\| then`, `match` |
| **Recoverable failure** | an operation can fail | `T ! E` (error union): `try`, `catch` |
| **Bugs** | unrecoverable programmer error | `panic(...)` — aborts |

`??` is absence-only. `catch` is failure-only. This split is deliberate:
absence and failure are different concepts and get different operators.

## Error sets

Errors are codes grouped into named sets, declared like enums:

```pascal
type IOError :: error
    NotFound
    PermissionDenied
    ConnectionReset
end
```

- An error set is a distinct type. Error values are immutable and default
  to the `val` capability (freely shareable, sendable across actors).
- **Subset coercion:** `error{A} ! T` coerces to `error{A, B} ! T`. A function
  may propagate a narrower error set through a wider one; the reverse is a
  compile error.
- v0.1 has named sets only — no open `anyerror`, no payload-carrying errors
  (see Future work).

## The error union type: `T ! E`

```pascal
function open_file(path: string): File ! IOError
```

`T ! E` reads "a `T`, or an error from set `E`". It is a type-level operator
(the lexer tokenizes `!=` greedily; a lone `!` is valid only in type position).
It composes with the rest of the type system: `?Config ! IOError`,
`val (Config ! IOError)`.

## Propagation: `try`

```pascal
function load_config(path: string): Config ! IOError
begin
    var file := try open_file(path)
    defer file.close()

    var text := try file.read_all()
    return parse_config(text)
end
```

`try expr`:
- If `expr` evaluates to success, yields the `T`.
- If `expr` evaluates to an error, the error is **returned from the enclosing
  function immediately** (all `defer`s run first).

Rules (all compile-time):
1. The enclosing function must return an error union. `try` in a function
   that doesn't is a compile error.
2. The error set of `expr` must coerce to the function's error set
   (subset-to-superset). Otherwise a compile error.
3. `try` applied to a non-error-union value is a compile error.
4. `try` inside a `defer` or `errdefer` body is a compile error — there is
   nowhere to propagate to. Use `catch` there.
5. `catch` binds tighter than `try`: `try f() catch d` parses as
   `try (f() catch d)`, and since the inner expression is a plain `T`,
   rule 3 makes it a compile error with a suggestion to drop `try`.
   Pick one: propagate (`try`) or handle (`catch`).

## Local handling: `catch`

```pascal
var file := open_file(path) catch File.stderr
var file2 := open_file(path) catch |err| begin_fallback(err)
```

`expr catch fallback`:
- If `expr` is success, yields the `T`.
- If `expr` is an error, evaluates `fallback`, which must produce a `T`.
- `catch |err|` binds the error value for the fallback expression.
- The fallback is a single expression. Multi-statement recovery belongs in
  a named function.

## Cleanup: `defer` and `errdefer`

- `defer expr` runs on **every** exit path: normal return and error
  propagation. (Behavior on `panic` is backend-defined.)
- `errdefer expr` runs **only** when the function exits via error
  propagation — for rolling back partial work that shouldn't be undone
  on success:

```pascal
function create_pair(): Pair ! AllocError
begin
    var a := try allocate()
    errdefer a.free()

    var b := try allocate()   (* if this fails, a.free() runs *)
    return Pair(.a := a, .b := b)
end
```

## Panics

`panic(message)` is a prelude builtin for unrecoverable bugs — failed
invariants, unreachable code paths. It aborts; it is not catchable and never
part of a function's type. (Formally it returns the `Never` type; the
`Never` type itself is future work.)

## Capabilities

- Error values default to `val`: immutable, shareable, sendable. An error
  crossing an actor boundary is always safe.
- `try`/`catch` preserve the capability of the success payload.

## `main` and errors

A program entry point may return an error union:

```pascal
function main(): void ! AppError
begin
    try run()
end
```

The runtime prints the error and exits nonzero. (Plain `procedure main()`
remains valid.)

## Koda parity

Koda uses the identical model: same `T ! E` types, same `try`/`catch`,
same `defer`/`errdefer`. A Maia function returning `T ! E` is callable
from Koda with the same handling, and vice versa.

## Grammar deltas

See the Maia grammar doc. Summary: `!` type operator (`union_type`);
`error` type constructor; `try`/`catch` in the expression chain
(`catch` tighter than `try`); `errdefer` statement; `error`,
`catch`, `errdefer`, `try` are reserved words; `panic` is a prelude
builtin, not a keyword.

## Future work (explicitly out of v0.1)

- Payload-carrying errors (rich error values beyond codes).
- Open `anyerror` type.
- `match` on error unions (needs typed patterns).
- Block-valued `catch` targets (needs block expressions).
- The `Never` type and its full semantics.
- `defer` behavior on `panic`, per backend.
