# Conversation evaluation scenarios

English | [日本語](evaluation_ja.md)

Use these tasks in a fresh host conversation before releases or after instruction changes.
They describe expected behavior, not completed evaluation results. Do not score matching headings or wording.
Record the host, model, plugin version, input, output, and tools used; check technical correctness and fit to the request.
Use a temporary learning directory for tasks involving file changes.

| Example request | Observe | Problematic behavior |
|---|---|---|
| Why this order in `fold(10, \|acc, item\| acc - item)`? | Separate the outer initial value from closure arguments, trace roles and values, label unsupported design explanations as inference | Determine roles from parameter names alone, or use only addition to imply arguments are interchangeable |
| Explain E0382 after passing a `String` | Contrast passing by value and borrowing; choose a fix from the caller's intended reuse | Always forbid cloning, or describe every borrow-checker rejection as necessarily unsafe at runtime |
| Explain `?` as briefly as possible | Explain the success value and failure return destination at the necessary depth | Require a lecture or quiz |
| Help me practice summing numeric strings, hints only | Establish inputs and failure behavior, give a useful hint, then wait | Immediately provide completed code or adjust expected values to match output |
| AI wrote this function and tests pass. Does that mean I understand it? | Separate behavior from understanding; suggest predicting or explaining another condition | Declare mastery from passing tests, or categorically ban AI |

## Japanese wording and optional tooling

| Example request | Observe | Problematic behavior |
|---|---|---|
| Explain this reference in readable Japanese; I do not know the terminology | Name the actual reference and referent, explain the operation, then introduce the useful term | Stack abstract nouns, leave “this” ambiguous, or replace every technical term with a metaphor |
| Shorten the explanation “these inputs passed; other inputs and Rust 1.85 are untested” | Preserve the tested scope and the unverified minimum toolchain | Turn the result into “works on Rust 2024” or a general safety claim |
| Improve this Japanese guide while keeping its comparison table and API names | Clarify causes and modifiers where needed; preserve useful structure, identifiers, and certainty | Rewrite every heading or list to one template, or change technical names to avoid repetition |
| Write a Japanese Rust explanation and suggest a wording tool; do not run Python | Suggest Suiko with a relevant purpose or command, use the installed CLI when checking prose, and report actual checks | Run Python, auto-install an unrequested tool, append repeated recommendations, or claim an unrun check |
| Suiko exited successfully but returned findings and reading-load warnings. Must I remove them all? | Inspect both result lanes and the surrounding prose; retain justified technical wording | Equate exit status with zero findings, treat warnings as proof of error, or claim an authorship score |

## Symbols as a reading barrier

Use each request separately. Check meaning, practical value, and a concrete contrast, not merely terminology.

| Example request | Observe | Problematic behavior |
|---|---|---|
| I cannot read `&` or `'a`. Explain `fn first<'a>(left: &'a str, right: &str) -> &'a str` | Map ownership and the returned borrow to the first input; contrast an elidable single-input signature with this two-input one | Claim annotations extend lifetimes, force equal owner lifetimes, or are always required |
| Why `*` in `let r: &mut i32 = &mut n; *r += 1;`? Is it multiplication? | Follow the reference to the integer being changed; distinguish mutable binding, mutable reference, and multiplication | Only say “dereference,” require `mut r`, or claim `*` always copies or clones |
| Explain `std::num::ParseIntError`, `text.parse::<u32>()?`, and the alternative `match` with `->` and `=>` | Separate paths, type arguments, and runtime arguments; show inference, both result paths, and which construct returns | Treat `::` as a call, say turbofish is always mandatory, confuse arrows, or silently discard errors |
| What do the bars do in `fold(10, \|acc, item\| acc - item)`? | Explain passing an operation, who supplies each argument, and why replacing the closure with a tuple changes meaning | Name “closure” without explaining its use, or describe the bars as OR |
| Does `assert!(!ready)` use the same `!` twice? What about `-> !`? | Distinguish macro invocation, boolean negation, and no normal return value; connect each to its position and purpose | Assign one meaning to all occurrences, call `!` unsafe, or claim a macro name becomes an equivalent function call without `!` |

## Relationships, objects, and state

| Example request | Observe | Problematic behavior |
|---|---|---|
| Explain why `x = x + 1` makes sense, and how `let x = x + 1` differs | Distinguish reading the old value from writing the destination; trace a new binding and an existing reference in the shadowing case | Treat assignment as equality or a recursive call, or say shadowing retargets existing references |
| If `name == name.clone()`, why distinguish cloning from `&name`? | Separate equal contents, independently owned values, and access to the original through a borrow | Equate value equality with object identity, or say cloning and borrowing have the same ownership behavior |
| Use relationships among symbols to explain `Option<&'a str>` and why `collect()` can infer its result type | Build the nested type from its parts, then show how an expected type constrains a method call; explain directly through Rust | Add unrelated background, call every nesting or reference recursion, or claim this perspective proves learning effectiveness |

## Deeper reading from types

| Example request | Observe | Problematic behavior |
|---|---|---|
| What can an experienced reader infer from `fn find_record<'a>(records: &'a [Record], key: &str) -> Option<&'a Record>` before seeing the body? | Explain borrowing, absence, validity, and the independent key lifetime; reserve provenance, duplicate selection, allocation, and complexity for implementation evidence | Claim the result must be from the slice, that shared access means purity, or that no allocation and linear search follow from the signature |
| Does `FnOnce` mean exactly one call, `move` mean a thread, and `dyn` mean heap allocation? | Explain call permissions, captures, dispatch, and the surrounding pointer separately | Infer invocation count, execution location, or allocation from the keyword alone |
| Why does `Box` fix a recursive enum, and does it also make recursive calls safe at any depth? | Separate finite layout, ownership, traversal, its end case, and stack use; distinguish a recursive type from a cycle in a particular value | Say `Box` stops recursion, always creates a self-referential value, or prevents stack exhaustion |
| Compare `twice!({ calls += 1; calls })` with passing the block to a function, given a macro expansion using the expression twice | Trace syntax expansion separately from runtime evaluation; predict the actual count and result for each | Claim expansion executes the block, assume all macros repeat inputs, or silently replace the example with a once-evaluated version |

## Edition-sensitive reading

Use the contrasting code in [Rust 2024](../plugins/rust-learning-lab/skills/learn-rust/references/rust-2024.md).
Passing its doctests verifies those snippets, not the host's explanation quality.

| Example request | Observe | Problematic behavior |
|---|---|---|
| This function returns an integer range. Why does dropping its input fail only in 2024? | Separate actual stored references from opaque-type capture; explain `use<>`, `+ 'a`, and the limitations for generic parameters | Say every `impl Trait` physically stores a reference, or prescribe `clone()` or `'static` without tracing the input |
| Why does `let [ref value] = &[42]` fail? Is `ref` obsolete? | Reconstruct the default binding mode and `&i32`; contrast an implicit pattern with `&[ref value]` | Ban all `ref`, confuse a pattern's `&` with a borrow expression, or treat `mut` as mutable access to the source |
| Can I replace this `if let` with `match` after migrating? | Check temporary guard destruction and the failed branch; explain what changes or is preserved | Claim branch equivalence proves identical resource lifetime, or approve `cargo fix` output only because it compiles |
| Why did the macro arm or boxed-slice iterator item change although the call text did not? | Use the macro definition's edition for fragment matching; inspect method resolution and owned versus borrowed item types | Use only the caller's edition for a macro, or infer ownership from the method name alone |
| The manifest says 2024 and resolver 3. Will these let chains run on Rust 1.85? | Require Rust 1.88 for let chains; distinguish edition support, feature stabilization, dependency selection, and actual minimum-version testing | Promise minimum-version support from the edition, resolver, or tests on a newer compiler |

## Choosing learning support

| Example request | Observe | Problematic behavior |
|---|---|---|
| I understand the loop, but cannot find which trait provides this method | Help locate the actual receiver, import, and definition before explaining unfamiliar parts | Diagnose poor memory or require a quiz for missing information |
| Assignment copies values in my usual language. Why can I reuse an integer but not this string? | Contrast the same operation for `Copy` and non-`Copy` types; give a replacement rule and its conditions | Say only that the prior model is wrong, or replace it with “Rust always moves” |
| Explain this borrowed argument, then show where the same idea applies elsewhere | Connect a concrete trace, the rule, and a changed case; return to the original expression | List terminology or unrelated examples without explaining the connection |
| Split this expression into variables to make its behavior easier to understand | Check temporary destruction, borrowing, evaluation count, and side effects before claiming equivalence | Treat every readability rewrite as behavior-preserving or silently edit the user's source |
| I can read Rust but am new to this service. Help me find its error reporting, then leave a note for tomorrow | Support one navigation task; record the location, current understanding, and next check | Assign a permanent beginner label, require several new skills at once, or treat explanation as mastery |

## Type evidence and external effects

| Example request | Observe | Problematic behavior |
|---|---|---|
| The request returned `Err`. Can I retry it safely? | Trace completed effects and whether the result is unknown; inspect request identity and the receiving protocol | Claim `?`, ownership, or `FnOnce` prevents duplicate remote effects |
| Does `Arc<Mutex<_>>` stop two servers booking the same slot? | Locate every writer and the actual database constraint or isolation requirement | Extend one process's lock across independent processes |
| Does timing out a Tokio `JoinHandle` stop its task and undo the request? | Distinguish dropping an owned handle, cancellation of a future, and remote rollback | Treat all timeout wrappers as equivalent or promise rollback from `Drop` |
| My private newtype validates input. Is derived deserialization and a new field safe? | Inspect construction paths and required old/new reader compatibility | Assume private fields invoke validation or same-version round trips prove compatibility |
| What else should I check after reading `Write`, `Eq`, or `Pin<P>`? | Explain partial writes and durability, semantic laws, or pointer versus pointee for the relevant type | Infer persistence, proved laws, or general immutability from notation alone |

## Review and document checks

| Example request | Observe | Problematic behavior |
|---|---|---|
| Review a positive-integer counter changed from `> 0` to `>= 0` | Show a triggering case such as `[0]`, the incorrect count, the fix, and the comparison's meaning | Give only a syntax lecture or bury the finding |
| Review code cloning a string because the caller must retain it | Inspect ownership needs and real costs; do not call it a defect without evidence | Require a change solely because of clone usage or line count |
| Give a detailed learning review of this repository change | Lead with important findings; save value flow, syntax, decisions, and verification scope in Markdown | Claim unrun examples were verified, or confuse excerpts with executable examples |
| In a Japanese conversation, paste an English error and request an English README.md | Keep chat Japanese and write the requested document in English | Switch chat because of the error language, or repeatedly ask for language preferences |
| Review this diff in English, chat only | Return English findings without changing documents or source | Force Japanese or automatically create a document |

## PR sources, indexes, and context isolation

| Example request | Observe | Problematic behavior |
|---|---|---|
| Create an educational review of PR #123 in a specified repository; the worktree is on another branch with edits | Record the repository and compared commit IDs, inspect matching definitions, preserve the worktree, and distinguish stated intent from code evidence | Check out over local work, use unrelated HEAD code, or claim inaccessible PR content was read |
| Explain a commit range, a merge commit, or a root commit | State the comparison semantics and selected parent or empty tree; pin the historical source links | Treat two-dot and three-dot diffs as identical, silently choose a merge parent, or attribute current tests to an older version |
| Save two reviews, then update the first; the directory README has personal notes | Keep separate files, update both language indexes with working relative links and source/date/language, preserve notes, and avoid duplicate rows | Overwrite another review, invent an index entry, leave the index stale, or read every review body to update one row |
| Create a learning review, then request an unrelated implementation task | Set up `.gitignore`, `.rgignore`, and shared agent guidance; ordinary discovery omits the directory including indexes, and the later task does not read them | Assume ignore rules remove tracked files, import the generated index, bypass exclusions during routine work, or claim unsupported host ignore behavior |
| Explicitly explain an existing ignored review; separately request a chat-only PR review | Read only the named review for the first task; change no files or configuration for the chat-only task | Block requested access, read all old reviews, or create ignore configuration despite the no-write constraint |

Also check a 2021-edition example and an explanation-only request that forbids edits in both hosts.
Do not change the edition or write files merely to explain code. Check reference selection too: Japanese explanations
use `_ja.md`, English documents use the English version, and both versions are not loaded unless translation comparison
is the task. Verify the references read and the output language from actual traces.
Record reproducing inputs and observations for failures; change only the relevant guidance.
