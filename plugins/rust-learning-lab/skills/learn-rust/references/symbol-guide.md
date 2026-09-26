# Rust symbol guide

English | [日本語](symbol-guide_ja.md)

Use this inventory when the user requests a list or unfamiliar punctuation prevents reading code.
It covers common Rust notation, not every grammar production or a particular macro's private syntax.
Names are search aids: explain the role at the actual occurrence, what it enables, and a small contrast.
For executable contrasts, use [Worked examples](worked-examples.md). Examples in these tables are syntax fragments,
not standalone programs. Choose the relevant rows rather than requiring the whole inventory to be memorized.

## 1. Grouping, separators, and paths

| Notation | Read it here as | Purpose and useful contrast |
|---|---|---|
| `(…)` | `f(x)` calls; `(x + y)` groups; `(x, y)` constructs a tuple | Parentheses alone do not mean a call; inspect what precedes them and the commas inside |
| `()` / `(x,)` | Unit value/type; one-element tuple | `()` carries no additional value; `(x,)` is a tuple, while `(x)` just groups `x` |
| `{…}` | A block in `if ok { 1 } else { 0 }`; named fields in `Point { x: 1, y: 2 }` | A block can yield a value; a struct expression constructs a value. `use path::{A, B}` instead groups imports |
| `[…]` | `[1, 2]` creates an array; `values[i]` indexes | `[T; N]` is an array type and `[T]` a slice type; position separates construction, access, and types |
| `[value; N]` | Repeat an element to make an array of length `N` | `[0; 3]` makes three zeros, unlike `[0, 3]`; non-`Copy` elements have restrictions, so this is not arbitrary cloning |
| `,` | Separate arguments, fields, elements, or match arms | A trailing comma is often allowed; in `(x,)` it distinguishes a tuple from grouping |
| `;` | End a statement; separate element and length in array notation | In `{ compute(); }`, the computed value is discarded and the block yields `()`; `{ compute() }` yields that value |
| `:` | `value: Type` annotates a type; `T: Trait` sets a bound; `Point { x: 1 }` names a field | State whether it connects a binding to a type, a type to a constraint, or a field to its value |
| `::` / `.` | `std::mem::size_of` follows a path; `value.len()` calls a method; `pair.0` accesses a tuple field | `::` selects a named item; `.` uses a value. Neither punctuation alone determines whether the code calls a function |
| `->` / `=>` | `fn f() -> u32` specifies a return type; `Some(x) => x` pairs a pattern with an expression | A type declaration differs from choosing an expression to evaluate; neither arrow itself executes `return` |

## 2. References, types, and patterns

| Notation | Read it here as | Purpose and useful contrast |
|---|---|---|
| `&` / `&mut` | `&value` / `&mut value` borrow; `&T` / `&mut T` are reference types | Access a value without taking ownership; distinguish shared access from exclusive access that permits mutation, and from `mut` on a binding |
| `*` | `*r` follows a reference; `*const T` / `*mut T` declare raw pointer types | Reach a referent or describe a pointer; multiplication is a different use. Dereferencing is not automatically cloning |
| `'a` / `'static` / `'_` | A named lifetime, the static lifetime, or an inferred lifetime | Describe validity relationships, not instructions to extend a value's life. `T: 'static` does not require an owned value to live forever |
| `'label:` / `break 'label` | Name a loop or block; select where to break | Control the destination of a jump; this apostrophe does not annotate a borrow |
| `<…>` / `::<…>` | `Vec<u8>` names a type with arguments; `parse::<u32>()` supplies arguments in an expression | Select a generic instance; distinguish these from runtime arguments and `<` / `>` comparisons. Arguments can include lifetimes and constants, not just types |
| `<T as Trait>::Item` | Select `Item` through `Trait` for `T` | Make the source of an associated item explicit when shorthand would be ambiguous |
| `+` / `?` in bounds | `T: Read + Send` requires both; `T: ?Sized` allows types without a compile-time known size | Combine requirements or relax the implicit `Sized` requirement; these are not addition or failure propagation |
| `'a: 'b` / `T: 'a` | One lifetime outlives another; a type meets a lifetime bound | State validity requirements. Explain references inside `T`, not an instruction to retain every value of `T` |
| `_` | Ignore a matched value in `Some(_)`; infer a type in `Vec<_>` | Avoid binding an unneeded value or spelling a known type. `_name` still binds a value and can move it; it is not the wildcard `_` |
| `@` | `n @ 1..=5` binds the matched value while checking the range | Keep the whole value as `n` and constrain what matches in one pattern |
| `\|` in patterns | `A \| B` accepts either pattern | Combine alternatives; unlike a closure's parameter delimiters or an operator applied to values |
| `&` in patterns | `let &n = reference;` matches a reference | Destructure an existing reference; unlike `&value`, it does not create a new borrow. Binding behavior depends on the pattern and types |

The [pattern reference](https://doc.rust-lang.org/reference/patterns.html) covers matching and binding behavior.

## 3. Operators, ranges, and control flow

| Notation | Read it here as | Purpose and useful contrast |
|---|---|---|
| `=` | Initialize in `let n = 1`, or assign in `n = 2` | Set a value; `==` instead compares values. `type Count = u32` defines an alias, not a runtime assignment |
| `+` `-` `*` `/` `%` | Arithmetic, including remainder `%`; leading `-x` negates | Compute values; integer division differs from floating-point division. For custom types, inspect the operator implementation |
| `+=` `-=` `*=` `/=` `%=` | Compute and assign to the left operand | Update a target; do not mechanically duplicate a complex left operand when expanding the expression |
| `==` `!=` `<` `<=` `>` `>=` | Equality and ordering comparisons | Produce a boolean; `<` and `>` in generics are delimiters instead |
| `&&` `\|\|` | Boolean AND and OR with short-circuit evaluation | Skip the right operand when the left determines the result; the right operand may have side effects |
| `!` / `&` / `\|` / `^` | Negation, AND, OR, XOR | For integers these operate on bits; boolean `!` flips true/false and boolean `&`, `\|`, `^` do not short-circuit |
| `<<` `>>` / `&=` `\|=` `^=` `<<=` `>>=` | Bit shifts and compound bit assignments | Manipulate bits; `>>` may instead close nested generic arguments such as `Vec<Vec<u8>>` |
| `a..b` / `a..=b` | Range excluding / including the end | Select bounds explicitly; for integer ranges, `0..3` covers 0, 1, 2 while `0..=3` also includes 3 |
| `..b` `..=b` `a..` `..` | Ranges with one or both bounds omitted | Useful for slicing; `&values[..]` borrows the whole slice. A range value is not automatically iterable for every form or type |
| `..` in patterns / struct expressions | `Point { x, .. }` ignores other fields; `Point { x: 1, ..old }` takes remaining fields from `old` | Match only needed parts or reuse fields; struct update can move non-`Copy` fields, so it is not automatically a clone |
| `\|x\| expr` / `\|\| expr` | A closure with one / no parameters | Supply an operation that can capture its surroundings; the empty parameter list is not boolean OR |
| `?` after an expression | Unwrap success or propagate failure from the enclosing function or closure | With `Result`, continue with `Ok` or return an error; with `Option`, continue with `Some` or return `None` |
| `!` as a return type | `fn stop() -> !` does not return normally | Express divergence, such as a loop that never exits; it does not mean a returned error |
| `...` | Legacy inclusive range-pattern notation, or variadic parameters in foreign declarations | Do not teach it as a general ellipsis or spread operator. Use `..=` for current inclusive range patterns; foreign variadics have separate restrictions |

Use the [operator reference](https://doc.rust-lang.org/reference/expressions/operator-expr.html) for evaluation and type-specific rules.

## 4. Macros, attributes, and comments

| Notation | Read it here as | Purpose and useful contrast |
|---|---|---|
| `name!(…)` / `name![…]` / `name!{…}` | Invoke a macro | The macro defines its accepted input; changing delimiters or treating it as an ordinary function is not a general equivalence |
| `$name` / `$name:expr` / `$crate` | Macro metavariable, captured expression, or defining crate | Match and reuse syntax in `macro_rules!`; `$` is not ordinary string interpolation |
| `$(…)*` / `$(…)+` / `$(…)?` | Repeat zero or more, one or more, or optionally once in a macro rule | Match or expand repeated syntax. In `$($item:expr),*`, the comma separates repetitions; `*`, `+`, `?` have macro-specific roles |
| `=>` in `macro_rules!` | Separate the matcher from the expansion | Define a syntax transformation; unlike a match arm, it is not runtime branching |
| `#[…]` / `#![…]` | An attribute on the following item / on the enclosing item | Attach instructions such as `#[test]` or `#![allow(dead_code)]`; `#` is not a Rust line-comment marker |
| `//` / `/* … */` | Line / block comments | Explain source without executing it; block comments can nest |
| `///` / `/** … */` | Documentation for the following item | Feed API documentation tools, unlike ordinary comments |
| `//!` / `/*! … */` | Documentation for the enclosing item | Commonly document a module or crate instead of the next function |

For details, consult [macro rules](https://doc.rust-lang.org/reference/macros-by-example.html),
[attributes](https://doc.rust-lang.org/reference/attributes.html), and [comments](https://doc.rust-lang.org/reference/comments.html).

## 5. Literals and prefixes

| Notation | Read it here as | Purpose and useful contrast |
|---|---|---|
| `'x'` / `"text"` | A character / string literal | `char` represents a Unicode scalar value; a string literal has type `&'static str`. A lifetime name has no closing quote |
| `\n` `\r` `\t` `\\` `\0` `\'` `\"` | Escape sequences inside suitable literals | Write newline, carriage return, tab, backslash, NUL, or quotes without confusing delimiters |
| `\xNN` / `\u{…}` | Byte/ASCII or Unicode escape, depending on literal kind | Specify a value numerically; allowed escapes and ranges differ between character, byte, and string literals |
| `r"…"` / `r#"…"#` | Raw string literal | Keep backslashes literal; matching counts of `#` allow quotes in the contents. They are delimiters, not attributes |
| `b'x'` / `b"…"` / `br#"…"#` | Byte, byte string, raw byte string | Represent bytes rather than Unicode characters; `b'x'` is `u8`, and a byte string is a reference to a byte array |
| `c"…"` / `cr#"…"#` | C string / raw C string literal | Produce a `CStr` reference for C interoperability; check target toolchain support and interior-NUL restrictions |
| `1_000` / `42u8` / `1.0f32` | Digit separators / numeric type suffixes | Improve readability or fix a numeric type; `_` here is not a pattern or a missing value |
| `0xFF` / `0o77` / `0b1010` / `1e3` | Hexadecimal, octal, binary, or floating-point exponent notation | Specify how a number is written, not different arithmetic operators |
| `r#type` | A raw identifier | Use a keyword as an identifier where permitted; unlike `r#"…"#`, it is not a string |
| `{}` / `{:?}` / `{name}` inside format strings | Formatting placeholders, such as in `format!("{name}")` | These belong to the formatting language, not Rust blocks. `{{` and `}}` print literal braces; formatting traits determine supported presentations |

Check literal forms against the [token reference](https://doc.rust-lang.org/reference/tokens.html), raw names against
[identifiers](https://doc.rust-lang.org/reference/identifiers.html), and placeholders against
[formatting syntax](https://doc.rust-lang.org/std/fmt/index.html).

## Read relationships and changes, not just glyphs

Use this perspective when the learner asks what makes notation meaningful or useful. Connect a symbol to what it
refers to, its role in the expression, and the effect of evaluating the expression. Apply these ideas to the learner's
code; explain Rust without requiring prior study of semiotic terminology.

### Give each occurrence a role in a relationship

Read `fn first<'a>(left: &'a str, right: &str) -> &'a str` as a relationship between inputs and output, rather than
decoding each apostrophe in isolation. The repeated `'a` connects the validity of the returned reference to the first
input. Its name can consistently change to `'text` without changing that relationship. Similarly, contrast `&value`
with passing the owned value, and `&T` with `&mut T`. The practical benefit becomes visible through what the caller
can keep using and what the callee can do. The [lifetime example](worked-examples.md#a-connects-the-returned-reference-to-an-input)
traces that relationship without suggesting annotations extend an owner's life.

A symbol's contextual roles are not arbitrary choices for the reader. Rust's grammar and types determine them.
For `*r`, ask what `r` refers to; for `a * b`, inspect the operand types and multiplication implementation.
Meaning in context does not mean that the reader can freely reinterpret a well-typed program.

### Separate describing a value from changing state

In `x = x + 1`, the right side reads the old value and computes a new one, while the left side identifies the
destination. It is not an equation asserting that a number equals itself plus one. Track before and after values,
then contrast `x == x + 1`, which tests equality, and `let x = x + 1`, which introduces a new binding.
Use [the executable contrast](worked-examples.md#assignment-and-shadowing-through-time) instead of asking learners
to memorize “assignment” and “shadowing.” This explanation follows Rust's
[assignment rules](https://doc.rust-lang.org/reference/expressions/operator-expr.html#assignment-expressions), not a
claim that assignment is recursive function execution.

### Distinguish a type, an equal value, and the same object

In `let name: String = String::from("Rust")`, `String` names a type, `name` binds a particular value, and
`String::from("Rust")` is an expression that constructs the value. Give each occurrence its role before calling
everything a “symbol.” Rust's [place and value expression rules](https://doc.rust-lang.org/reference/expressions.html#place-expressions-and-value-expressions)
help explain when a location is accessed and when its value is read, copied, or moved.

After `let other = name.clone()`, `name == other` can be true while the two strings are
separately owned. In contrast, `let borrowed = &name` gives access to `name` through a reference. Content equality,
separate ownership, and borrowing the same object answer different questions. Use them to explain why changing
`&name` to `name.clone()` changes ownership and cost even when both let a function read the same text. Do not
equate a name, a value, and a memory address, or use pointer identity as a universal definition of object identity.

### Build the meaning of a whole from its parts

Read `Option<&'a str>` in layers: `str` describes string data, `&'a str` a reference valid for `'a`, and `Option<…>`
the presence or absence of that reference. Read `text.parse::<u32>()?` as selecting a conversion type, calling the
method, and then continuing with a number or returning an error. Name the intermediate types, rather than treating
punctuation as decoration around the “real” operation. The components are already needed to explain the whole.

Also read surrounding constraints back into a part: in `let values: Vec<u32> = source.collect()`, the destination
type helps determine what `collect()` must produce. Meaning is not recovered by reading isolated tokens only
from left to right. Connect bottom-up composition with the signature, expected type, and later uses that constrain
an expression; do not describe type inference as changing runtime behavior after execution.

Keep composition, recursive functions, and broader self-reference distinct. Nested types alone do not
establish self-reference, an `&` reference need not refer to itself, and a lifetime parameter is not recursion.
This perspective helps formulate explanations; it does not by itself demonstrate improved learning or establish
claims about what present-day computers or AI can understand.
