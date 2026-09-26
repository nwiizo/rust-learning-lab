# Choosing learning support for Rust

English | [日本語](learning-design_ja.md)

Use for difficulty learning, practice, revision, learning to debug, or ongoing study. Do not pack every method into one answer.
Assess reading, decomposition, implementation, testing, debugging, and retention separately, then choose useful support.
These are applications to Rust informed by education research; this combination itself has not been evaluated.
For research findings and limitations, see [Research and scope](learning-evidence.md).

## Match support to the difficulty

These observations help choose explanations; they are not a classification of a learner's ability.

| Observed difficulty | First support to try | Rust example |
|---|---|---|
| Unfamiliar notation or terms | Put a short definition next to a minimal example | Distinguish the `&` in `&str` from the expression `&value` |
| Unknown callee or inferred types | Show definitions, types, and argument mappings together | Map `fold`'s initial value to closure parameters and return value |
| Understands lines but cannot trace results | Write state changes down | Tabulate each iteration's accumulator and current item |
| Consistent prediction error | Compare successful and unsuccessful cases | Contrast reusing `i32` and `String` after passing them by value |
| Can read but cannot start writing | Supply input/output examples, subgoals, and partial implementation | Separate examining input, selecting elements, and returning a result |

When difficulties overlap, supply the prerequisite needed to proceed. Do not assume a typo reveals a conceptual error.
Start with what the user's explanation and code show; ask only about unknowns that change the answer.

## Choose one activity before choosing a method

Someone experienced in another language may know algorithms but need help with Rust ownership; someone who
understands a signature may still need help navigating a new codebase. Match support to the current task,
not a permanent label such as beginner or expert.

| Current activity | Useful starting point |
|---|---|
| Locate information | Show the definition, inferred type, or relevant API documentation; do not turn a missing fact into a quiz |
| Understand existing behavior | Trace one input through one path, naming the role of each group of operations |
| Implement an understood plan | Map the plan's values and steps to Rust types and expressions |
| Extend existing behavior | Identify the changed condition and the behavior that must remain |
| Explore an uncertain design | Compare small alternatives against one concrete need before choosing a general abstraction |

For onboarding, begin with one activity, such as finding where an error becomes a user-visible message.
Avoid simultaneously requiring unfamiliar navigation, domain rules, syntax, and a feature implementation.
Looking up an infrequent API is normal; reserve optional recall practice for recurring decisions that interrupt work.

## Reduce the burden of reading

- Express one target capability, such as explaining why a variable remains usable after a call. Return to that target
  after filling prerequisites. Do not turn a syntax question into unrelated architecture or optimization. Connect unfamiliar
  syntax to what it represents and why it is needed, without piling up new concepts to explain the background.
- Keep small inputs, names, and operations stable initially, changing only the condition being taught.
  Avoid combining generics, async code, and custom traits as simultaneous new concepts.
- Read meaningful groups: unpack symbols and expressions, then name roles such as filtering, transformation, or aggregation.
  Connect `acc` and `item` to accumulator and current item, but verify names against behavior. Label subgoals in worked
  examples and distinguish reusable decisions from incidental values. Do not stop at translating individual lines.
- Externalize difficult state using only useful columns: operation/before/after for values, or variable/type/owner/referent/later
  usability for ownership. Separate compile-time constraints from runtime changes. Connect where values reside to permitted
  reads, writes, and moves. Do not claim every borrow-checker rejection would necessarily produce a runtime bug.
- Expand long expressions in an explanation or learning copy while preserving correspondence. Introducing a binding
  can change borrowing and temporary destruction; check evaluation count, side effects, and resource scope before
  calling a rewrite equivalent. Use the [edition contrasts](rust-2024.md) where relevant. For metaphors such as
  ownership as boxes, state their limits and return to actual types and compiler rules.

Connect three levels: the concrete trace, the rule it illustrates, and a changed case where that rule helps.
For example, trace a `String` argument moving into a function, explain the non-`Copy` ownership transfer,
then contrast passing `&str` when the caller needs its text again. Return to the user's original expression.
Neither abstract terminology alone nor an endless series of examples supplies this connection.

Distinguish a variable's type from its role. Two `usize` values may be a current position and a count of matches;
show which operations update each. Names are clues that need checking. Familiar groups are learned through
use and explanation, not by imposing a fixed number of symbols a person should hold in memory.

## Replace a misleading prior model

When the learner explains a consistent prediction error, identify the rule they used before adding more detail.
A rule from another language may be useful in some cases and misleading in others. Do not infer it from a typo.

For “assignment makes an independent copy,” contrast `i32` and `String` in the same small use-after-assignment
example. Give the replacement rule: the value's `Copy` implementation determines copying versus moving;
independent duplication uses the type's actual duplication operation. For “`&T` means nothing can change,”
separate ordinary shared access from interior mutability. See [Expert reading](expert-reading.md).

Make the counterexample's observation and the replacement explanation explicit; “that is wrong” leaves no
usable alternative. If practice was requested, try a changed case with less help and revisit it later when useful.
A correct immediate answer does not establish that the earlier model will never return. Teach a precise rule
with its conditions rather than replacing one slogan with another.

## Move from explanations to independent practice

Answer ordinary questions with a conclusion and explanation. If practice helps, add a short optional question and
separate its answer into another paragraph or a disclosure block. Wait for answers only when interactive practice is requested.
Honor preferences such as “hints only” or “show the answer too.”

- **Predict:** before execution, predict output, a type, or whether compilation succeeds, and give a reason. Do not invent
  runtime results for non-compiling code; locate and explain the error. Running, investigating, modifying, and creating can
  follow, but every step is not mandatory every time.
- **Explain:** describe in your own words where a value comes from and where it goes in one line of a worked example.
  Distinguish reciting terminology from applying it to the code.
- **Complete:** if writing everything is too difficult, use blanks or a short line-reordering task. Provide prerequisites and
  expected behavior; allow multiple valid solutions. Success at reordering does not prove independent writing ability.
  Next reduce choices or scaffolding and ask for a small independent change.
- **Modify:** after tracing an example, vary an input or condition and explain the result. Check why an approach applies
  and where it stops applying, rather than only memorizing syntax.
- **Repair:** locate the first failed prediction and compare expected and actual types or values. Give the correct result
  and reasoning, then check whether the revised reasoning applies to another small example.

Worked examples, partial completion, and independent changes are adjustable support, not a fixed progression.
Make hints more concrete when stuck and reduce support when successful. More questions are not a substitute for progress.
Report demonstrated understanding and unknowns, not judgments about personality or talent.

## Decompose a problem and write code

- If starting is difficult, establish concrete input and expected output, then describe how to solve it by hand.
  Identify values, branches, and repetition and map subgoals to code. Avoid requiring too much algorithm design and unfamiliar syntax at once.
- Implement one behavior and check a small input. Then extend to relevant empty or boundary cases. Establish expected
  results before execution rather than adapting the answer to observed output. Do not impose a fixed test-first ritual or a large test suite.
- Connect independent work to a small part of something the user wants to build. For aggregation, limit the task to producing
  a result from known input; do not add unrelated setup, dependencies, or UI work.
- After success, change conditions rather than just names or numbers: for example, replace a total with the total of qualifying
  values. Explain what can be reused and what needs new reasoning. Separate reproducing an answer from transfer to a new problem.

## Learn debugging and feedback

- Separate compiler errors, runtime failures, and unexpected results. Establish reproducing input and expected behavior.
  Trace back from the reported line to where the value or type was determined. Teach tool operation separately from causal reasoning.
- Connect observation, hypothesis, and the operation that tests it. To investigate accumulation, inspect values at each
  iteration and locate the first divergence. Do not change unrelated locations without evidence.
- Respond to an attempt with what works, the first mismatch, and one next investigation. Adjust hints from where to look
  toward concrete operations; return to an explained example after repeated difficulty. Do not withhold a requested answer.
- Recheck the failing example after fixing it and try relevant additional input. Connect cause and correction rather than
  stopping at passing tests or accepting a compiler suggestion. If AI applied the fix, assess its behavior separately from
  whether the learner can repair it independently.

## Retrieval and spaced revision

Experiments with prose materials found that immediate performance or confidence after rereading did not align with
later recall. Use this as a reason to offer retrieval without the explanation, rather than relying on immediate agreement.
Do not claim the same effect for every Rust skill.
Source: [Roediger & Karpicke, 2006](https://journals.sagepub.com/doi/10.1111/j.1467-9280.2006.01693.x).

- Ask for decisions and reasons, not only definitions: instead of just “What is borrowing?”, ask whether the original value
  remains usable after a call and which type supports that conclusion.
- If flashcards are useful, put one decision on each, with a short answer, reason, and applicability condition on the back.
  Keep recurring syntax and decision cues, not slogans or whole-program memorization.
- In ongoing study, revisit a task later without the explanation, ideally with changed input or context. The next day,
  a few days later, and a week later are starting suggestions, adjustable to performance and purpose, not a universal optimum.
  Do not automatically create reminders or scheduled events.
- Check reasons as well as correctness. If confidence and results differ, revisit assumptions. Immediate repetition does
  not guarantee lasting retention or transfer. No response, or paraphrasing after seeing the answer, is not independent success.

Spacing research found that useful intervals depend on the desired retention period. The schedule above is not a fixed
Rust-validated plan. Source: [Cepeda et al., 2008](https://pubmed.ncbi.nlm.nih.gov/19076480/).

## Make resuming easier

Use a short conversational resumption note when useful, following
[Engineering practice](engineering-practice.md#leave-enough-information-to-resume).
Keep explanation given, success with help, and independent understanding distinct; save learning history only when requested.
