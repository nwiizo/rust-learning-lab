# Check source and output quotations

English | [日本語](checking-documents_ja.md)

Use when a learning document needs repeatable checks of quoted source,
observed output, or an intentionally failing example. Prefer existing Cargo
tests and rustdoc for ordinary project examples. The optional Rust CLI
`rust-learning-check` supports standalone programs using the standard library.
It is maintained separately and is not bundled or installed by this plugin.
Use an available local checkout or installed binary; do not assume it exists.

## Prepare the checks

Keep the manifest, documents, and source files under one directory. All paths,
including Markdown markers, are relative to the manifest directory. A minimal
manifest for a working example and its intentionally failing contrast is:

```toml
version = 1
edition = "2024"
documents = ["lesson.md"]

[[examples]]
id = "borrowed"
path = "borrowed.rs"
kind = "run"

[[examples]]
id = "moved"
path = "moved.rs"
kind = "compile-fail"
error_codes = ["E0382"]
corrected = "borrowed"
```

Set the edition for the examples and use a compatible compiler. Before a
fenced block, place `<!-- rlc:source borrowed.rs -->` to match contiguous
complete source lines, or `<!-- rlc:stdout borrowed -->` to match full output.
Use `stderr` for the other stream. For a failing example, stderr is the rendered
compiler diagnostic. Copy output from a real run; mark omissions explicitly
and leave shortened illustrations unmarked. CRLF and LF are equivalent, and
output comparison ignores one final newline, but preserves other whitespace.
At least one marked block is required. Inspect diagnostic locations as well
as codes; the same code can occur at an unintended expression.

## Run and report

Run `rust-learning-check check checks.toml` if installed. With a local CLI
checkout, run `cargo run --locked --manifest-path <cli-checkout>/Cargo.toml -- check <checks.toml>`.
Replace both placeholders with actual paths. The CLI requires Rust 1.90 or later
to build and supports macOS and Linux. `RUSTC` selects the compiler executable.
Record the printed compiler version and edition with the result.

The CLI checks references and source quotes before execution, requires a
successful exit for run examples, and checks the exact error-code set plus a
working correction for compile-fail examples. It compares complete output,
not substrings. A nonzero exit remains a failure even if stdout matches.
The default limit is 30 seconds per subprocess and 1 MiB per output stream.
Execute only trusted examples: these limits are not a sandbox, and programs
inherit the caller's environment and access. Exit codes are 0 for success,
1 for failed checks, and 2 for invalid CLI usage; stop at the first failure
and treat earlier PASS lines as partial results.

If the CLI is unavailable or the example requires Cargo dependencies/features,
use the existing toolchain and compare source and output directly. State what
was run and what was only inspected. Unmarked text, diagram meaning, equivalent
behavior on all inputs, and learning outcomes remain outside these checks.
Use [Decisions and evidence](verified-explanations.md) to choose other evidence.
