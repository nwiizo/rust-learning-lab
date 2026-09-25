# Reviewing Rust code

English | [日本語](code-review_ja.md)

Identify defects, mismatches with intent, and the effects of changes with evidence that makes a fix actionable.
Do not edit source for a review-only request. When fixes are also requested, apply and verify them within scope.
Detailed reviews with learning or design explanations can become [review documents](review-document.md).
Follow repository instructions and any requested output format.

## Establish scope and assumptions

Read the specified diff, files, or snippet, then inspect purpose, callers, and relevant tests as needed.
In diff reviews, concentrate on problems introduced by the change; do not attribute pre-existing problems to it.
For a snippet, do not invent unseen callers or requirements. Check the edition, toolchain, and dependency type
definitions when they affect the conclusion. If an unknown comparison base prevents identifying the diff,
ask only about that uncertainty while continuing useful inspection.

## Trace values across boundaries

Follow changed conditions from input to output. These are perspectives to select from, not a mandatory checklist.

- **Arguments and ownership:** swapped same-type arguments, unintended moves or copies, reference depth, and caller effects.
  Neither `clone()` nor references alone establish a defect. Judge ownership needs, data size, and measurable costs.
- **Branches and failure:** tail expressions, return values, where `?` returns, reachable `unwrap()` calls, discarded errors,
  and boundary inputs. Check whether requirements permit behavior such as dropping invalid inputs with `filter_map(Result::ok)`.
- **Iterators and side effects:** laziness, short-circuiting, how much input is consumed, order, and item types.
  Check whether a loop rewrite changes execution count or work remaining after failure.
- **State and concurrency:** borrow and lock-guard scopes, holding state across `.await`, and state left after cancellation.
  A guard across `.await` does not by itself prove deadlock; identify the lock type, runtime, and waiting relationships.
- **Observable behavior:** public APIs, input formats, units, errors, output order, and required performance conditions.
  For `unsafe`, check safety requirements and caller responsibilities; successful compilation does not prove safety.

## Substantiate findings

Identify the mismatch using concrete input, a call path, or a type mapping. Where possible, confirm with existing tests
or a small reproduction. Keep execution using user data, credentials, or configuration separate from ordinary verification;
check necessary authorization and isolation. Do not modify production data for an explanation.

For unexecuted findings, distinguish static evidence from unverified conditions. Separate API rules from author intent;
do not invent reasons absent from documentation or implementation. A warning, hypothetical future requirement,
or personal preference alone is not a defect. When essential information is missing, state it briefly as an open point,
separate from established findings.

## Make findings actionable

Order by impact and combine findings with the same cause. Connect location, triggering conditions, observable impact,
evidence, and the smallest correction. Include relevant line numbers for actual files; do not invent filenames or line
numbers for snippets. Base severity on real impact, distinguishing defects from optional style improvements.

For beginners, explain syntax directly involved in the finding. For `?` inside a closure, trace the success value,
the closure exited on failure, and the outer operation receiving the result. Go beyond “returns an error” to show
whether it reaches the caller or is discarded on the way. Return to the finding and fix after explaining the syntax;
do not bury the problem beneath unrelated instruction.

Prefer the smallest fix that meets the conditions. Add alternatives only when behavior or cost differs.
State what was inspected and what remains uncertain. If no problems are found, say so; do not equate that with universal
safety or manufacture findings to fill a quota. Add an optional transfer question or small modification only when learning
is an explicit goal.
