# MK-Ultra Maia — Formal Grammar (EBNF)

**Status:** draft v0.1.1 · **Scope:** Maia (compiled sub-language) only; Koda gets its own grammar.
**Locked decisions:** Pascal-style syntax (`begin`/`end`, no braces); Pony-style reference
capabilities only (`iso`, `trn`, `ref`, `val`, `box`, `tag`); Rust-style ownership qualifiers removed.

## Notation

```
=           definition
|           alternation
[...]       optional
{...}       zero or more repetitions
(...)       grouping
"..."       terminal (keyword, operator, delimiter)
(* ... *)   comment
```

## Lexical notes (see spec §2 for full token definitions)

- Source is UTF-8. Identifiers, keywords, literals, and comments are per spec §2.3–2.6.
- **NEWLINE terminates statements.** There are no semicolons. A `\` at end of line
  continues a statement onto the next line. Indentation is insignificant.
- `NEWLINE` below means one or more newline tokens.
- `!` vs `!=`: the lexer tokenizes `!=` greedily. A lone `!` is the
  error-union type operator, valid only in type position; in expression
  position it is a syntax error.

---

## Program structure

```ebnf
program          = { top_level_decl } ;

top_level_decl   = [ export_prefix ] decl_body ;
decl_body        = module_decl
                 | mod_decl
                 | use_decl
                 | const_decl
                 | var_decl
                 | function_decl
                 | procedure_decl
                 | type_decl
                 | actor_decl
                 | extern_decl ;

export_prefix    = "export" [ "(" "module" ")" ]
                 | "package" ;

module_decl      = "module" identifier NEWLINE ;
mod_decl         = "mod" identifier "::" module_path NEWLINE ;
                   (* binds a local name to a module for qualified access:
                      mod std::mod/std/std → std.sqrt.
                      The :: is the same binding operator used by var/const/type.
                      Contrast use, which binds selected names into the
                      current scope. *)
use_decl         = "use" module_path [ ".(" identifier { "," identifier } ")" ]
                   [ ".*" ] [ "as" identifier ] NEWLINE ;
module_path      = identifier { "/" identifier } ;
```

## Declarations

```ebnf
const_decl       = "const" identifier [ ":" type_expr ] "::" expr NEWLINE ;

var_decl         = "var" identifier { "," identifier }
                   [ ":" type_expr ] [ "::" expr_list ] NEWLINE ;
expr_list        = expr { "," expr } ;

function_decl    = [ attributes ] [ "async" ] "function" identifier
                   [ generic_params ] "(" [ param_list ] ")"
                   [ ":" type_expr ] ( block | NEWLINE ) ;
                   (* bodyless form is for extern declarations *)

procedure_decl   = [ attributes ] "procedure" identifier
                   [ generic_params ] "(" [ param_list ] ")" block ;

param_list       = param { "," param } ;
param            = identifier [ ":" type_expr ] ;

generic_params   = "[" generic_param { "," generic_param } "]" ;
generic_param    = identifier [ ":" type_expr ] ;

type_decl        = "type" identifier "::" type_expr NEWLINE
                 | "type" identifier "::" struct_def
                 | "type" identifier "::" enum_def
                 | "type" identifier "::" error_def ;

error_def        = "error" NEWLINE { identifier NEWLINE } "end" ;

struct_def       = "struct" NEWLINE { struct_field } "end" NEWLINE ;
struct_field     = identifier ":" type_expr NEWLINE ;

enum_def         = "enum" NEWLINE { enum_variant } "end" NEWLINE ;
enum_variant     = identifier [ "::" expr ] NEWLINE ;

attributes       = { attribute } ;
attribute        = "@" identifier [ "(" [ expr_list ] ")" ] NEWLINE ;
                   (* e.g. @comptime, @link("python3") — NEWLINE only when
                      the attribute stands alone; inline use has no NEWLINE *)

actor_decl       = "actor" identifier NEWLINE { actor_member } "end" NEWLINE ;
actor_member     = [ export_prefix ]
                   ( var_decl | const_decl | behavior_decl
                   | function_decl | procedure_decl ) ;
behavior_decl    = "behavior" identifier "(" [ param_list ] ")" block ;

extern_decl      = [ attributes ] "extern" string_literal
                   ( function_decl | procedure_decl | var_decl ) ;
```

## Types

```ebnf
type_expr        = [ capability ] union_type ;
union_type       = base_type [ "!" base_type ] ;   (* T ! E — error union *)
capability       = "iso" | "trn" | "ref" | "val" | "box" | "tag" ;
                   (* Pony-style reference capabilities; see spec §3.4.
                      Default capability per type is a semantic rule,
                      not a grammar rule — open question below. *)

base_type        = primitive_type
                 | array_type
                 | slice_type
                 | option_type
                 | pointer_type
                 | channel_type
                 | struct_def
                 | enum_def
                 | identifier ;          (* named / generic type application
                                            e.g. array(int32) is identifier
                                            followed by call-style args —
                                            parsed as primary_expr in
                                            type position; see note *)

primitive_type   = "int8" | "int16" | "int32" | "int64"
                 | "uint8" | "uint16" | "uint32" | "uint64"
                 | "f32" | "f64" | "bool" | "string"
                 | "usize" | "isize" ;

array_type       = "[" expr "]" type_expr ;      (* [5]int32, [3][3]f64 *)
slice_type       = "[" "]" type_expr ;           (* []int32 *)
option_type      = "?" type_expr ;               (* ?int32 *)
pointer_type     = "^" type_expr ;               (* ^u8 — Pascal style *)
channel_type     = "channel" "(" type_expr ")" ; (* channel(int32) *)
```

> **Note:** generic type application such as `array(int32)` parses as an
> identifier followed by a parenthesized argument list in type position.
> Whether this is elaborated as a generic instantiation or a distinct
> nominal type is a semantic (name-resolution) rule.

## Statements

```ebnf
block            = "begin" NEWLINE { statement } "end" ;

statement        = var_decl
                 | const_decl
                 | assignment
                 | if_stmt
                 | while_stmt
                 | for_stmt
                 | loop_stmt
                 | match_stmt
                 | defer_stmt
                 | errdefer_stmt
                 | return_stmt
                 | break_stmt
                 | continue_stmt
                 | block
                 | safe_block
                 | unsafe_block
                 | asm_stmt
                 | expr_stmt ;

assignment       = assign_target assign_op expr NEWLINE ;
assign_target    = identifier { ( "." identifier | "[" expr "]" ) } ;
assign_op        = ":=" | "+=" | "-=" | "*=" | "/=" | "%="
                 | "&=" | "|=" | "^=" | "<<=" | ">>=" ;

if_stmt          = "if" expr [ "|" identifier "|" ] "then" NEWLINE
                   { statement }
                   { "elsif" expr "then" NEWLINE { statement } }
                   [ "else" NEWLINE { statement } ]
                   "end" ;
                   (* [ "|" identifier "|" ] is the Option-unwrap binding:
                      if maybe |value| then ... end *)

while_stmt       = "while" expr "do" NEWLINE { statement } "end" ;

for_stmt         = "for" identifier [ "," identifier ] "in" expr "do"
                   NEWLINE { statement } "end" ;

loop_stmt        = "loop" NEWLINE { statement } "end" ;

match_stmt       = "match" expr NEWLINE { match_arm } "end" ;
match_arm        = match_pattern ":" ( block | statement ) ;
match_pattern    = expr | "else" ;

defer_stmt       = "defer" expr NEWLINE ;
errdefer_stmt    = "errdefer" expr NEWLINE ;       (* runs only on error exit *)
return_stmt      = "return" [ expr ] NEWLINE ;
break_stmt       = "break" NEWLINE ;
continue_stmt    = "continue" NEWLINE ;

safe_block       = "safe" "do" NEWLINE { statement } "end" ;
unsafe_block     = "unsafe" "do" NEWLINE { statement } "end" ;

asm_stmt         = "asm" ( string_literal NEWLINE
                         | "begin" NEWLINE { asm_line } "end" ) ;
asm_line         = (* one line of target-specific assembly text;
                      opaque to the grammar *) NEWLINE ;

expr_stmt        = expr NEWLINE ;
```

## Expressions (precedence, loosest first)

```ebnf
expr             = try_expr ;
try_expr         = [ "try" ] catch_expr ;
catch_expr       = coalesce_expr
                   [ "catch" [ "|" identifier "|" ] coalesce_expr ] ;
coalesce_expr    = or_expr { "??" or_expr } ;              (* null coalescing *)
or_expr          = and_expr { "or" and_expr } ;
and_expr         = not_expr { "and" not_expr } ;
not_expr         = [ "not" ] comparison_expr ;
comparison_expr  = range_expr [ comp_op range_expr ] ;
comp_op          = "==" | "!=" | "<" | ">" | "<=" | ">=" ;
range_expr       = bitor_expr [ range_op bitor_expr ] ;
range_op         = ".." | "..<" ;
bitor_expr       = bitxor_expr { "|" bitxor_expr } ;
bitxor_expr      = bitand_expr { "^" bitand_expr } ;
bitand_expr      = shift_expr { "&" shift_expr } ;
shift_expr       = additive_expr { ( "<<" | ">>" ) additive_expr } ;
additive_expr    = multiplicative_expr { ( "+" | "-" ) multiplicative_expr } ;
multiplicative_expr = unary_expr { ( "*" | "/" | "%" | "//" ) unary_expr } ;
unary_expr       = { unary_op } postfix_expr ;
unary_op         = "+" | "-" | "~" | "&" ;                (* & = address-of *)
postfix_expr     = primary_expr { postfix_op } ;
postfix_op       = call_suffix | index_suffix | field_suffix
                 | deref_suffix | cast_suffix ;
call_suffix      = "(" [ expr_list ] ")" ;
index_suffix     = "[" expr "]" ;                          (* arr[i], arr[0..3] *)
field_suffix     = "." identifier ;
deref_suffix     = "^" ;                                   (* p^ — Pascal style *)
cast_suffix      = "as" type_expr ;

primary_expr     = literal
                 | identifier
                 | struct_literal
                 | array_literal
                 | "(" expr ")"
                 | lambda_expr
                 | await_expr
                 | spawn_expr
                 | consume_expr
                 | builtin_call ;

struct_literal   = identifier "(" [ field_init { "," field_init } ] ")" ;
field_init       = "." identifier "::" expr ;
                   (* Point(.x :: 10, .y :: 20) *)

array_literal    = "[" [ expr_list ] "]" ;                 (* [1, 2, 3] *)

await_expr       = "await" expr ;
spawn_expr       = "spawn" postfix_expr ;
consume_expr     = "consume" expr ;                        (* move semantics *)
builtin_call     = "@" identifier "(" [ expr_list ] ")" ;
                   (* @target_os(), @compile_error("...") *)

lambda_expr      = ( "function" "(" [ param_list ] ")" [ ":" type_expr ]
                   | "procedure" "(" [ param_list ] ")" )
                   block ;
                   (* anonymous function/procedure, Delphi style.
                      Bars are NOT used for lambdas: |x| would clash
                      with the Option-unwrap binding
                      (if maybe |value| then ... end). *)
```

## Literals (terminals — lexical detail per spec §2.5)

```ebnf
literal          = integer_literal | float_literal | string_literal
                 | "true" | "false" | "null" ;
```

---

## Judgment calls (divergences from spec v0.1.0)

1. **Braces removed everywhere.** `struct { }` → `struct … end`; `enum { }` →
   `enum … end`; `match { }` → `match … end`; `asm { }` → `asm begin … end`
   (or a string literal). Per the locked Pascal-style decision.
2. **`use Math.{add, PI}` → `use Math.(add, PI)`.** Selection list uses
   parentheses instead of braces. `use Math.*` is retained (`*` is not a brace).
3. **Struct literal `Point { .x = 10 }` → `Point(.x = 10, .y = 20)`.**
   Parenthesized construction; unambiguous with calls in the grammar.
4. **Pointer type `*u8` → `^u8`.** Spec §2.7 already defines `^` as dereference
   and `&` as address-of (Pascal style); the `*u8` in the FFI example was
   inconsistent with that. Dereference is postfix `p^`.
5. **Ownership qualifiers deleted.** `own`, `ref`-as-borrow, `mut`, `rc`, `weak`
   are gone. `ref` now means only the Pony capability (mutable, shareable,
   aliased, non-sendable). `rc`-style sharing is expressed via capabilities
   (`ref`/`val`) — the exact shared-ownership story is a semantic rule to write.
6. **`//` kept for integer division** (spec's operator, not Pascal's `div`).
7. **Newlines terminate statements; `\` continues.** The spec's examples never
   use semicolons; the grammar makes that rule explicit.
8. **Pipeline `|` omitted.** Spec marks it Koda-only; this grammar is Maia-only.
9. **`::` binds, `=` is gone.** Per your call: `::` replaces `=` everywhere a
   name is bound to a value or definition — `var`, `const`, `type` aliases,
   enum discriminants (`Ok :: 200`), and struct literal fields
   (`Point(.x :: 10, .y :: 20)`). `=` now appears nowhere in the grammar;
   `==` (comparison) and `:=` (assignment) are unaffected.
10. **Default capabilities — Pony mapping (locked).** When a type is written
    without an explicit capability, the compiler fills in:
    primitives and enums → `val`; structs → `ref`; actors → `tag`.
    `var`/`const` governs whether the *binding* can be re-pointed;
    the capability governs what can be done *through* the reference
    (mutate, alias, send). Only `iso`, `val`, and `tag` cross actor
    boundaries — a bare `ref` struct can never cause a data race.
11. **Match arm separator `:` (locked).** `match` arms use Pascal-style `:`
    (`0: writeln("zero")`), not `=>`.
12. **Generic bounds `T: Bound` (locked).** Inline Delphi-style constraints,
    e.g. `function sort<T: Comparable>(items: []T): []T`. No `where`
    clauses in v0.1.
13. **Maia lambdas (locked).** Anonymous functions/procedures use Delphi
    style — `function(req: Request): Response begin ... end` as an
    expression. Needed for seamless Maia/Koda interop (Koda has closures).
    `|x|` syntax rejected: bars already mean Option-unwrap binding.
    Capture semantics follow capability rules (to be specified).
14. **Shared ownership (locked).** Capabilities only — no implicit reference
    counting. Shared mutable state is `ref`-capability aliasing within one
    actor. Actor boundaries require `iso`, `val`, or `tag`. Shared
    ownership with reclamation is a library type on explicit allocators,
    not a language feature.
15. **Move semantics (locked).** Pony semantics: `iso` values move implicitly
    on use (assignment, argument passing) — entailed by what `iso` means.
    `consume x` is the explicit destructive read yielding `iso`.
16. **Inline assembly (locked).** Opaque per-line text inside
    `asm begin … end`, passed through to the backend. A structured
    template syntax (Rust `asm!`-style) is deferred past v0.1.
17. **Error model — Zig-style error unions (locked).** `T ! E` type operator;
    `try` propagates (compile-time checked against the enclosing function's
    error set); `catch` handles locally, `??` stays Option-only;
    `errdefer` runs cleanup on error paths; named error sets via
    `type Name :: error … end` with subset-to-superset coercion; errors
    default to `val`. `panic` is a prelude builtin, not a keyword.
    Full model in `mkultra-error-handling.md`.
18. **`mod` vs `use` (locked).** `mod` binds a local name to a module
    with the `::` binding operator — `mod std::mod/std/std`,
    `mod math::mod/math/math` — for qualified access (`std.sqrt`).
    Module paths are slash-delimited (`/`), in both `mod` and `use`
    (`use mod/math.(sqrt, PI)`); `.` stays member access only.
    `use` binds selected names into the current scope.
    Selection lists belong to `use` only.
19. **Module tree (locked).** `mod/` is the tree root;
    `mod/<specialty>/` is a specialty namespace (`math/`, `sys/`,
    `std/`, …); `mod/<specialty>/<name>` are individual mods. The
    aggregate `mod/<specialty>/<specialty>` (e.g. `mod/math/math`) is a
    hand-written facade file re-exporting the specialty's public surface
    via `export use …` — never compiler-synthesized. No hidden behavior.
20. **Declaration-only module access (locked).** The module tree is
    reachable only through `mod` and `use` declarations. Module paths
    (`/`) never appear in expressions — no inline access like
    `mod/sys/io.println(...)`; the grammar simply has no production for
    it. Koda one-liner convenience is covered by a prelude with pre-bound
    common mods, not a second mechanism.
21. **One source of truth for names (locked).** A name bound by `mod` or
    `use` is declared exactly once per scope. Redeclaring it — or
    shadowing it with a local `var`/`const`/parameter — is a compile
    error. Below its declaration, the name means one thing, unambiguously.
22. **Aggregate vs individual mods (locked).** Bind the specialty
    aggregate (`mod math::mod/math/math`) only when using several mods
    from that specialty. If you need just one, bind it directly
    (`mod sqrt::mod/math/sqrt`). A file's `mod` block states exactly what
    it depends on — no broader than necessary. Unused `mod`/`use`
    bindings are compile errors.
23. **`use` from `mod`-bound names (locked).** A `use` source may be a
    `mod`-bound name rather than a path: `mod add::mod/math/add` followed
    by `use add.(add)` brings the `add` item into scope, so `add(x, y)`
    calls it while other items stay qualified (`add.precise(a, b)`).
    Single identifiers in `use` position resolve against `mod` bindings —
    real paths are rooted at `mod/`, so there is no ambiguity. The `mod`
    declaration must precede the `use`.
    A combined single-line form (`mod add::mod/math/add.(add)`) was
    considered and rejected: it binds one name to two entities (module
    and item), violating judgment 21. Two lines because two bindings.

## Open questions (need your call)

1. **Match arm separator:** DECIDED — `:` (Pascal style), not `=>`.
2. **Default capability:** DECIDED — Pony mapping (judgment call 10).
3. **Generic bounds:** DECIDED — inline `T: Bound` (judgment call 12).
4. **Lambdas in Maia:** DECIDED — yes, Delphi-style anonymous
   functions/procedures (judgment call 13), for seamless Maia/Koda interop.
5. **Shared ownership story:** DECIDED — capabilities only, no implicit RC
   (judgment call 14).
6. **`consume` vs move-by-default:** DECIDED — Pony semantics (judgment
   call 15).
7. **asm block contents:** DECIDED — opaque per-line text (judgment call 16).

8. **Module discovery:** does the compiler find modules by scanning the
   `mod/` tree implicitly, or must every module be explicitly declared
   (Rust-style `mod` declarations)? The hand-written facade answers the
   public-surface question; this is about how the tree itself is built.
