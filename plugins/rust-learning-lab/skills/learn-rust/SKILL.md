---
name: learn-rust
description: Explain Rust syntax, code, and compiler errors through their purpose, types, and ownership, and support focused practice. Review Rust code and diffs with concrete findings, impacts, and understandable fixes. Create educational review documents from PRs or commits. Do not add learning exercises to implementation-only requests.
---

# Learn Rust

English | [日本語](SKILL_ja.md)

Help users read syntax and reason independently about the next problem using types and compiler diagnostics.
Lead with the answer, then give the small examples and reasons needed to understand it. Do not force a fixed outline.

## Choose the language and depth

Honor explicit language preferences first; otherwise infer the language from the latest request and conversation.
English code, errors, or quotations alone do not switch the conversation's language. For documents, follow the requested
language or existing document and repository conventions. Japanese conversation and an English README can coexist.
Do not ask about language when context is clear. Preserve identifiers, API names, and diagnostic text where useful.

References have English `references/<topic>.md` and Japanese `references/<topic>_ja.md` versions with reciprocal links.
Read only the relevant version: Japanese for Japanese explanations or documents, English for English ones and as the
fallback for other languages. Respond in the user's language even when using the English fallback. For mixed-language
work, choose by the current output. Do not load both translations unless comparing them is the task.
`SKILL_ja.md` is a translation for readers, not an additional skill or a required second instruction load.

For beginners, do not skip symbols or implicit behavior needed to understand the answer. The perspectives below are
options, not a checklist for every response. Narrow the difficulty from the question and code, skip familiar material,
and start with a small beginner-friendly explanation if knowledge is unclear.
For explanation requests, prioritize reading code; edit source only when implementation or fixes are also requested.

## Use as a reviewer

For code or diff reviews and change-safety questions, read [Code review](references/code-review.md) and lead with
findings and evidence. Use syntax, types, and ownership to explain why a problem exists and what the fix changes.
Explain necessary notation to beginners; do not repeat prerequisites to experienced readers.
Do not turn an ordinary learning question into a review or exercise.

For detailed reviews or design explanations using repository changes, PRs, or commits as learning material, also produce
Markdown following [Review documents](references/review-document.md). Resolve the requested source and record the compared
revisions. Default to separate files in `docs/rust-learning-review/`; maintain its English and Japanese README indexes
on every document creation or update. Exclude the directory from Git and routine agent discovery, and read it only for
requested learning work or document maintenance. Respect explicit destinations, chat-only requests, and read-only constraints.
Do not automatically create documents for brief findings or standalone snippet questions.

## Support understanding and retention

Distinguish following an explanation from independently predicting, explaining, implementing, or repairing code.
Use programming education research to select practice for the target skill and current difficulty.
Do not assume results from other languages or classrooms transfer unchanged to individual Rust learning.

- Distinguish unfamiliar syntax, missing definitions or types, difficulty tracing state, and misapplied prior knowledge.
  Choose definitions, visible information, state tables, or contrasting examples. Do not ask again for facts already provided.
- Avoid stacking new concepts. Connect syntax, types, and values in the same example; label meaningful groups such as
  transformation or accumulation and connect lines to their purpose. Externalize hard-to-track state in a small table or note.
- When deeper learning is useful, choose recall without the explanation, self-explanation, or a small modification.
  Answer first when an answer or fix is requested. Exercises and revision are optional; wait only in requested interactive practice.
- For mistakes, show where prediction diverges from the result and test a small changed condition. Do not equate
  one correct answer or “I understand” with retention; adjust support using re-explanation and application to another example.
- When the user cannot start writing, work from input/output examples and small subgoals. In debugging, connect
  observations, hypotheses, and checks. Assess understanding through reasons and a small unaided change, not compilation or passing tests alone.

For difficulty learning, practice, revision, learning to debug, or ongoing study, read the relevant parts of
[Learning design](references/learning-design.md). For claims about learning methods or effects, read
[Research and scope](references/learning-evidence.md). For AI-assisted projects, learning specifications or tests,
alternative implementations, or resuming study, use [Engineering practice](references/engineering-practice.md).
Establish inputs, outputs, and conditions to preserve; assess working code and learner understanding separately.

## Teach from the provided code

- For errors or bugs, identify the problematic expression or type and the smallest fix first. Add alternatives only when their tradeoffs differ.
- Go beyond translating the error: explain what happened, what Rust prevents, and what the fix changes.
- Choose moves or borrows from the user's intent. Do not categorically reject `clone()`; distinguish the need to own a value,
  the amount copied, and allocation. Show whether passing a value moves it or copies it through `Copy`, and whether the original binding remains usable.
- When inference or iterators are confusing, expose types and reference depth at relevant stages. Explain related concepts without repeating an ownership lecture.
- Paraphrase terminology briefly. Include internals, advanced details, and understanding checks only when needed or requested.

## Read syntax and explain its purpose

- State what the code does, then unpack the syntax and background needed to understand it. Explain keywords, symbols,
  and expressions before reading the whole construct in plain language. Identify a function's name, parameters and types, return type, and body.
- Explain the problem a feature solves, information it gives the compiler or caller, and the difference if omitted or replaced.
  Use small contrasts instead of stopping at “that is the rule.” Do not confuse practical benefits with historical reasons
  for a symbol or an author's intention. Check official material or RFCs before claiming adoption history.
- Show how position changes a symbol's role: `&T` is a reference type, while `&value` borrows a value. Connect the role
  to types and ownership. Distinguish mutable bindings from `&mut T`, and shadowing from assignment, where relevant.
- Do not introduce necessary notation such as `::<T>`, `|x|`, `?`, or `'a` without explaining what it specifies, receives,
  or returns in this code. Definitions alone are insufficient.
- Use small contrasts for questions about semicolons, patterns versus expressions, and similar distinctions.
  Stay with relevant syntax instead of listing the language's syntax or repeating known basics.

## Parameters, arguments, and order

- Put the signature next to the call and map values to parameters. Explain each role, type, and whether it is passed by value, shared reference, or mutable reference.
- Map `self`, `&self`, or `&mut self` to `value` in `value.method(arg)`. Distinguish the receiver from arguments
  inside parentheses, and explain automatic borrowing or dereferencing when needed.
- Distinguish a closure passed to the outer function from the arguments it receives when invoked. Show who supplies
  the values: `fold` supplies the accumulator and current item to `|acc, item|`. Separate captures from parameters;
  do not confuse `move` with invocation count or thread execution.
- For “why this order,” distinguish following the declaration from why the API chose it. Separate language constraints,
  conventions, and API choices; separate documented design reasons from inference. Do not confuse argument positions
  with evaluation or execution order. Show semantic consequences even when swapped arguments share a type.

## Return values and control flow

- Trace concrete inputs to returns. Where relevant, explain `-> T`, tail expressions, `return`, `;`, and `()`:
  which expression returns to which caller, rather than printing output or mutating an existing value.
- Name intermediate values in method chains and map input/output types to operations. Show iterator item types,
  reference depth, when closures run, and what drives iteration. Do not infer ownership from `into_iter()` alone; inspect the receiver and implementation.
- For `match`, `if let`, or destructuring, distinguish the matched value, pattern, and bound variable types.
  Separate values wrapped in `Some` or `Ok` from extracted values, and explain what happens if a branch is not taken.
- Trace absence and failure as well as success for `Option`, `Result`, and `?`. Identify the function or closure exited,
  the required return type, and error conversions in the actual code.
- Separate constructing a future from driving it. Do not describe `.await` as spawning a thread or automatically running work in parallel; use the actual runtime context.

## Types, names, and implicit information

- Explain where inference gets information: arguments, assignment targets, return types, or later uses. Separate generic
  arguments from runtime values; for `collect::<Vec<_>>()`, explain what is specified and what is inferred.
- Substitute concrete types for `T`, bounds, `where`, or associated types and show which operations the constraints enable.
  Separate argument-position and return-position `impl Trait`; compare with `dyn Trait` only when useful to the question.
- Locate unfamiliar functions or methods in the standard library, dependencies, or project. Distinguish `::` and `.`,
  associated functions and methods, and trait-provided methods; include necessary `use` declarations. Macros marked `!`
  have their own input syntax, so ordinary function argument rules do not automatically apply.
- For lifetimes, first trace who owns the value and when references are needed. Explain that `'a` describes relationships
  between valid reference lifetimes; annotations do not extend a value's lifetime.

## Examples and verification

Choose the expression, function, or program size needed for the question. Do not add dependencies or restructure the
user's project just to teach an example. Label intentionally non-compiling examples. When expanding or rewriting code,
preserve evaluation count, ownership, and side effects. Label differences in pseudocode that is not strictly equivalent.

When exact diagnostics or behavior matter, check with an available compatible toolchain. Verify API descriptions against
the target version's definitions or official documentation. Do not invent design intent. Distinguish executed code from
unrun illustrations and observations from predictions. Explain the panic possibility in teaching uses of `unwrap()` or
`expect()`, and connect to `match` or `?` when appropriate. Do not suggest `unsafe` as an easy escape from an error.
Use [Worked examples](references/worked-examples.md) for contrasts in argument order, ownership, and failure paths.
Adapt them to the question rather than treating their wording as a fixed answer template.

## Editions

Separate editions from compiler versions. When relevant, check `Cargo.toml` and `rustc --version`, including workspace
inheritance, dependency versions, and features as needed. Match examples to the target; do not suggest unavailable APIs
or syntax. Do not change a project's edition merely to explain it.
Read [Rust 2024](references/rust-2024.md) when edition-specific behavior or migration is relevant.
