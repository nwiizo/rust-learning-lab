# Build explanations around decisions and evidence

English | [日本語](verified-explanations_ja.md)

Use the relevant sections for substantial Rust explanations, learning reviews, diagrams, or chaptered materials.
A short syntax question still needs a direct answer. For review destinations, source revisions, and indexes,
follow [Review documents](review-document.md).

## Identify the decision the reader cannot yet make

Use [Learning design](learning-design.md) to identify the uncertain step from the question, code, and reader's
explanation. Someone who routinely fixes borrow errors may still be unable to explain why moving a reference's
last use changes whether a mutation compiles. Set a concrete goal such as locating that later use and explaining
the conflict. State necessary assumptions about prior knowledge, then introduce only the missing prerequisites.

Keep a compiler's suggested edit distinct from the reader's goal. Removing a later use may make an example compile
while losing behavior the caller needs. Explain which condition a contrast changes, what the compiler then accepts,
and whether the change remains suitable for the original task. Return to the original code after the contrast.

## Connect important claims to appropriate checks

For claims that depend on execution or compilation, check the smallest relevant example before presenting an
observed result. For a language rule, use official specifications or API documentation and show its application;
one successful example cannot establish the rule for all programs.

| Claim in a Rust explanation | Evidence to use | Limit to preserve |
|---|---|---|
| This expression prevents compilation | The exact example, compatible toolchain, and diagnostic at the expression | A failed build can have an unrelated cause |
| This change preserves the computed result | Assertions on relevant inputs and failure paths | Passing examples do not prove equivalence for all inputs or side effects |
| This reference is needed after a mutation | The later use in source and a compiler-checked contrast | A lexical scope drawing alone does not determine where a borrow is needed |
| This PR changes the returned error | Matching revisions and a test distinguishing before/after behavior | A test against the current checkout may not describe the reviewed revision |
| This implementation allocates less or runs faster | Allocation evidence or measurements with conditions | Types or the absence of `clone()` do not establish performance |

Copy quoted output from the actual run. Mark omissions and distinguish paraphrased diagnostics from verbatim
text. Attach the command, toolchain version, edition, and relevant features or revision when they affect
reproduction. Use a claim/check/result/limit table when it makes a long document easier to verify.
If tools are unavailable, label predictions and unrun examples instead of manufacturing output.

Use existing tests or `rustdoc --test --edition <edition> <document>` where appropriate. A `compile_fail` doctest
establishes failure to compile, not by itself the intended reason: inspect the diagnostic and check that the
corrected counterpart compiles. Use assertions when the value matters; printing it does not check it. Distinguish
runtime panics from compile errors. See the official
[rustdoc testing guide](https://doc.rust-lang.org/rustdoc/write-documentation/documentation-tests.html).

For repeatable quotation checks, [Checking documents](checking-documents.md) describes an optional Rust CLI
that compares source and output and checks compile-fail error codes with a working correction.

## Make diagrams answer a question the code makes hard to follow

Before drawing, identify the source expressions, entities, and relationships the figure must preserve in a short
list or state table. Choose a table for a few changing values,
a sequence diagram for ordering, or a graph for dependencies when it clarifies the question. Use existing
rendering tools; Mermaid is sufficient for many small diagrams.

For ownership and borrowing, distinguish a binding, its value, and a reference. Label arrows as “owns,” “borrows,”
or “moves.” In an example borrowing a `String`, separate the borrow's last use from destruction of the owned value.
Do not depict a move as guaranteed physical relocation, or lifetime annotations as extending storage duration.
The [Rust Book's borrowing examples](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html)
show why the position of a reference's later use matters.

Mark whether a figure represents source relationships, one observed execution, or a hypothetical path. A single
trace does not show every possible concurrent ordering. Tie labels and edges to source or checked examples;
do not turn an unverified assumption into a factual-looking arrow.

When a renderer is available, inspect the final rendering at the intended display width: follow each arrow to
its endpoint, read the labels, and check that relationships are distinguishable without color alone. Simplify or
split a layout that implies a wrong relationship. If rendering is unavailable, provide a readable text/table
alternative and state that visual layout was not checked. Valid syntax and attractive layout do not establish
semantic correctness.

## Order longer material by prerequisites and decisions

Split material into chapters when the requested depth and prerequisite relationships justify it.
Identify each chapter's decision, prerequisites, new concepts, and example
in a compact outline. Introduce unfamiliar concepts before requiring them, or explicitly identify assumed knowledge.

A borrowing tutorial might first compare passing an owned value with passing a borrow, then locate a reference's
last use, then choose a signature that lets the caller reuse its value. This is an example, not a fixed Rust
curriculum. Refer back instead of repeating earlier explanations. Begin with a small working example when useful.

If exercises are requested, connect each goal to a judgment and reason the learner can demonstrate. For repair
exercises, check that the starter fails for the intended reason and the answer fixes it while preserving required
behavior. Prediction and tracing exercises may start from working code; do not break it just to require a failing
test. Validate answers, separate them from prompts, and respect hints-only requests. Use
[Learning design](learning-design.md) to adjust support.

## Separate document checks from learning evidence

Report source agreement, execution checks, diagram inspection, and learner responses separately. AI review may
suggest missing prerequisites or confusing transitions; simulated reactions and recall remain hypotheses about
the document. They are not a learner's unaided response or a measurement of next-day retention. Use
[Research and scope](learning-evidence.md) for learning-effect claims.
