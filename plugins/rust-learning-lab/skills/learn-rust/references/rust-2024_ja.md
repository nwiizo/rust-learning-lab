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

たとえば、捕捉範囲を指定する `use<..>` は Rust 1.82 から Edition を問わず使えるが、
暗黙の捕捉ルールが変わったのは2024。let chainsには **Rust 1.88以上と2024 Editionの両方** が必要になる。
「2024対応」と書かれた例が1.85でも動くとは限らないため、機能が安定したバージョンを別に確認する。

Rust 1.88以上では、このように束縛した変数を次の条件で使える。

```rust,edition2024
let candidate = Some(4);
let accepted = if let Some(value) = candidate && value > 0 {
    value
} else {
    0
};
assert_eq!(accepted, 4);
```

手前の条件が成立しなければ、その先の条件は評価しない。この機能は `if` / `while` の条件を
`&&` でつなぐもので、`let` を一般的な真偽値の式にしたり、`||` で連結できるようにしたりするものではない。

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

## 同じコードを二つのEditionで読む

以下の比較では、RustのコードブロックごとにEditionを指定している。
`rustdoc --test --edition 2024` でも個別の指定が優先されるため、2021の例は同じコンパイラで
Editionだけを変えて検証できる。これはEditionによる差の確認であり、最小対応コンパイラでの
検証ではない。`compile_fail` は意図したコンパイルエラーを示す。

### パターン：束縛される変数の型を導く

まず、どちらのEditionでも動く形から読む。

```rust,edition2024
let [value] = &[42];
// value: &i32
let _: &i32 = value;
```

配列のパターンに対して、照合対象は配列への参照になっている。この参照を暗黙にたどることで、
既定の束縛方法が移動・コピーから借用に変わる。そのため `ref` を書かなくても `value` は `&i32` になる。

```rust,edition2021
let [ref value] = &[42];
let _: &i32 = value;
```

```compile_fail,edition2024
let [ref value] = &[42];
```

2024の制限は明示的な指定を**どこに書くか**に関するもので、`ref` 全体の禁止ではない。
必要なら、そこまでの参照部分を明示できる。

```rust,edition2024
let &[ref value] = &[42];
let _: &i32 = value;
let &[mut number] = &[42];
number += 1;
assert_eq!(number, 43);
```

このパターンの `&` は参照の層に明示的に照合する。式の `&` のように借用を作るわけではない。
その先の `ref value` が要素を借用する。`mut number` は要素をコピーした変更可能なローカル変数
`i32` を作り、元の配列を変更可能にするわけではない。2024では `mut`、`ref`、`ref mut` を書けるのは
既定の束縛方法がmoveの場合に限られ、参照パターンも、それ以前の部分で暗黙の借用を使っていると書けない。
通常の借用による照合では最初の形を使い、推論された型を見えるように説明する。

### 戻り値の具体型を隠す：借用の可能性と実際の中身

スライスの長さを読み、二つの整数だけを持つ範囲を返す関数を考える。
この関数本体はスライスへの参照を保持していない。

```rust,edition2021
fn positions(items: &[u8]) -> impl Iterator<Item = usize> {
    0..items.len()
}
let items = vec![10, 20];
let positions = positions(&items);
drop(items);
assert_eq!(positions.collect::<Vec<_>>(), [0, 1]);
```

```compile_fail,edition2024,E0505
fn positions(items: &[u8]) -> impl Iterator<Item = usize> {
    0..items.len()
}
let items = vec![10, 20];
let positions = positions(&items);
drop(items);
assert_eq!(positions.collect::<Vec<_>>(), [0, 1]);
```

2024では、具体型を隠した戻り値の型が入力のライフタイムを暗黙に捕捉する。
呼び出し側は、この実装が参照を保持していなくても、返却値が `items` を借用する可能性を考慮する。
入力の借用に依存しないことをAPIの性質として示したいなら、明示する。

```rust,edition2024
fn positions(items: &[u8]) -> impl Iterator<Item = usize> + use<> {
    0..items.len()
}
let items = vec![10, 20];
let positions = positions(&items);
drop(items);
assert_eq!(positions.collect::<Vec<_>>(), [0, 1]);
```

`use<>` は捕捉範囲の指定であり、importでも実行時の操作でもない。
ここではジェネリックパラメーターの捕捉を一つも許可しない。`use<'a, T>` は列挙したものの捕捉を許可する。
一方、`+ 'a` は隠された型が `'a` の期間有効であることを求める。`'a` を捕捉することと、
`'a` の期間有効であることは別の条件になる。型パラメーター `T` の捕捉を通じて、その型に含まれる
ライフタイムへの依存が残る場合もある。`use<>` を万能の修正として示さない。
本体が `items.iter()` を返すなら、実際に入力を借用している。

現在の捕捉範囲の指定では、スコープ内の型・constパラメーターはすべて列挙し、
戻り値のほかの境界条件で使ったライフタイムも含める必要がある。引数の `impl Trait` は
名前のない型パラメーターを導入するため、捕捉リストを使うなら先に `T: Trait` などの名前を付ける。
ジェネリックなAPIでは、上の空リストから一般化せず、
[捕捉範囲の指定に関する制約](https://doc.rust-lang.org/reference/types/impl-trait.html#precise-capturing)を確認する。

この2021の比較例は独立した関数についてのもの。traitメソッドや `async fn` では2024以前から
捕捉ルールが異なっていたため、すべての `impl Trait` に一般化しない。
通常の参照を返す関数のライフタイム省略規則や、クロージャーの `move` による捕捉とも区別する。

### 一時値：ガードの破棄が次にできる操作を変える

一時値にも破棄される時点がある。`RefCell` の借用ガードは、破棄されるまで実行時の借用を保持する。
ここでは `try_borrow_mut()` を使い、パニックやロック待ちを起こさずに、その時点を観察する。

```rust,edition2021
use std::cell::RefCell;
let slot = RefCell::new(None::<u32>);
if let Some(_) = *slot.borrow() {
    unreachable!();
} else {
    assert!(slot.try_borrow_mut().is_err());
};
```

```rust,edition2024
use std::cell::RefCell;
let slot = RefCell::new(None::<u32>);
if let Some(_) = *slot.borrow() {
    unreachable!();
} else {
    assert!(slot.try_borrow_mut().is_ok());
};
```

2024では照合が失敗すると、`else` に入る前に一時的なガードが破棄される。
2021では `else` の間も残っている。`match` の照合対象にある一時値は、引き続き選ばれた分岐の間も生存する。
したがって、`if let` を `match` に書き換えると、分岐の選択が同じでもリソースを使える時点が変わり得る。
自動移行が古い動作を保つために `match` を選んだ場合、その動作を本当に保ちたいか確認する。
変数に束縛したガードにはその変数のスコープがあるため、「すべての借用が `else` の前に終わる」とは説明しない。

末尾式にも関連する変更がある。ブロック末尾の一時的なガードがそのブロックのローカル変数より先に
破棄されるようになり、次の関数が通るようになった。

```compile_fail,edition2021,E0597
use std::cell::RefCell;
fn width() -> usize {
    let label = RefCell::new(String::from("rust"));
    label.borrow().len()
}
assert_eq!(width(), 4);
```

```rust,edition2024
use std::cell::RefCell;
fn width() -> usize {
    let label = RefCell::new(String::from("rust"));
    label.borrow().len()
}
assert_eq!(width(), 4);
```

一時値のスコープが短くなることで、以前は通ったコードが拒否される場合もある。
たとえば、ブロック内の一時値を参照先とする参照を、そのブロックの外へ持ち出す場合が該当する。

```rust,edition2021
let width = { &String::from("rust") }.len();
assert_eq!(width, 4);
```

```compile_fail,edition2024,E0716
let width = { &String::from("rust") }.len();
assert_eq!(width, 4);
```

所有者を長く生存させたいなら、外側で変数に束縛する。

```rust,edition2024
let label = String::from("rust");
let width = { &label }.len();
assert_eq!(width, 4);
```

ライフタイム注釈では所有者の生存期間を延ばせない。「2024ならライフタイムエラーが直る」と
覚えるのではなく、所有者、ガード・参照、最後の使用、破棄時点を別々に追う。

### マクロの照合：同じトークンでも選ばれるルールが変わる

マクロの照合は、展開後の式を評価する前に行われる。次の二つは同じ呼び出しに対して、
異なるルールが選ばれることを確かめている。

```rust,edition2021
macro_rules! classify {
    ($value:expr) => { "expression" };
    (const $value:expr) => { "const-prefixed" };
}
assert_eq!(classify!(const { 7 }), "const-prefixed");
```

```rust,edition2024
macro_rules! classify {
    ($value:expr) => { "expression" };
    (const $value:expr) => { "const-prefixed" };
}
assert_eq!(classify!(const { 7 }), "expression");
```

最初に一致したルールが選ばれる。2024の `expr` は最上位の `const` ブロックと `_` も受け取るため、
以前は二番目に一致したトークンが、範囲の広がった最初のルールに一致し得る。
ここで効くのは**マクロを定義した側のEdition**であり、呼び出し側のEditionではない。
`expr_2021` は以前の狭い照合範囲を保つ。マクロの入力として `_` を受け取れることと、
展開先のあらゆる式の位置に `_` を書けることを混同しない。

### メソッド呼び出し：受け手の型から所有権を読む

十分に新しいコンパイラでは `Box<[T]>` の `IntoIterator` 実装はどのEditionにもある。
ただし、このメソッド呼び出しの解決方法がEditionで変わる。

```rust,edition2021
let names = vec![String::from("Ada")].into_boxed_slice();
for name in names.into_iter() {
    let _: &String = name;
}
assert_eq!(names.len(), 1);
```

```rust,edition2024
let names = vec![String::from("Ada")].into_boxed_slice();
for name in names.into_iter() {
    let _: String = name;
}
// names has been moved.
```

2021のメソッド呼び出しは借用し、2024ではboxを消費して所有した要素を返す。
`.iter()` ならどちらのEditionでも借用による反復を明示できる。
`IntoIterator::into_iter(names)` なら、このboxについてはどちらでも所有権を移す反復を明示できる。
メソッド呼び出しの互換ルールを `for name in names` にまで当てはめない。

これらの例では、記号の位置を特定し、推論される型やスコープを導き、観察可能な結果を予測するという
熟練者の読み方を示している。予測がEditionに依存するときは、その前提も示し、記号だけで決まるように説明しない。

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

宣言と使用を分けて考える。`unsafe extern` では、宣言者が外部関数の情報の正確さを確認する。
`unsafe fn` では呼び出し側に守るべき条件があり、`unsafe { ... }` では実装者が個々の操作を行える根拠を示す。
外部関数の宣言は既定でunsafeだが、追加の安全条件なしに呼べる関数はブロック内で `safe fn` と明示できる。
ブロック全体に `unsafe` が付いていても、すべての呼び出しがunsafeになるわけではない。
このブロック構文は Rust 1.82 から使え、2024では必須になった。

環境変数の変更には、以前のEditionでもプラットフォームに依存する安全条件があった。
変わるのは呼び出し側に要求される記述であり、基盤となる操作の競合可能性ではない。
子プロセスだけに変数を渡したい場合は、親の環境を書き換える代わりに `Command::env` を検討する。

## Cargo、マクロ、整形

- `edition = "2024"` は通常 `resolver = "3"` を意味し、`rust-version` と互換性のある
  依存バージョンを優先する。仮想ワークスペースでは必要に応じて
  `[workspace] resolver = "3"` を明示する。
- `macro_rules!` の `$e:expr` は `const` ブロックと `_` 式も受け取る。
  以前の照合範囲を維持する必要がある場合は `expr_2021` を使う。
- Rustfmt には style edition がある。`cargo fmt` はマニフェストの Edition に合わせる。
  エディタが `rustfmt` を直接呼ぶ場合は、必要に応じて `rustfmt.toml` の
  `style_edition = "2024"` でCIと揃える。

Resolver 3の優先方針だけでは、最小対応バージョンで動く証拠にならない。
固定された依存関係、有効なfeature、対象プラットフォームを確認し、宣言した最小ツールチェーンで
ビルド・テストする。同様に、整形できたこともEditionによる実行時の動作が保たれた証拠にはならない。

ほかの診断上の変更は、必要な質問で扱う。型変換先が推論で決まらない場合のnever型のフォールバックは
`()` から `!` に変わるが、これは `!` と `()` が同じ型だったという意味ではない。
必要に応じて、意図した結果の型を明示する。`gen` やガード付き文字列のトークン形式は予約であり、
ジェネレーターブロックや新しい文字列リテラルをすでに使えることの根拠にはならない。

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
- [Macro fragment specifiers](https://doc.rust-lang.org/edition-guide/rust-2024/macro-fragment-specifiers.html)
- [IntoIterator for boxed slices](https://doc.rust-lang.org/edition-guide/rust-2024/intoiterator-box-slice.html)
- [Let chains and Rust 1.88](https://blog.rust-lang.org/2025/06/26/Rust-1.88.0/)
- [Never type fallback](https://doc.rust-lang.org/edition-guide/rust-2024/never-type-fallback.html)
- [Reserved syntax](https://doc.rust-lang.org/edition-guide/rust-2024/reserved-syntax.html)
- [Unsafe attributes](https://doc.rust-lang.org/edition-guide/rust-2024/unsafe-attributes.html)
- [Unsafe `extern` blocks](https://doc.rust-lang.org/edition-guide/rust-2024/unsafe-extern.html)
- [Newly unsafe functions](https://doc.rust-lang.org/edition-guide/rust-2024/newly-unsafe-functions.html)
- [set_var safety](https://doc.rust-lang.org/std/env/fn.set_var.html)
- [Disallow references to `static mut`](https://doc.rust-lang.org/edition-guide/rust-2024/static-mut-references.html)
- [Cargo resolver](https://doc.rust-lang.org/edition-guide/rust-2024/cargo-resolver.html)
- [Rustfmt style edition](https://doc.rust-lang.org/edition-guide/rust-2024/rustfmt-style-edition.html)
