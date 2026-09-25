# Reviewing a positive-integer count through reference flow

English | [日本語](review-example_ja.md)

This short review examines the two `count_positive` implementations in the
[worked examples](worked-examples.md#preserve-required-behavior-across-implementations).
It demonstrates what to check and how to read the syntax when replacing a loop with an iterator.
It is not a review result for an actual PR or a user's project.

## Purpose and findings

The goal is to count integers greater than zero without modifying the input. Both implementations follow that condition.
The examples check ordinary, empty, and zero-only input against expectations derived from the requirement.
No defects are identified within that scope; this does not establish safety of an arbitrary surrounding program.

## Where values go in a one-line implementation

```rust
fn count_positive(values: &[i32]) -> usize {
    values.iter().filter(|value| **value > 0).count()
}

let input = [-1, 0, 2, 3];
assert_eq!(count_positive(&input), 2);
assert_eq!(input, [-1, 0, 2, 3]);
assert_eq!(count_positive(&[]), 0);
assert_eq!(count_positive(&[0]), 0);
```

`fn count_positive(values: &[i32]) -> usize` declares a function that accepts a shared reference to an integer slice
and returns a count. `values` is the parameter, `&[i32]` its type, and `usize` the return type after `->`.
In `count_positive(&input)`, the borrowed array is coerced to a slice reference and passed to `values`.
Reading the collection does not require owning a `Vec<i32>` or accepting a mutable `&mut [i32]`.

| Stage | Receives | Passes onward |
|---|---|---|
| `values.iter()` | A receiver of type `&[i32]` | An iterator with items of type `&i32` |
| `.filter(...)` | An iterator and a predicate closure | An iterator over qualifying `&i32` items |
| `\|value\| **value > 0` | A reference to an item: `&&i32` | A `bool` indicating whether to keep it |
| `.count()` | The filtered iterator | A count of type `usize` |

The explicit argument to `filter` is one closure. The iteration machinery supplies `value` when calling it.
`|value|` declares the parameter; the following expression is its result. There are two reference layers because
items are `&i32` and the predicate receives a reference to each item. Each `*` follows one layer.
An item continues when `**value > 0` is `true`.

Constructing `iter().filter(...)` has not yet tested every element. `count()` drives iteration and runs the predicate.
The final expression has no semicolon, so its count is returned. Nothing deletes or reorders the original `input`,
which remains usable after the call.

## Check more than brevity

The types would remain the same if `> 0` became `>= 0`, but counting zero would violate the requirement.
The `[0]` case with expected count zero detects that mistake. Comparing implementations only with each other could
miss the same mistake in both; retain expectations derived from the requirement.

The predicate has no side effects here. If another change adds logging or other effects, when and how often it runs
also matter. Similar-looking code alone does not establish equivalent behavior.

## Change one condition

If the requirement becomes “integers greater than or equal to zero,” which expression and expectations need changing?

<details>
<summary>Answer and reasoning</summary>

Use `**value >= 0`. The first input produces 3 and `[0]` produces 1. Empty input still produces 0.
Neither the signature nor the decision to borrow input needs changing.

</details>

## Sources and verification

Both original implementations are in the [worked examples](worked-examples.md). Check this document's standalone
example with:

```sh
rustdoc --test --edition 2024 plugins/rust-learning-lab/skills/learn-rust/references/review-example.md
```

One executable example has been checked with Rust 1.98.1 / Edition 2024. This verifies the finite inputs and Rust
example above; it does not measure learning effectiveness or a model's review accuracy.
