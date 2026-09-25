# Review documents worth revisiting

English | [日本語](review-document_ja.md)

Use for detailed reviews that teach from repository changes, explanations of design intent, or requests for a document.
Connect user-visible changes to the flow of values that implements them, rather than merely saving chat findings.
Writing a review document does not authorize source changes, commits, or pushes.

## Destination and scope

Honor the requested path. Otherwise follow existing documentation conventions with `docs/rust-learning-review.md`
or a descriptive path such as `docs/<topic>-learning-review.md`. Do not write files for chat-only, read-only,
or no-documentation requests. If writing is unavailable, provide the text in conversation without claiming it was saved.

If a document for the same purpose exists, read its contents and modification state before updating it.
Do not overwrite a record of another change or the user's in-progress writing; choose another name instead.
Make the title describe the change or design decision, rather than only the skill name. Record the relevant files,
diff, or version near the beginning or end. Label uncommitted changes as such. Do not invent PR numbers or identify
a release that has not been created.

## Organize the reading path

Use this sequence as a starting point, selecting the sections the subject needs rather than filling every heading.

1. **Purpose and findings:** user-visible changes and important findings. Put problems before the teaching material.
2. **Overall flow:** connect code by how values are created, selected, passed, borrowed, and returned.
3. **Decisions and syntax:** use actual code and the relevant explanation perspectives below.
4. **Try another condition:** when learning is the goal, separate a short optional question from its answer and reasoning.
5. **Sources and verification scope:** implementation links, checks actually run, results, and unverified points.

For each code location, explain what it needs to distinguish, prevent, or accomplish. Then connect declaration
and call and read the symbols, parameters, and return value. Separate declared argument order from API design;
do not confuse type information such as lifetimes with runtime arguments. Expose inference, implicit borrowing,
and reference depth where needed. Use tables when there is an actual comparison: caller to parameter, before to after,
or success to failure.

Keep syntax connected to purpose. Explain `&[T]` not just as notation but as a decision to preserve the caller's values
and avoid modifying them here. When comparing alternatives, identify the conditions where each fits.
In long documents, refer back rather than repeating basic explanations.

## Make examples trustworthy

Use links relative to the saved document and verify their targets and described content. Do not embed private absolute
paths such as a user's home directory in public documents. Distinguish implementation excerpts, simplified executable
examples, and intentional compile failures. State omitted error handling or external dependencies. Support claims
that a simplified version behaves identically to the implementation.

For `rustdoc --test --edition <target-edition> <document>`, mark non-standalone excerpts as `rust,ignore` and intentional
failures as `compile_fail`. Do not change a failing executable example to `ignore` merely to make the checks pass.
An excerpts-only document has not been verified by execution. Before running examples, check for file, network,
credential, or other side effects.

Record commands, toolchain, passes, failures, and exclusions. Separate past application test results from documentation
checks run now. Say “checked against the source” for static inspection; do not imply execution of behavior that was not tested.

When useful for explanation density and continuity, read the [short review example](review-example.md).
Its headings and length are not a required template. Return important findings and the document link in chat when finished.
