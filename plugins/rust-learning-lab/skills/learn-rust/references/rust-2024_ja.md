# Rust 2024 Edition

[English](rust-2024.md) | 日本語

Rust 2024、Edition 移行、または Edition によって診断や動作が異なる質問でだけ読む。
変更項目を一度に教えず、質問に関係する節だけを使う。

## 最初に整理すること

- **Edition とツールチェーンは別物。** Rust 2024 Edition は Rust 1.85.0 で安定した。
  `edition = "2024"` は構文や一部の意味をクレートごとに選び、`rustc 1.xx` は
  コンパイラと標準ライブラリの版を表す。
- **Edition はクレートごとに選べる。** 異なる Edition のクレートは同じ依存関係の中で
  相互利用できるため、依存クレートまで一斉に移行する必要はない。
- **判断材料は `Cargo.toml`。** `[package]` の `edition` を確認する。省略時は
  後方互換性のため2015として扱われる。`edition.workspace = true` ならルートの
  `[workspace.package]` を確認する。`cargo new` は最新の安定 Edition を明記する。
- **`rust-version` は別の設定。** サポートする最小ツールチェーンを表す。
  Rust 2024 へ移るなら1.85以上が必要だが、実際に使う機能や依存関係に合わせて決める。

Rust 1.85 で同時に安定した機能が、すべて Rust 2024 専用とは限らない。
「Edition を変えた効果」と「toolchain を更新した効果」を混同しない。

## 新規プロジェクト

Rust 2024 の学習用プロジェクトを確実に作るには、Edition を明示する。

```bash
cargo new --edition 2024 hello-rust
cd hello-rust
cargo run
```

`cargo new` の既定値は最新の安定 Edition なので、将来も Rust 2024 を再現したい例では
`--edition 2024` を付ける。説明では、`cargo new` がプロジェクトを作り、生成された `Cargo.toml` の
`edition = "2024"` が Edition を選び、`cargo run` がビルドと実行を行うと伝える。
古い教材を再現するなど明確な理由がある場合だけ `cargo new --edition 2021` のように指定する。

## 既存プロジェクトの移行

移行を実行する依頼では、先に Git の作業状態と現在のテスト結果を確認する。
説明だけの依頼で、コマンドを勝手に実行しない。

公式ガイドを参照し、必要な依存更新とEdition対応を分けて進める。

```text
依存関係と移行前の動作を確認
↓
cargo fix --edition
↓
Cargo.toml を edition = "2024" へ変更
↓
cargo build / cargo test
↓
cargo fmt
```

- 依存関係の更新が必要な場合は、対象と理由を確認して行う。`cargo update` は
  `Cargo.lock` も変え得るため、無条件の全更新を前提にせずEdition対応との差を確認する。
- `cargo fix --edition` は Edition を書き換える前に実行する。互換 lint を有効にして、
  可能な範囲で現在の動作を保つ修正を適用する。
- `cargo fix --edition` が準備するのは次の Edition まで。2015や2018から始める場合は、
  Edition の変更と検証を一段ずつ繰り返して2024へ進める。
- 無効な feature や別 platform の `cfg` は通常の実行だけでは検査されない。
  必要に応じて `--all-features` や `--target` を使い、対象範囲を説明する。
- 自動修正できない警告、doctest、コード生成、マクロ、FFI は手動確認が必要になる。
- 自動的に追加された `unsafe` ブロックや、`if let` から `match` への変更は、
  コンパイルできることだけで採用せず、安全条件や破棄タイミングを確認する。
- 最後に Git diff とテストを確認し、Edition 変更による差を依存更新や整形の差と分ける。

## 初学者の診断に影響しやすい変更

| 領域 | Rust 2024 の要点 | 説明するときの着眼点 |
|---|---|---|
| match ergonomics（参照の自動処理） | 暗黙に参照を読み飛ばした後の `mut`、`ref`、`ref mut`、`&` パターンが制限された | パターン、照合対象、束縛された変数の型を別々に示す。まず冗長な `ref` を外し、必要なら参照部分をすべて明示する |
| `if let` の一時値 | 照合対象の一時値は `else` に入る前に破棄される | ロックガードや `RefCell` の借用がいつ解放されるかを追う。旧 Edition と同じ寿命が必要なら `match` を検討する |
| 末尾式の一時値 | ブロック末尾の一時値がローカル変数より先に破棄され、外側へ延長されない場合がある | E0597 などでは式をローカル変数へ分け、参照先と一時値の破棄順を図示する |
| 戻り値位置の `impl Trait` | `use<..>` がなければ、スコープ内のジェネリックパラメーターを暗黙に捕捉する。ライフタイム差が特に表れやすい | 引数位置の `impl Trait` と区別する。まず返却値が何を借用し得るかを説明し、厳密な制御が必要な場合だけ `use<'a, T>` へ進む |
| `gen` | 将来の `gen` ブロック用の予約語になった | 既存名は変更するか、移行互換性が必要なら `r#gen` という raw identifier を使う |
| `Box<[T]>::into_iter()` | 2024 では要素を所有して返し、以前の Edition のメソッド呼び出しは参照を返す | 受け手と要素の型を確認する。一般に `into_iter()` は受け手自体を値で受けるが、その受け手が参照の場合もあり、要素の所有権を常に移すわけではない |
| prelude（自動 import） | `Future` と `IntoFuture` が追加された | 同名メソッドが曖昧なら完全修飾構文で、どの trait のメソッドかを示す |

match ergonomics の最小例では、2024で冗長な `ref` がエラーになることを示せる。

```rust
let [value] = &[42];
// value: &i32
```

`let [ref value] = &[42];` がなぜ不要かを、「照合対象の `&` により既定の束縛方法が
すでに参照になっている」と説明する。複雑な完全明示パターンから始めない。

## `unsafe` 周辺

Rust 2024 は、安全性の確認を必要とする場所を構文上も見つけやすくしている。
`unsafe` を付ければ安全になるのではなく、コンパイラに任せられない安全条件を確認する責任があると説明する。

- `extern` ブロックは `unsafe extern "C" { ... }` と書く。宣言した関数のシグネチャが
  外部実装と一致することはコンパイラが確認できない。
- `no_mangle`、`export_name`、`link_section` は `#[unsafe(...)]` と書き、
  シンボル衝突やリンク時の条件を確認する。
- `unsafe fn` の中でも、`unsafe` 操作は小さな `unsafe { ... }` に入れる。
  2024では `unsafe_op_in_unsafe_fn` が標準で警告する。
- `std::env::set_var` と `std::env::remove_var` は2024では unsafe。ほかのスレッドが
  動き得る状況で環境変数を書き換えないなど、ドキュメントの `Safety` 節を先に確認する。
  `cargo fix` が囲んだだけの `unsafe` を完成形として見せない。
- `static mut` への参照作成は標準で拒否される。初学者には通常、値に応じて
  アトミック型、`Mutex` / `RwLock`、`OnceLock` / `LazyLock` を先に示す。

FFI やグローバル状態は高度な題材なので、質問へ直接必要な安全条件だけを扱う。

## Cargo、マクロ、整形

- `edition = "2024"` は通常 `resolver = "3"` を意味し、`rust-version` と互換性のある
  依存バージョンを優先する。仮想ワークスペースでは必要に応じて
  `[workspace] resolver = "3"` を明示する。
- `macro_rules!` の `$e:expr` は `const` ブロックと `_` 式も受け取る。
  以前の照合範囲を維持する必要がある場合は `expr_2021` を使う。
- Rustfmt には style edition がある。`cargo fmt` はマニフェストの Edition に合わせる。
  エディタが `rustfmt` を直接呼ぶ場合は、必要に応じて `rustfmt.toml` の
  `style_edition = "2024"` でCIと揃える。

## 公式資料

- [Workspace package inheritance](https://doc.rust-lang.org/cargo/reference/workspaces.html#the-package-table)
- [IntoIterator](https://doc.rust-lang.org/std/iter/trait.IntoIterator.html)
- [What are Editions?](https://doc.rust-lang.org/edition-guide/editions/)
- [Rust 1.85.0 and Rust 2024](https://blog.rust-lang.org/2025/02/20/Rust-1.85.0.html)
- [Creating a new project](https://doc.rust-lang.org/edition-guide/editions/creating-a-new-project.html)
- [Transitioning an existing project](https://doc.rust-lang.org/edition-guide/editions/transitioning-an-existing-project-to-a-new-edition.html)
- [Match ergonomics reservations](https://doc.rust-lang.org/edition-guide/rust-2024/match-ergonomics.html)
- [`if let` temporary scope](https://doc.rust-lang.org/edition-guide/rust-2024/temporary-if-let-scope.html)
- [Tail expression temporary scope](https://doc.rust-lang.org/edition-guide/rust-2024/temporary-tail-expr-scope.html)
- [RPIT lifetime capture rules](https://doc.rust-lang.org/edition-guide/rust-2024/rpit-lifetime-capture.html)
- [Unsafe attributes](https://doc.rust-lang.org/edition-guide/rust-2024/unsafe-attributes.html)
- [Unsafe `extern` blocks](https://doc.rust-lang.org/edition-guide/rust-2024/unsafe-extern.html)
- [Newly unsafe functions](https://doc.rust-lang.org/edition-guide/rust-2024/newly-unsafe-functions.html)
- [Disallow references to `static mut`](https://doc.rust-lang.org/edition-guide/rust-2024/static-mut-references.html)
- [Cargo resolver](https://doc.rust-lang.org/edition-guide/rust-2024/cargo-resolver.html)
- [Rustfmt style edition](https://doc.rust-lang.org/edition-guide/rust-2024/rustfmt-style-edition.html)
