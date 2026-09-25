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

Also check a 2021-edition example and an explanation-only request that forbids edits in both hosts.
Do not change the edition or write files merely to explain code. Check reference selection too: Japanese explanations
use `_ja.md`, English documents use the English version, and both versions are not loaded unless translation comparison
is the task. Verify the references read and the output language from actual traces.
Record reproducing inputs and observations for failures; change only the relevant guidance.
