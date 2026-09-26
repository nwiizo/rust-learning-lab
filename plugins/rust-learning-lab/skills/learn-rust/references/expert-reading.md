# Reading types for guarantees and open questions

English | [日本語](expert-reading_ja.md)

Use this reference for deeper questions about what an experienced Rust reader can infer from symbols and types.
Start with the user's purpose and one signature. State what follows from the language rules, what the implementation
must confirm, and what requires measurement. These are ways to reason, not a checklist to recite or a claim that
all experienced programmers think alike. Use [Symbol guide](symbol-guide.md) for unfamiliar notation.

## Read a signature as a set of connected constraints

Consider this signature, with `Record` a project-defined type:

`fn find_record<'a>(records: &'a [Record], key: &str) -> Option<&'a Record>`

Read it as “search using borrowed records and a borrowed key, and perhaps return a borrowed record whose validity
is tied to the records.” Then make each inference and its limit explicit:

| Visible information | What the caller can rely on | What it does not establish |
|---|---|---|
| `records: &'a [Record]` | Borrow a slice without consuming the caller's collection; the callee cannot mutate ordinary fields through this shared reference | No interior mutation, no global side effects, no synchronization, or no allocation inside the function |
| `key: &str` | Borrow a string slice; no caller-owned `String` has to move into the function | How keys compare, whether normalization is performed, or whether text is copied internally |
| `Option<…>` | A normal return has two cases: `Some` and `None`; callers must choose how to handle absence | Why nothing was found, whether the implementation can panic, or whether it always terminates |
| `&'a Record` in the output | The returned reference must be valid for the declared relationship; the key's independent lifetime does not limit it | Which record is chosen, or a proof that the returned reference literally came from this slice; a suitable longer-lived reference could also fit |
| The same `'a` on input and output | Callers must preserve the required borrow validity while using the result | That the owner is kept alive automatically, or that the result remains usable after dropping the collection |

The output lifetime represents a validity constraint, not a complete account of runtime provenance. A descriptive
name suggests intent, but does not prove membership, ordering, uniqueness, or complexity. The
[executable search example](worked-examples.md#read-the-signature-then-check-the-search) lets the key expire before
using the result, then inspects the body to establish which element is chosen and how it is searched.

When exploring the boundary, use the
[two-input lifetime example](worked-examples.md#a-lifetime-relationship-does-not-select-a-runtime-value): even a
`true` argument does not let a caller discard a lifetime constraint in the declaration. Compiler rejection can
mean the declared information is insufficient, not that this particular input certainly causes memory corruption.

## Follow ownership and permitted access through expressions

Read `value`, `&value`, `&mut value`, and `*reference` as different operations, not cosmetic variants.
For passing `T` by value, first check `Copy`: a non-`Copy` value moves, while a `Copy` value is copied. A `String`
move transfers ownership without cloning the text buffer. `clone()` has type-specific behavior and cost;
cloning a `String` differs from cloning an `Rc<T>` or `Arc<T>`. Avoid a blanket “clone is slow” rule.

For `&T`, shared access restricts ordinary mutation but does not imply purity. `Cell`, `RefCell`, atomics, locks,
or other interior-mutable fields can permit state changes through shared access. For `&mut T`, reason about
exclusive access to the borrowed target and any reborrows, not “there is only one pointer anywhere.” It does not
grant exclusive access to unrelated state. Consult the [interior mutability documentation](https://doc.rust-lang.org/std/cell/index.html).

For `*r`, inspect both the referent type and what the surrounding expression does: it may read a `Copy` value,
borrow the referent, mutate it, or attempt a forbidden move. If method syntax hides these steps, show the receiver
type and necessary dereferencing or borrowing. In the [recursive list example](worked-examples.md#a-recursive-type-indirection-and-a-recursive-call-solve-different-problems),
matching `&List` binds references, and `rest.sum()` borrows the next node rather than consuming it.
In Edition 2024, that implicit reference binding does not allow adding redundant `ref`, `ref mut`, or `mut`
modifiers freely. Explain the inferred binding type first; use the [2024 pattern rules](rust-2024.md) when
contrasting older examples with the current edition.

## Read abstractions by who chooses the type and when work happens

| Notation or bound | Useful deduction | Boundary to preserve |
|---|---|---|
| `T: Trait` or argument-position `impl Trait` | The caller supplies a concrete type meeting the bound; the body can use the bound's operations | A bound does not guarantee a particular algorithm, performance, or side-effect-free implementation |
| Return-position `impl Trait` | The implementation supplies a hidden concrete type for each generic instantiation; the caller uses its exposed bounds | Branches cannot freely return unrelated types solely because both implement the trait; neither heap allocation nor dynamic dispatch follows from `impl` alone |
| `&dyn Trait` / `Box<dyn Trait>` | Use a trait object with dynamic method dispatch; the first borrows, the second owns | `dyn` alone does not imply a heap allocation; inspect the surrounding pointer and construction |
| `FnOnce` / `FnMut` / `Fn` | Calling through these traits respectively consumes, mutably borrows, or shares the callable receiver | `FnOnce` does not promise the API calls it exactly once; `Fn` does not imply pure computation or no interior mutation |
| `Iterator<Item = &'a T>` | Each yielded item is a shared reference valid for `'a` | The bound alone does not give order, finiteness, or cost. For ordinary lazy adaptors such as `map`, identify the consumer that drives them |

For Edition 2024 return-position `impl Trait`, also inspect lifetime capture: without `use<…>`, all in-scope
generic parameters, including lifetimes, are implicitly captured. An omitted visible `+ 'a` does not establish
independence from a borrowed input. A precise capture bound such as `use<'a, T>` names the permitted captures;
this `use` is not an import. Keep this separate from ordinary reference lifetime elision, and consult the
[2024 capture rules](https://doc.rust-lang.org/edition-guide/rust-2024/rpit-lifetime-capture.html) when relevant.

Explain generic argument names separately from values: `Item = &'a T` constrains an associated type, while
`|item| …` binds a value each time a closure is called. `move` changes how a closure captures its surroundings;
the body determines which call traits it implements. Capturing owned data does not by itself make a closure
callable only once or cause it to run on another thread. For `.filter(|x| …)`, inspect the iterator's `Item` and
the predicate signature before interpreting `*x`: another reference layer may be involved.

Verify these distinctions with [opaque types](https://doc.rust-lang.org/reference/types/impl-trait.html),
[trait objects](https://doc.rust-lang.org/reference/types/trait-object.html), and the
[`FnOnce`](https://doc.rust-lang.org/std/ops/trait.FnOnce.html),
[`FnMut`](https://doc.rust-lang.org/std/ops/trait.FnMut.html), and
[`Fn`](https://doc.rust-lang.org/std/ops/trait.Fn.html) definitions.

## Separate syntax processing, type checking, and execution

`::<u32>` chooses a type argument and `'a` describes a validity relationship; neither passes a numeric type ID or
a lifetime duration as an ordinary runtime argument. `?` in an expression can change runtime control flow.
`$value:expr` in a macro captures syntax that will be expanded before the resulting program runs.
For a macro defined in Edition 2024, `expr` also matches top-level const-block and underscore expressions;
`expr_2021` preserves the older matching range. Matching syntax does not guarantee that the expansion type-checks.
Use the [edition-specific macro rules](https://doc.rust-lang.org/edition-guide/rust-2024/macro-fragment-specifiers.html)
when the accepted syntax or the selected macro arm matters.
Use the [macro/function contrast](worked-examples.md#macro-expansion-operates-on-syntax-function-calls-receive-evaluated-values)
to show why repeating captured syntax can repeat side effects while repeating a function parameter does not
repeat the caller's argument evaluation. Do not collapse these stages into “the symbol does something.”

Likewise, distinguish the recursive definition of a type, traversal of its values, and a cycle in the values
themselves. `Box` supplies indirection for a finite layout, not an automatic traversal or unlimited stack space.
The [list example](worked-examples.md#a-recursive-type-indirection-and-a-recursive-call-solve-different-problems)
makes each of these jobs visible without importing a general philosophical claim as a language rule.

## Ask which representation and behavior a type promises

| Visible type or operation | Next question |
|---|---|
| `String`, `&str`, `char`, or a byte slice | Is the operation counting bytes, Unicode scalar values, or user-perceived characters? |
| `Eq`, `Ord`, or `Hash` | Does the implementation obey the required laws and their relationships, including after mutation? |
| `Write` and `Result` | Was all input written, and what does completion establish about buffering or durable storage? |
| `Pin<P>` | What does `P` point to, and does `Unpin` permit moving that pointee? |
| A safe API implemented with `unsafe` | Which invariants make every permitted safe use sound, including errors and destruction? |

A Rust [`char`](https://doc.rust-lang.org/std/primitive.char.html) is a Unicode scalar value;
[`String::len`](https://doc.rust-lang.org/std/string/struct.String.html#method.len) counts bytes.
Do not infer display width or a user's notion of one character from either. A domain name such as `Length`
needs units and operations to make its meaning clear.

Trait implementations carry semantic obligations beyond compiler checking. For example,
[`Eq`](https://doc.rust-lang.org/std/cmp/trait.Eq.html) requires reflexivity but the compiler does not prove it.
An implementation that compiles is not evidence that ordering, equality, and hashing agree.

[`Write::write`](https://doc.rust-lang.org/std/io/trait.Write.html) can successfully write fewer bytes than
requested. `write_all` handles that repetition, but an error can follow earlier successful writes.
Neither a generic writer's success nor `flush` universally means durable storage; inspect the concrete
writer and, for files, the stated scope of [`sync_all`](https://doc.rust-lang.org/std/fs/struct.File.html#method.sync_all).

[`Pin`](https://doc.rust-lang.org/std/pin/index.html) constrains moving the pointee through the pinned API;
moving the pointer wrapper is a separate question. It is neither general immutability nor synchronization.
When `unsafe` underlies a safe interface, compilation and a few passing examples do not prove its invariants.

For failures spanning processes, storage, or time, use [System boundaries](systems-thinking.md).
For choosing representations and checking a change, use [Engineering practice](engineering-practice.md).
Select the relevant question instead of presenting this entire table for every type.

## Turn a reading into a prediction that can be checked

For a deeper explanation, reveal the chain of reasoning: point to the notation, state the inferred type or
permission, predict one observable consequence, and identify the evidence needed to resolve what remains.
Suitable predictions include whether the original value can still be used, which owner must remain alive,
whether failure returns from the function or closure, and how often a side-effecting expression runs.
Answer first; independent prediction or a small modification is optional unless practice was requested.

Inspect actual definitions when a trait, alias, macro, or implicit conversion matters. A successful example
establishes that behavior for that example, not every property of the abstraction. Use profiling or measurement
for performance claims that types and implementation inspection cannot settle. Do not mistake compilation,
an explanation delivered, and understanding demonstrated independently for the same outcome.
