# Learning judgment and verification

English | [日本語](engineering-practice_ja.md)

Use for AI-assisted projects, learning specifications and tests, changing existing code, or ongoing study.
Do not attach this workflow to a simple syntax question. These are practical exercise-design guidelines;
the educational effect of this combination has not been measured.

## Define the intended capability

Separate a working product from the learner's goal. “Can sum numeric strings” and “can explain where `?` returns
when parsing fails” need different checks. Match the exercise to the difficulty: syntax, tracing, decomposition,
implementation, or verification.

For a small exercise, the following level of detail is enough. Do not ask again about conditions already supplied.

- Input `["2", "3"]` should produce `5`; an invalid element should produce an error.
- Read the input without changing it so the caller can reuse it. An empty collection should produce `0`.
- After implementation, predict results for empty and invalid input, explaining the answer from types and branches.

If correct behavior admits multiple interpretations, surface the decision that matters. Do not silently choose
an unspecified overflow policy to suit generated code. A brief teaching example can instead state its scope,
such as “only small values are considered here.”

## Develop the ability to check correctness

Derive expected results from requirements or hand calculation, not from the generated implementation.
Compilation checks constraints such as types; tests check behavior under selected conditions. Neither alone
guarantees that the intended purpose is met.

When learning tests, select relevant ordinary, empty, boundary, and failure inputs. Ask what mistake could still pass.
For example, testing only empty input will not reject a constant-zero implementation. Deliberately breaking code
belongs in a learning copy, not the original files. Do not require tests or specialized tools for every question.

When exact answers are difficult to enumerate, use relationships between runs. For example, reordering integer
inputs should not change a sum within a range where overflow cannot occur. Do not apply this blindly to floating-point
addition or order-sensitive operations. Such a property can still accept a constant result, so pair it with concrete expectations.

## Change small pieces and preserve necessary behavior

Before a change, identify observable inputs, returns, errors, and whether input is modified. In a loop-to-iterator
exercise, compare results and failure behavior for the same inputs, not just syntax. Changes such as replacing
`fold` with `try_fold` affect early termination and are not merely different notation.

Matching types do not guarantee matching meaning. Two functions accepting `&str` can differ in whitespace handling
or units. Separate behavior callers need from implementation details free to change. If side effects are observable,
check their order and frequency too. Keep a change small enough to examine one hypothesis instead of rewriting
several behaviors the learner cannot yet explain.

Before removing restrictions, duplication, or validation, inspect callers, tests, comments, and available history
as needed. Appearance alone does not establish redundancy. If the reason is unknown, say so rather than inventing
plausible background. Revisit old expected results and notes when requirements change; past passing tests do not
establish correctness under new assumptions.

## Adjust AI assistance

Provide complete code when requested. For learning, select an opportunity for the learner to make one decision:
predict a failing input without the answer, explain why borrowing fits, or implement a changed condition.
Choose the target skill, and honor requests for hints only.

Asking the same AI whether its answer is correct is not an independent check. Compare with expectations derived
from requirements, compiler diagnostics, execution, or official type definitions. Make unverified parts visible
and adjust assistance or example size. Do not require launching another agent.

Reading an explanation and agreeing with it does not establish independent verification ability. Conversely,
one wrong answer does not establish lack of ability; locate the mismatch among prediction, clues, and observation.
If repeated prompt rewriting is not helping, return to minimal input, types, or the error and choose one next check.
Do not leave learners stuck indefinitely or make failure itself the goal.

## Leave enough information to resume

At natural breaks in a long exchange, leave a brief conversational note when useful. Prioritize the goal,
assumptions, verified facts, unknowns, and next check rather than repeating the log. Preserve reasons for decisions
and conditions under which they should be revisited.

For example: “Chose `&str` because the caller reuses the string. Reproduced the move error. The learner explained
why the borrowed version remains readable; storing the value inside the function has not been covered.
Next compare a case requiring ownership.”

Record failed approaches with the conditions in which they failed; do not generalize them into rules such as
“never use `clone()`.” Update notes when assumptions change and remove stale or duplicate advice. Distinguish
explanation given, correct with assistance, and independently verified. Save learning history only when requested.
