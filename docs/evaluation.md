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
