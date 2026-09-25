# Rust Learning Lab

English | [日本語](README_ja.md)

**Understand the syntax. Explain the decisions. Verify the behavior.**

A learning and code review plugin for Codex and Claude Code. Read Rust from syntax and design rationale
through arguments, types, ownership, and control flow, then practice making and checking small changes.
Reviews identify concrete problems and explain why a fix works. Detailed learning reviews can become
Markdown documents you can revisit.

One shared skill, `learn-rust`, serves both hosts. Its instructions are written in Japanese, but responses
follow your requested language or the language of the conversation. Documents follow their requested
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

$learn-rust Review this diff and explain the design decisions and syntax behind your findings.
Write the review to docs/parser-learning-review.md.
```

Ordinary questions get an answer first. Exercises are optional. Include the code, full error message,
and what you already understand when available.

Detailed learning reviews of a repository produce a document such as `docs/rust-learning-review.md`
by default. An explicit destination, “chat only,” or read-only instruction takes precedence.
Reviewing does not authorize source edits; ask for fixes when you want them applied.
See a [short example review document](plugins/rust-learning-lab/skills/learn-rust/references/review-example.md)
(written in Japanese; generated reviews use the appropriate language).

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
relevant. The plugin includes no MCP servers, hooks, or custom runtime. It does not independently collect
data; conversation handling and tool execution follow your host's settings.

```sh
rustdoc --test --edition 2024 plugins/rust-learning-lab/skills/learn-rust/references/worked-examples.md
rustdoc --test --edition 2024 plugins/rust-learning-lab/skills/learn-rust/references/rust-2024.md
rustdoc --test --edition 2024 plugins/rust-learning-lab/skills/learn-rust/references/review-example.md
claude plugin validate .
claude plugin validate plugins/rust-learning-lab
```

CI checks distribution metadata and executable Rust examples. The [conversation scenarios](docs/evaluation.md)
are for manual evaluation; passing syntax checks and examples does not establish model behavior or learning outcomes.
Keep the versions in both host manifests aligned, and update both READMEs when usage changes.

MIT License. Linked papers and external resources retain their own terms.
