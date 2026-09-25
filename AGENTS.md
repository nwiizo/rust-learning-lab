# Repository guidance

English | [日本語](AGENTS_ja.md)

## Shared content and languages

- Both hosts load `plugins/rust-learning-lab/skills/learn-rust/SKILL.md`. Keep one skill implementation; do not fork by host or language.
- Use English in unsuffixed Markdown and Japanese in `_ja.md` counterparts. Update both when meaning changes, with reciprocal links and same-language local references. `SKILL_ja.md` is a reader translation, not another entrypoint.
- Infer response language from explicit preference, then the conversation; English code or errors alone do not change it. Documents follow their requested language or existing conventions. Load only relevant references in the language of the current output.
- Preserve code, commands, API names, numerical findings, and research limitations across translations. Keep corresponding Rust code blocks identical. Translate local heading fragments as well as link labels.

## Learning and review behavior

- Explain syntax through its purpose, argument mapping, types, ownership, and value flow. Separate language rules, API choices, historical evidence, and inference.
- Answer direct questions first. Exercises are optional. Distinguish working code, an explanation given, and understanding demonstrated independently.
- Reviews lead with substantiated findings and observable impact. Detailed learning reviews can produce Markdown connecting design decisions to syntax and verification. Respect chat-only, read-only, and destination instructions; reviews do not authorize source fixes.
- Preserve the rationale and conditions behind advice. Do not turn one example or a weak research result into a universal rule. Keep substantial conditional guidance in references rather than expanding every response or the skill entrypoint.
- Do not add private source material, reading notes, local paths, or excluded attributions to distribution files. Paraphrase applicable ideas in original guidance; retain the public primary-source citations supporting research claims.

## Verification and publication

- Inspect `git status` and `git diff`, preserve unrelated changes, and stage only task files. Commit, push, and publish only with authorization from the current conversation.
- Run the checks in `.github/workflows/validate.yml`: distribution metadata, bilingual file coverage, Rust code-block parity, and doctests in both languages. Run `actionlint` after workflow edits and `lychee --offline --include-fragments --no-progress '**/*.md'` for local links.
- Validate both host manifests after packaging changes. Static checks and doctests do not establish model behavior, review accuracy, or learning effectiveness; record behavioral evaluations only when actually performed.
- Before an authorized release, align both manifest versions, scan for secrets and excluded private material, and verify the branch and remote. Do not rewrite published tags. Test installation from the published marketplace in each host when distribution changes.
- For installed copies, use each host's marketplace refresh and plugin install/update commands. Keep caches out of Git, verify installed versions and content, and state when a new conversation or restart is required.
