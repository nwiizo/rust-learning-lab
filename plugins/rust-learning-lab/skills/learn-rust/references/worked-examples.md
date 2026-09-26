# Executable worked examples

English | [日本語](worked-examples_ja.md)

Use these contrasts to explain symbols, argument order, borrowing, and early returns. Run
`rustdoc --test --edition 2024 <this-file>` to check successful examples and intentional compile failures.
`assert_eq!` compares two values and fails the test when they differ. Adapt values and explanation depth to the question.

## Read symbols in context

Select the example closest to the question, not every section. Read the relevant expression aloud in ordinary words,
show its types or value flow, then explain what the notation enables and what a replacement changes.
The [official symbol reference](https://doc.rust-lang.org/book/appendix-02-operators.html) helps identify the role;
a glossary name alone does not explain the code. Keep these contextual distinctions when they occur:

- Borrowing and lifetimes: `&T` is a type, `&value` borrows, and `&pattern` matches a reference.
  `'a` can name a lifetime, `'a'` is a character literal, and `'outer: loop` names a loop for control flow.
- Access and operations: `*reference` follows a reference, `a * b` multiplies, and `*const T` / `*mut T` are raw pointer types.
  Distinguish raw pointers from references; do not introduce unsafe code just to teach `*`.
- Names and type arguments: `::` separates a path's components; `::<…>` supplies generic arguments in an expression.
  Neither is an operation on a runtime value by itself.
- Closures and propagation: the bars in `|x| expression` enclose parameters; `a | b` is an operator and `A | B` in a
  pattern offers alternatives. `result?` propagates failure; the `?` in `T: ?Sized` relaxes the implicit `Sized` bound.
- Macros and arrows: `name!(…)` invokes a macro, `!flag` negates a boolean, and `-> !` declares no normal return value.
  `-> T` gives a return type; `pattern => expression` associates a match arm with its result. In `macro_rules!`, `=>`
  separates a matcher from its expansion instead. Do not treat either arrow as a general instruction to return.

## Outer arguments and closure parameters

```rust
let values = [1, 2, 3];
let remaining = values.into_iter().fold(10, |acc, item| acc - item);
assert_eq!(remaining, 4);

let reversed = values.into_iter().fold(10, |item, acc| acc - item);
assert_eq!(reversed, -8);
```

`into_iter()` creates an iterator over the array's elements, and `fold` updates an accumulator.
Here the elements are inferred as `i32`; the array is also `Copy`, so `values` remains available for the second call.
That does not automatically hold for an array of `String`.

The relevant parts of the [`fold` definition](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.fold) are
`init: B`, `f: F`, and `F: FnMut(B, Self::Item) -> B`. `B` is the accumulator type, `Self::Item` the item type,
and `F` the supplied operation's type. `FnMut(...) -> B` requires a callable accepting those argument types and
returning `B`. Both values here are `i32`. The caller gives `fold` an initial `10` and an operation; `fold` supplies
values to that operation.

The bars in `|acc, item|` enclose parameters; `acc - item` is the returned expression. Parameter names are freely
chosen, but their positions remain accumulator first and current item second.
This lets the caller supply how to combine values while `fold` handles iteration. The bars define an operation to
call later; `(acc, item)` would instead construct a tuple from existing values, not a callable operation.

| Call | Accumulator (first argument) | Item (second argument) | Result of `acc - item` |
|---|---:|---:|---:|
| 1 | 10 | 1 | 9 |
| 2 | 9 | 2 | 7 |
| 3 | 7 | 3 | 4 |

The second expression reverses names, making the calculation item minus accumulator: `-9 → 11 → -8`.
Both arguments have the same type, so the code compiles while meaning something different.
This order comes from this API's definition. Passing the accumulated state along helps explain how it works, but
Rust does not impose this order on every API, and that explanation does not establish the historical design reason.

## `&` in a type and `&` in a borrowing expression

```rust
fn byte_len(text: &str) -> usize {
    text.len()
}

let name = String::from("Rust");
let length = byte_len(&name);
assert_eq!(length, 4);
assert_eq!(name, "Rust");
```

`text: &str` declares a shared reference to a string slice as the parameter type. `&name` is an expression borrowing
`name`; here a conversion from `&String` to `&str` applies. The function only borrows for reading, so `name` remains
usable afterward. `-> usize` is the return type. The final expression `text.len()` has no semicolon and supplies
the return value. `len()` counts bytes, not Japanese characters or characters in general.

The next example intentionally moves ownership and fails with E0382 at the final line.

```compile_fail,E0382
fn consume(text: String) -> usize {
    text.len()
}
let name = String::from("Rust");
let length = consume(name);
println!("{name}: {length}");
```

Borrowing is not always the correct choice. Passing by value can be appropriate when the receiving code needs ownership,
such as to retain the value. If choosing `clone()`, explain why separate ownership is needed and what duplication costs.

## `*` follows a reference; `&mut` permits updating its target

```rust
let mut count = 3;
let borrowed: &mut i32 = &mut count;
*borrowed += 1;
assert_eq!(count, 4);

let reference: &i32 = &count;
let copied: i32 = *reference;
assert_eq!(copied * 2, 8);
```

Read `*borrowed += 1` as “add one to the integer this reference points to.” `&mut count` borrows `count` for exclusive
access, and `&mut i32` describes that access in the type. `mut count` permits changing the binding's value;
the binding `borrowed` itself need not be mutable to update its target. Once that borrow is no longer used, `count`
can be read again. Without `*`, `borrowed += 1` attempts an unsupported operation on `&mut i32`, not on the integer.

In `let copied = *reference`, following the reference yields an `i32` place whose value is copied because `i32` is `Copy`.
The `*` does not mean “clone”: moving a `String` out through `&String` this way is not allowed.
In `copied * 2`, the same glyph means multiplication. Explain the operand types and position before naming the operator.
See the [operator rules](https://doc.rust-lang.org/reference/expressions/operator-expr.html).

## `'a` connects the returned reference to an input

```rust
fn first<'a>(left: &'a str, _right: &str) -> &'a str {
    left
}

let owner = String::from("Rust");
let selected;
{
    let temporary = String::from("short-lived");
    selected = first(&owner, &temporary);
}
assert_eq!(selected, "Rust");
```

`owner` owns the returned text. Read the signature as “borrow two strings and return a reference tied to the first.”
`<'a>` declares a lifetime parameter; `&'a str` uses it. The apostrophe marks a lifetime name, not a quoted character.
`_right` is an ordinary parameter whose leading underscore indicates intentional non-use; it makes the two-input
contrast visible here. The second input need not stay valid until the last use of `selected`.

The benefit is making the input/output relationship available to callers and the compiler without inspecting the
body. Annotations do not keep `owner` alive or force both owners to live equally long. The caller must still keep
`owner` valid while using `selected`. See the [lifetime rules and elision](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html).

This intentional failure removes the relationship: with two borrowed inputs, the elision rules do not select which
one the output borrows from, even though the body returns `left`.

```compile_fail,E0106
fn first(left: &str, _right: &str) -> &str {
    left
}
```

With only one input, `fn first(left: &str) -> &str { left }` can omit the annotations. Omission applies a known
relationship; it does not remove lifetimes or the compiler's checks.

## Where `?` returns on failure

```rust
use std::num::ParseIntError;

fn parse_count(text: &str) -> Result<u32, ParseIntError> {
    let count = text.parse::<u32>()?;
    Ok(count)
}

assert_eq!(parse_count("12"), Ok(12));
assert!(parse_count("twelve").is_err());
assert!(parse_count("-1").is_err());
```

`Result<u32, ParseIntError>` distinguishes a successful `u32` from a parsing error. `::<u32>` specifies the target type
for parsing. For this `Result`, `?` extracts the value from `Ok`; for `Err`, it returns an error from `parse_count`,
without reaching `Ok(count)`. The error type is the same here; propagating a different type requires an applicable conversion.
Check the [`?` rules](https://doc.rust-lang.org/reference/expressions/operator-expr.html#the-try-propagation-expression)
in terms of the actual type and enclosing function or closure.

### `::`, `::<…>`, `->`, and `=>` in the same operation

Here is the same operation with the early return written out; both branches remain visible.

```rust
use std::num::ParseIntError;

fn parse_count(text: &str) -> Result<u32, ParseIntError> {
    let parsed: Result<u32, ParseIntError> = text.parse();
    let count = match parsed {
        Ok(value) => value,
        Err(error) => return Err(error),
    };
    Ok(count)
}

assert_eq!(parse_count("12"), Ok(12));
assert!(parse_count("twelve").is_err());
```

`std::num::ParseIntError` is a path locating a type; `String::from` above locates an associated function on a type.
`text.parse()` calls a method on a value. `::` is not itself a function call, and `.` is not interchangeable with it.
In `text.parse::<u32>()`, the `::<u32>` selects the output type; `()` contains no explicit runtime arguments and
`text` supplies the receiver. This version puts the type on `parsed` instead, allowing `parse()` to infer it.
Even the earlier version can omit `::<u32>` because `Ok(count)` and the declared return type constrain `count`.
The explicit type argument makes the choice readable locally; it is not always needed to compile.

`-> Result<…>` tells callers what the function returns. Within `match`, `=>` connects a pattern to the expression
to evaluate when it matches: `Ok(value)` binds the successful number, then `value` supplies the arm's result.
`=>` alone does not exit the function; the `return` in the error arm does. `?` makes this propagation concise without
discarding errors. Simply removing `?` from the earlier example leaves a `Result` in `count`, so `Ok(count)` no
longer has the declared return type. For `Option`, `?` similarly extracts `Some` or returns `None` from an enclosing
function or closure returning `Option`; it is not a panic or a boolean test.

### `!` after a name and before a value

```rust
let ready = false;
assert!(!ready);
let message = format!("ready={ready}");
assert_eq!(message, "ready=false");
```

`assert!(…)` invokes a macro that fails the test if its condition is false; `!ready` is boolean negation and evaluates
to `true` here. `format!` uses formatting syntax to produce a `String`. The macro marker tells the reader to use that
macro's input rules; deleting it attempts an ordinary call, not the same operation with optional punctuation.
These macros do not require an `unsafe` block: `!` is not a danger marker. Its meaning depends on its position and,
for an operator such as `!value`, the operand type.

## Assignment and shadowing through time

```rust
let mut x = 3;
let before = x;
x = x + 1;
assert_eq!((before, x), (3, 4));
assert!(!(x == x + 1));

let x = 3;
let original = &x;
let x = x + 1;
assert_eq!((*original, x), (3, 4));
```

In the first part, the right-hand `x` reads `3`, `+ 1` produces `4`, and assignment updates the destination named
by the left-hand `x`. `before` keeps a copy because the value is an `i32`. `==` compares rather than assigns;
`!(…)` negates that comparison and `assert!` verifies it. These small integers do not overflow.

In the second part, each `let` introduces a binding. The final right-hand `x` still names the preceding binding,
and its value plus one initializes the new `x`. `original` continues to refer to the earlier `3`, as `*original`
shows. Shadowing changes which binding the name resolves to; it does not retarget an existing reference.
This is why `let x = x + 1` does not require `mut`, and why it is not merely another spelling of assignment.
See the [explanation of variables and shadowing](https://doc.rust-lang.org/book/ch03-01-variables-and-mutability.html).

## A lifetime relationship does not select a runtime value

```rust
fn choose<'a>(take_left: bool, left: &'a str, right: &'a str) -> &'a str {
    if take_left { left } else { right }
}

let left = String::from("left");
let right = String::from("right");
assert_eq!(choose(true, &left, &right), "left");
assert_eq!(choose(false, &left, &right), "right");
```

`<'a>` declares a lifetime parameter; both input references and the output use it. The signature lets the caller
rely on the result while both inputs can satisfy that shared validity requirement. The function body chooses
which text is returned; `'a` neither makes that choice nor requires equal owner lifetimes.
`if … { … } else { … }` is an expression whose chosen branch supplies the returned reference, without copying text.

Now consider the same function with `true` but a short-lived second owner. This intentionally fails with E0597.

```compile_fail,E0597
fn choose<'a>(take_left: bool, left: &'a str, right: &'a str) -> &'a str {
    if take_left { left } else { right }
}

let left = String::from("left");
let selected;
{
    let right = String::from("right");
    selected = choose(true, &left, &right);
}
assert_eq!(selected, "left");
```

The caller is checked against the declared relationship, not a promise inferred by specializing the function body
for this `true`. The signature ties the result to both inputs; `right` cannot satisfy the later use of `selected`.
This does not prove that this particular execution would actually return a dangling reference. It shows the
limits of what the interface permits the caller to establish. If the operation always returns the first input,
use the earlier `first` signature, with the second lifetime independent. If either input can be returned, retain
the owners long enough or return owned text when that is the intended behavior. Adding `'static` cannot keep a
local owner alive. See the [lifetime explanation](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html).

## A recursive type, indirection, and a recursive call solve different problems

This intentional E0072 failure describes a value containing another full value of the same type, without
indirection. The compiler cannot assign it a finite layout.

```compile_fail,E0072
enum List {
    End,
    Link(i32, List),
}
```

Adding `Box` gives each link a fixed-size owning pointer to the next list value.

```rust
enum List {
    End,
    Link(i32, Box<List>),
}

impl List {
    fn sum(&self) -> i32 {
        match self {
            Self::End => 0,
            Self::Link(value, rest) => *value + rest.sum(),
        }
    }
}

let values = List::Link(2, Box::new(List::Link(3, Box::new(List::End))));
assert_eq!(values.sum(), 5);
assert_eq!(values.sum(), 5);
```

Read the pieces by their jobs:

| Piece | Role and consequence |
|---|---|
| `Box<List>` | Own a separately allocated `List`; `<List>` is a type argument, not a function call. Indirection makes the enum's layout finite, not the whole list a fixed-size allocation |
| `List::Link(…)` / `Box::new(…)` | Construct a variant / allocate and own its contents. `::` selects names; `()` supplies values |
| `&self` / `match self` | Borrow the list for reading. Matching a shared reference with these patterns binds `value: &i32` and `rest: &Box<List>` automatically; it does not move the list apart |
| `*value` / `rest.sum()` | Read the integer through its reference / borrow the next list for the recursive call. Method lookup dereferences the box and borrows as needed |
| `Self::End => 0` / `-> i32` | The end case supplies zero; the signature declares an integer result. The other arm adds the current number to the tail's sum |

The calls follow `2 + (3 + 0)` for this list. A second `sum()` still works because the first borrowed it.
`Box` solves the type's layout problem; the recursive call traverses the values, and `End` stops this traversal.
The type refers to itself, but this value is a finite chain, not a node pointing back to itself. A recursive type
can also be processed with a loop. Conversely, a recursive function need not operate on a recursive type.
This is an explanation example, not a recommendation to replace `Vec` with a linked list. Deep recursive calls
can exhaust the stack; heap indirection does not make execution depth unlimited. The small sum here does not overflow.
See [recursive types with `Box`](https://doc.rust-lang.org/book/ch15-01-box.html),
[pattern binding modes](https://doc.rust-lang.org/reference/patterns.html#binding-modes), and
[method lookup](https://doc.rust-lang.org/reference/expressions/method-call-expr.html).

## Macro expansion operates on syntax; function calls receive evaluated values

```rust
macro_rules! twice {
    ($value:expr) => { $value + $value };
}

let mut calls = 0;
let result = twice!({ calls += 1; calls });
assert_eq!((result, calls), (3, 2));

fn twice_value(value: i32) -> i32 {
    value + value
}

let mut calls = 0;
let result = twice_value({ calls += 1; calls });
assert_eq!((result, calls), (2, 1));
```

`$value:expr` captures an expression's syntax. `=>` separates that pattern from the expansion, and each `$value`
inserts the captured syntax. `twice!(…)` therefore expands to two evaluations of the block: the first yields `1`,
the second `2`. Inside each block, `calls += 1;` changes state and the final `calls` supplies the value.
Expansion itself does not increment `calls`; executing the resulting expression does.

The ordinary function call first evaluates its argument block once, obtaining `1`, then adds that integer to
itself inside the function. Its two uses of `value` do not re-evaluate the caller's block. `-> i32` describes
the result type; unlike the macro's `=>`, it does not define a syntax transformation.

This macro deliberately exposes a repeated-evaluation pitfall. Macros need not evaluate arguments twice:
binding the input once in the expansion can avoid it, but that is a behavior change for this example.
When explaining `!` and `$`, identify which syntax is generated and when its expressions run. Neither the
exclamation mark nor this expansion implies a recursive call or a general ability to redefine runtime meaning.
See [macro expansion rules](https://doc.rust-lang.org/reference/macros-by-example.html).

## Read the signature, then check the search

```rust
struct Record {
    key: String,
    score: u32,
}

fn find_record<'a>(records: &'a [Record], key: &str) -> Option<&'a Record> {
    records.iter().find(|record| record.key.as_str() == key)
}

let records = [
    Record { key: String::from("rust"), score: 1 },
    Record { key: String::from("rust"), score: 2 },
];
let selected;
{
    let key = String::from("rust");
    selected = find_record(&records, &key);
}
let score = match selected {
    Some(record) => Some(record.score),
    None => None,
};
assert_eq!(score, Some(1));
assert!(find_record(&records, "missing").is_none());
```

First hide the body and read the signature. `&[Record]` borrows a slice, `&str` borrows the search key, and
`Option<&'a Record>` permits absence or a borrowed result with the declared relationship to `records`.
The local key is dropped before `selected` is used: its lifetime is not tied to the output. The signature alone
does not say which duplicate wins or how much work the search performs.

Now inspect the body. `iter()` yields `&Record`; `find` borrows each candidate for its predicate, so the closure
parameter is `&&Record`. Field access automatically dereferences to reach `key`; `as_str()` borrows its text,
and `==` compares it with the supplied key. `find` returns the first matching iterator item, an `&Record`, not the
predicate's temporary `&&Record`. The returned `Option` is the function's final expression without a semicolon.

The body establishes a sequential search stopping at the first matching key, consistent with the duplicate test.
It does not clone strings or allocate a result collection; the calls to `String::from` allocate test input outside
the search. Worst-case search work grows with the number of records and the cost of string comparisons, not just
the record count. Actual elapsed time still requires measurement. The match on `selected` copies the `u32` score
and retains the distinction between `Some` and `None`, without panicking on absence.
See [`Iterator::find`](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.find) and
[`String::as_str`](https://doc.rust-lang.org/std/string/struct.String.html#method.as_str).

## An error return does not undo a side effect

This deliberately failing operation models a state change followed by a missing acknowledgement.
It is a local control-flow example, not a network or durable-storage implementation.

```rust
#[derive(Debug, PartialEq)]
enum SubmitError {
    AcknowledgementLost,
}

fn submit(applied: &mut u32) -> Result<(), SubmitError> {
    *applied += 1;
    Err(SubmitError::AcknowledgementLost)
}

fn attempt(applied: &mut u32) -> Result<&'static str, SubmitError> {
    submit(applied)?;
    Ok("confirmed")
}

let mut applied = 0;
assert_eq!(attempt(&mut applied), Err(SubmitError::AcknowledgementLost));
assert_eq!(applied, 1);
assert!(attempt(&mut applied).is_err());
assert_eq!(applied, 2);
```

`&mut u32` lets `submit` change the caller's counter. `*applied += 1` performs the effect before constructing
the error. `?` exits `attempt` before `Ok("confirmed")`, but neither `Result` nor `?` restores the previous
counter. Retrying performs the change again. Here the test can inspect the true state; a remote caller
may be unable to do so after losing the response. A recovery policy needs more than a generic “retry errors.”
Use [System boundaries](systems-thinking.md) for request identities, unknown outcomes, and verification scope.

## Validation gives a type a narrower meaning

Here the chosen domain rule is a quantity from 1 through 100. Parsing a `u32` checks different conditions.

```rust
mod order {
    #[derive(Debug, PartialEq)]
    pub struct Quantity(u32);

    #[derive(Debug, PartialEq)]
    pub enum QuantityError {
        OutOfRange,
    }

    impl TryFrom<u32> for Quantity {
        type Error = QuantityError;

        fn try_from(value: u32) -> Result<Self, Self::Error> {
            if (1..=100).contains(&value) {
                Ok(Self(value))
            } else {
                Err(QuantityError::OutOfRange)
            }
        }
    }

    impl Quantity {
        pub fn get(&self) -> u32 {
            self.0
        }
    }
}

use order::{Quantity, QuantityError};

assert_eq!("0".parse::<u32>(), Ok(0));
assert_eq!(Quantity::try_from(0), Err(QuantityError::OutOfRange));
assert_eq!(Quantity::try_from(101), Err(QuantityError::OutOfRange));
assert_eq!(Quantity::try_from(1).map(|q| q.get()), Ok(1));
assert_eq!(Quantity::try_from(100).map(|q| q.get()), Ok(100));
assert!("many".parse::<u32>().is_err());
```

`struct Quantity(u32)` defines a distinct type; its field is private to `order`. `pub` exposes the type
and selected operations, not direct field construction outside that module. `TryFrom<u32>` describes
a conversion that can fail; `type Error` selects its error type. `1..=100` includes both endpoints,
and `Self(value)` constructs the validated type. `get(&self)` reads through a borrow and copies a `u32`.

The benefit is concentrating the range decision at a boundary, so later operations can accept `Quantity`
instead of repeatedly interpreting an unrestricted number. That property depends on this module's entire
construction and mutation surface. Adding an unchecked constructor, setter, or derived deserializer can
invalidate the reasoning. The wrapper does not prove that inventory exists or the caller may place an order.
The assertions check syntax failure, domain failure, and both accepted endpoints separately.

## Preserve required behavior across implementations

```rust
fn count_positive_loop(values: &[i32]) -> usize {
    let mut count = 0;
    for value in values {
        if *value > 0 {
            count += 1;
        }
    }
    count
}

fn count_positive_iter(values: &[i32]) -> usize {
    values.iter().filter(|value| **value > 0).count()
}

for (input, expected) in [(&[-1, 0, 2, 3][..], 2), (&[][..], 0), (&[0][..], 0)] {
    assert_eq!(count_positive_loop(input), expected);
    assert_eq!(count_positive_iter(input), expected);
}
```

The requirement is the number of positive integers, excluding `0`. Compare each implementation with expectations
from that requirement, not only with the other implementation. An input containing `0` detects changing `> 0` to `>= 0`.
`iter()` yields `&i32`, and `filter` supplies a reference to each item, making `value` an `&&i32`.
`**value` follows two references. If this is too much at once, first trace types and conditions in the loop version.
Passing these finite examples is not proof of equivalence for all inputs and side effects.
