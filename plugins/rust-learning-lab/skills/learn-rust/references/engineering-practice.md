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

## Read one user-visible path through unfamiliar code

Start with an actual input and expected outcome, then locate its entry point, one successful path,
and one relevant failure path. Avoid reading every module before establishing what the program does.
Use a small map when it resolves uncertainty:

`input → parsing → validation → domain operation → external effect → reported result`

For each arrow, inspect the Rust type, who owns the value, and how failure crosses the boundary.
For example, an input `String` may be borrowed as `&str`, parsed into `u32`, validated into a quantity,
and moved into a command. Explain which step changes representation and which establishes a business rule.
Follow `?` to the enclosing function or closure and then to the caller that decides what the user sees.

| Question while reading | Evidence to inspect |
|---|---|
| Where does this method come from? | Actual receiver type, trait imports, implementation, and relevant macro expansion |
| Why does this value stay borrowed? | Lifetime relationship, retained fields, reborrows, and later uses |
| What does this failure mean? | Error variants, conversions, call sites, and effects completed before the return |
| Why is this apparent duplication present? | Different callers, compatibility needs, relevant tests, and change history |
| Which part of my model is uncertain? | A concrete unknown and the smallest observation that could resolve it |

Use editor type hints and definition navigation to make implicit information visible; use a targeted
compiler diagnostic or test to check a hypothesis. A name or old diagram suggests a model, not proof.
Existing tests record selected expectations and may preserve an old defect. Separate observed behavior,
intended behavior, and an unverified explanation of why the code was written that way.

## Design types to make a caller's next decision clear

Prefer a distinction that helps the caller over a sophisticated implementation hidden behind an ambiguous API.
An `enum` can replace independent flags when only specific combinations are meaningful. A private-field
newtype with a validating constructor can give `Quantity` a narrower meaning than `u32`.
The [quantity example](worked-examples.md#validation-gives-a-type-a-narrower-meaning) separates parsing from
validation and shows where that condition is established. Do not claim that naming a wrapper proves its
contents valid; inspect constructors, mutation, and deserialization.

Borrowed versus owned inputs should express the required use. A parser that only reads text can accept
`&str`; a command queued beyond the input owner's lifetime needs suitable owned data or another justified
lifetime arrangement. Start from that requirement instead of treating either cloning or lifetime parameters
as a design failure.

Evaluate an abstraction at the call site, using questions adapted from
[Cognitive Dimensions](https://www.cl.cam.ac.uk/~afb21/CognitiveDimensions/CDtutorial.pdf):

| Reading or changing the API | Rust-oriented question |
|---|---|
| Visibility | Can the caller find the inferred types and relevant bounds together? |
| Hidden dependencies | Are required trait imports, feature flags, and runtime assumptions discoverable? |
| Change effort | Does changing one validation rule require coordinated edits across callers? |
| Early decisions | Must a learner design a generic hierarchy before trying one concrete operation? |
| Recognizable roles | Do names and types distinguish a cursor from a count, and accepted work from completed work? |

These questions expose tradeoffs, not a quality score. A helper with many generic parameters can hide
simple work; repeated validation can hide a rule that belongs at one boundary.
Use a trait when a real interchangeability or testing need benefits from it; a function, closure, enum,
or concrete struct may be enough. Neither fewer lines nor more generic bounds is a universal quality measure.

Comments and tests have different jobs. A test can demonstrate a failure case; a safety comment explains
why an unsafe operation's preconditions hold for all permitted callers. Retain comments about units,
ordering, compatibility, and decisions that code alone cannot communicate. Do not convert “prefer clear
code” into “remove explanatory comments,” or treat a test suite as proof of an unsafe abstraction's soundness.

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

Separate tests of a decision from tests of an adapter. A pure validation function can have small boundary
tests; a database adapter needs evidence about the real database behavior on which the claim depends.
A fake that returns `Ok` verifies only the caller's response to that chosen result. Coverage records code
executed, not whether assertions reject the relevant wrong behavior. Preserve meaningful public behavior
during refactoring; do not mechanically preserve a mock's internal call sequence unless order is required.

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

## Turn tradeoffs into conditions and observations

Avoid stopping at “it depends.” Name the user need, constraint, options, selected reason, and condition
that would justify reconsideration. For example: a sequential implementation may be easier to maintain
while meeting the measured workload; adding asynchronous work needs an actual waiting/concurrency problem,
not merely a more advanced-looking API.

| Concern | Rust-oriented question |
|---|---|
| Performance | Is time spent in parsing, copying, allocation, locking, a query, or queueing? Measure representative inputs before selecting the optimization |
| Operability | Can an operator distinguish invalid input, temporary unavailability, and an outcome that is still unknown? |
| Compatibility | Can older clients and stored data still be read after this public type or format changes? |
| Usability | Do error messages explain the affected input and next action, while preserving machine-readable error distinctions where needed? |
| Deployment | Do the intended targets, features, edition, and minimum toolchain match what was actually tested? |

Model only the relationship needed for the question: an ownership sketch for a borrow, a state transition
for an enum, a sequence for external effects, or a component view for dependencies. Label arrows so that
“calls,” “owns,” and “sends” are not confused. A diagram should answer a question rather than replace
inspection of the code.

For data and concurrency, continue with [System boundaries](systems-thinking.md). A successful local test
does not establish production readiness, and a Rust memory-safety argument does not establish data
freshness, durable processing, or appropriate use of user data.

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

For continuing study, choose a capability relevant to the user's work: tracing an unfamiliar request,
explaining an API boundary, diagnosing a failure, or reviewing a small change. A bounded cycle of reading,
prediction, observation, and explanation is more concrete than collecting new tools or completed chapters.
Offer tools only when they remove a demonstrated obstacle. Do not attach career predictions, productivity
multipliers, or claims of durable learning to one exercise or to using AI.

In a review explanation, connect a finding to observable impact and an actionable change. Avoid judging
the author's ability from unfamiliar style. The learner should be able to explain to another person what
can fail and why the proposed change helps, without reciting compiler terminology.

## Leave enough information to resume

At natural breaks in a long exchange, leave a brief conversational note when useful. Prioritize the goal,
assumptions, verified facts, unknowns, and next check rather than repeating the log. Preserve reasons for decisions
and conditions under which they should be revisited.

For interrupted code investigation, include the function or location, current hypothesis, and next observation
with its expected result. Keep only enough detail to restart that activity, not every attempted command.

For example: “Chose `&str` because the caller reuses the string. Reproduced the move error. The learner explained
why the borrowed version remains readable; storing the value inside the function has not been covered.
Next compare a case requiring ownership.”

Record failed approaches with the conditions in which they failed; do not generalize them into rules such as
“never use `clone()`.” Update notes when assumptions change and remove stale or duplicate advice. Distinguish
explanation given, correct with assistance, and independently verified. Save learning history only when requested.
