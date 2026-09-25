# Executable worked examples

English | [日本語](worked-examples_ja.md)

Use these contrasts to explain argument order, borrowing, and early returns. Run
`rustdoc --test --edition 2024 <this-file>` to check successful examples and intentional compile failures.
`assert_eq!` compares two values and fails the test when they differ. Adapt values and explanation depth to the question.

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
