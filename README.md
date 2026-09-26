# Rust Learning Lab

English | [日本語](README_ja.md)

**Understand the syntax. Explain the decisions. Verify the behavior.**

A learning and code review plugin for Codex and Claude Code. Read Rust from syntax and design rationale
through arguments, types, ownership, and control flow, then practice making and checking small changes.
Reviews identify concrete problems and explain why a fix works. Detailed learning reviews can become
Markdown documents you can revisit.

One shared skill, `learn-rust`, serves both hosts. Its instructions and all eight references are available in
English and Japanese. Responses follow your requested language or the language of the conversation. Documents follow their requested
language or the repository's conventions. English code or error messages alone do not switch the response language.

## Install

Use a version of your host CLI that supports plugins. Rust is optional for explanations;
running examples requires a toolchain compatible with the target project.
Distribution checks use Codex CLI 0.157.0, Claude Code 2.1.282, and Rust 1.98.1.

### Codex

```sh
codex plugin marketplace add nwiizo/rust-learning-lab
codex plugin add rust-learning-lab@rust-learning-lab
```

Start a new conversation and invoke `$learn-rust`, or select **Learn Rust** in the skill picker.

### Claude Code

```sh
claude plugin marketplace add nwiizo/rust-learning-lab
claude plugin install rust-learning-lab@rust-learning-lab
```

Start a new conversation and use `/rust-learning-lab:learn-rust` followed by your question.

This repository provides its own marketplace using the
[Codex plugin format](https://developers.openai.com/plugins/build/plugins) and the
[Claude Code marketplace format](https://code.claude.com/docs/en/plugin-marketplaces).
Distribution here does not imply inclusion in either host's official catalog.

## Use

These examples use Codex syntax. In Claude Code, replace `$learn-rust` with `/rust-learning-lab:learn-rust`.

```text
$learn-rust In fold(10, |acc, item| acc - item), who supplies each argument?
Explain the order, types, and intermediate values.

$learn-rust How do &str and &name differ? Explain the syntax and why borrowing helps here.

$learn-rust Help me practice Result and ?. Give hints first and wait for my answer.

$learn-rust Create an educational review of PR #123 in owner/repo.
Explain the before/after behavior, syntax, argument order, and design decisions.
Save it to docs/rust-learning-review/pr-123-parser.md and update the directory index.

$learn-rust Create a learning document from commit <sha> (or the range <base>..<head>).
Record the compared revisions, explain ownership and error handling, and review potential problems.
Save it under docs/rust-learning-review/ and update the directory index.
```

Ordinary questions get an answer first. Exercises are optional. Include the code, full error message,
and what you already understand when available.

Replace the example repository, PR number, and commit placeholders with real sources. A PR URL also works.
The skill reads the selected diff and relevant code at that revision; it separates stated intent from verified behavior.

Detailed learning reviews produce separate Markdown files under `docs/rust-learning-review/` by default.
Each creation or update also maintains `README.md` (English) and `README_ja.md` (Japanese) in that directory with
relative document links, source PR/commit, language, and update date. The indexes explain that these are educational
notes about specific source snapshots, not product documentation or repository instructions.
An explicit destination, “chat only,” or read-only instruction takes precedence.
Reviewing does not authorize source edits; ask for fixes when you want them applied.
See a [short example review document](plugins/rust-learning-lab/skills/learn-rust/references/review-example.md).

On the first document, the skill adds the directory to `.gitignore` and `.rgignore` and records its context-exclusion
rule in the project's agent guidance, preserving existing settings. The indexes are ignored too. Routine work skips
these materials; explicitly requested learning work can read the relevant document. The guidance also tells agents not
to copy this content automatically into memory, rules, summaries, or product documentation. These measures do not block
every possible direct read. Already tracked files remain tracked, and instructions forbidding configuration
changes take precedence. The plugin does not install a background indexer: the skill updates the index when it writes a review.

## What it supports

- **Syntax and rationale:** read symbols, signatures, and inferred types; distinguish language rules, conventions, and speculation.
- **Arguments and flow:** trace receivers, references, closure arguments, return values, and early exits with concrete inputs.
- **Independent practice:** choose tracing, prediction, partial completion, small edits, or debugging to match the difficulty.
- **Review and verification:** locate defects and behavioral changes, then explain their conditions, impact, and fixes through the code.
- **Continuity:** distinguish an explanation given from understanding demonstrated, and leave a focused next step when useful.

The guidance draws on ten programming education and AI assistance studies, plus two memory studies.
The [research notes](plugins/rust-learning-lab/skills/learn-rust/references/learning-evidence.md)
record findings and limitations. Classroom or other-language results are not treated as proof of effectiveness
for individual Rust learners. This plugin's learning effectiveness has not been measured.

## Develop and verify

Both hosts use `plugins/rust-learning-lab/skills/learn-rust/`. Supporting references are read only when
relevant, in the language of the current explanation or document. Each English reference links to its `_ja.md`
counterpart; Japanese references link back and use Japanese local references. Other response languages use English
references as a fallback without changing the response language. Both translations are not loaded by default.
The plugin includes no MCP servers, hooks, or custom runtime. It does not independently collect
data; conversation handling and tool execution follow your host's settings.

```sh
rustdoc --test --edition 2024 plugins/rust-learning-lab/skills/learn-rust/references/worked-examples.md
rustdoc --test --edition 2024 plugins/rust-learning-lab/skills/learn-rust/references/rust-2024.md
rustdoc --test --edition 2024 plugins/rust-learning-lab/skills/learn-rust/references/review-example.md
claude plugin validate .
claude plugin validate plugins/rust-learning-lab
```

Run the doctest commands for the `_ja.md` counterparts too. CI checks distribution metadata, bilingual file coverage,
matching Rust code blocks, and executable Rust examples in both languages. The [conversation scenarios](docs/evaluation.md)
are for manual evaluation; passing syntax checks and examples does not establish model behavior or learning outcomes.
Keep the versions in both host manifests aligned and update both languages when meaning changes.
[Repository guidance](AGENTS.md), also loaded by Claude Code, records the language, review-document, verification,
and publication workflow. See [the skill instructions](plugins/rust-learning-lab/skills/learn-rust/SKILL.md) for reference routing.

MIT License. Linked papers and external resources retain their own terms.
