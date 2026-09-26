# 実行できる説明例

[English](worked-examples.md) | 日本語

記号・引数順・借用・早期リターンを具体的に説明するときの対比例。
Rust 2024で `rustdoc --test --edition 2024 <このファイル>` を実行すると、成功例と意図したコンパイル失敗を確認できる。
`assert_eq!` は二つの値が等しいことを確認し、違う場合にテストを失敗させるマクロ。
例の値や説明量は学習者の質問へ合わせる。

## 記号を使われている場所から読む

全節を説明せず、質問に近い例を選ぶ。対象の式を普段の言葉で読み下し、型や値の流れを示してから、
記法が可能にすることと、別の書き方にすると変わることを説明する。
[公式の記号一覧](https://doc.rust-lang.org/book/appendix-02-operators.html)は役割の確認に使えるが、
名称だけではコードの説明にならない。次の違いは対象コードに現れたときに区別する。

- 借用とライフタイム：`&T` は型、`&value` は借用、`&pattern` は参照に照合するパターン。
  `'a` はライフタイムの名前になり、`'a'` は文字リテラル、`'outer: loop` は処理の行き先を指定するためのループの名前になる。
- 参照先と演算：`*reference` は参照先をたどり、`a * b` は乗算、`*const T` / `*mut T` は生ポインタ型。
  生ポインタと参照を区別し、`*` の説明のためだけにunsafeコードを持ち込まない。
- 名前と型引数：`::` はパスの各部分を区切り、`::<…>` は式の中でジェネリック引数を指定する。
  どちらも、それだけで実行時の値を操作するものではない。
- クロージャと失敗の伝播：`|x| 式` の縦棒は引数を囲む。`a | b` は演算子、パターンの `A | B` は複数の候補。
  `result?` は失敗を呼び出し側へ伝えるが、`T: ?Sized` の `?` は暗黙の `Sized` 制約を緩める。
- マクロと矢印：`name!(…)` はマクロ呼び出し、`!flag` は真偽値の反転、`-> !` は通常の戻り値を返さない宣言。
  `-> T` は戻り値の型、`パターン => 式` はmatchの分岐と結果の対応を示す。`macro_rules!` の `=>` は
  照合する形と展開内容を区切る。どちらの矢印も、一般的な「ここでreturnする」という指示にはしない。

## 外側の引数とクロージャの引数

```rust
let values = [1, 2, 3];
let remaining = values.into_iter().fold(10, |acc, item| acc - item);
assert_eq!(remaining, 4);

let reversed = values.into_iter().fold(10, |item, acc| acc - item);
assert_eq!(reversed, -8);
```

`into_iter()` が配列から要素を順に取り出すイテレータを作り、`fold` が累積値を更新する。
ここでは要素が `i32` と推論され、配列も `Copy` なので二回目にも `values` を使える。
`String` の配列などでも同じとは限らない。

[`fold`の型定義](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.fold)で重要なのは、
`init: B` と `f: F`、および `F: FnMut(B, Self::Item) -> B` の対応。
`B` は累積値の型、`Self::Item` は要素の型、`F` は渡す処理の型を表す。
`FnMut(...) -> B` は、括弧内の型の引数を受け取り `B` を返す呼び出しが可能であるという条件。
この例ではどちらの値も `i32`。`fold` へは初期値 `10` と処理を渡し、処理へ値を渡すのは `fold` 側になる。

`|acc, item|` の二本の `|` は引数部分を囲み、その後の `acc - item` が返す値。
引数名は自由でも、最初に累積値、次に要素が入るという位置は変わらない。
これにより、繰り返しは `fold` に任せ、値をどう組み合わせるかを呼び出し側から渡せる。
縦棒で囲むのは後で呼ぶ処理の引数。`(acc, item)` へ置き換えると、既存の値からタプルを作る式になり、呼び出せる処理にはならない。

| 呼び出し | 累積値（第1引数） | 要素（第2引数） | `acc - item` の結果 |
|---|---:|---:|---:|
| 1 | 10 | 1 | 9 |
| 2 | 9 | 2 | 7 |
| 3 | 7 | 3 | 4 |

二つ目の式では名前を逆にしたため、実際の計算が「要素 − 累積値」になり、`-9 → 11 → -8` と進む。
両方が同じ型なのでコンパイルできても、意味は違う。
順序はこのAPIの定義で決まる。累積値を受け渡す流れとして理解はできるが、
言語がすべてのAPIにこの順番を強制するわけではなく、歴史的な採用理由を推測で断定しない。

## 型の中の `&` と借用する式の `&`

```rust
fn byte_len(text: &str) -> usize {
    text.len()
}

let name = String::from("Rust");
let length = byte_len(&name);
assert_eq!(length, 4);
assert_eq!(name, "Rust");
```

`text: &str` は「文字列スライスを共有参照で受け取る」引数の宣言。
`&name` は `name` を借用する式で、ここでは `&String` から `&str` への変換が働く。
関数は読むために借りるので、呼び出し後にも `name` が使える。`-> usize` は戻り値の型、
末尾の `text.len()` はセミコロンなしの式なので、その値を返す。
`len()` が返すのはバイト数であり、日本語の文字数を数える関数として説明しない。

次は所有権を移すため、最後の行でE0382になる意図した失敗例。

```compile_fail,E0382
fn consume(text: String) -> usize {
    text.len()
}
let name = String::from("Rust");
let length = consume(name);
println!("{name}: {length}");
```

借用が常に正解なのではない。関数側が値を保持するなど、所有する必要がある場合は値渡しにも理由がある。
`clone()` を選ぶなら、元の値と別に所有する必要と複製の費用を説明する。

## `*` は参照先をたどり、`&mut` は参照先の変更を可能にする

```rust
let mut count = 3;
let borrowed: &mut i32 = &mut count;
*borrowed += 1;
assert_eq!(count, 4);

let reference: &i32 = &count;
let copied: i32 = *reference;
assert_eq!(copied * 2, 8);
```

`*borrowed += 1` は「この参照が指す整数に1を足す」と読む。`&mut count` は `count` を排他的に借りる式で、
`&mut i32` はそのアクセスを型で表す。`mut count` は束縛された値の変更を許す指定。
参照先を変更するために、`borrowed` という束縛自体を `mut` にする必要はない。
その借用をもう使わなくなれば `count` を再び読める。`*` を省いた `borrowed += 1` は、整数ではなく
`&mut i32` に対して未対応の演算を行おうとする。

`let copied = *reference` では、参照先の `i32` の値を読む。`i32` は `Copy` なので値がコピーされる。
`*` は「cloneする」という意味ではない。同じように `&String` 経由で `String` を取り出して所有権を移すことはできない。
`copied * 2` の同じ記号は乗算になる。演算子名の前に、対象の型と記号の位置を示す。
仕様は[演算子の規則](https://doc.rust-lang.org/reference/expressions/operator-expr.html)で確認できる。

## `'a` は返す参照と入力の関係を表す

```rust
fn first<'a>(left: &'a str, _right: &str) -> &'a str {
    left
}

let owner = String::from("Rust");
let selected;
{
    let temporary = String::from("short-lived");
    selected = first(&owner, &temporary);
}
assert_eq!(selected, "Rust");
```

返す文字列を所有しているのは `owner`。関数の宣言は「二つの文字列を借り、最初の入力に結び付いた参照を返す」と読む。
`<'a>` はライフタイムのパラメータを宣言し、`&'a str` はそれを使う。アポストロフィはライフタイム名の目印で、文字を囲む引用符ではない。
`_right` は通常の引数で、先頭のアンダースコアは意図的に使わないことを示す。ここでは入力が二つある場合の違いを見せるために置いている。
二つ目の入力は、`selected` を最後に使う時点まで有効である必要はない。

この記法には、関数の本体を調べなくても、呼び出し側とコンパイラが入力と出力の関係を分かるようにする価値がある。
注釈が `owner` を生かし続けたり、二つの所有者を同じ長さだけ生存させたりするわけではない。
呼び出し側では、`selected` を使う間は `owner` が有効でなければならない。
詳しくは[ライフタイムと省略の規則](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html)を参照する。

次は関係を省いた意図した失敗例。参照の入力が二つあるため、省略の規則だけではどちらから借りた参照を返すか決まらない。
本体が `left` を返すと書いてあっても、この宣言ではエラーになる。

```compile_fail,E0106
fn first(left: &str, _right: &str) -> &str {
    left
}
```

入力が一つなら `fn first(left: &str) -> &str { left }` と注釈を省ける。
省略は決まった関係を適用することであり、ライフタイムやコンパイラの検査をなくすことではない。

## `?` の失敗時の戻り先

```rust
use std::num::ParseIntError;

fn parse_count(text: &str) -> Result<u32, ParseIntError> {
    let count = text.parse::<u32>()?;
    Ok(count)
}

assert_eq!(parse_count("12"), Ok(12));
assert!(parse_count("twelve").is_err());
assert!(parse_count("-1").is_err());
```

`Result<u32, ParseIntError>` は成功時の `u32` と失敗時の `ParseIntError` を区別する型。
`::<u32>` は文字列をどの型へ変換するかを指定する。`?` はこの `Result` が `Ok` なら中の値を取り出し、
`Err` なら `parse_count` からエラーを返す。その場合は最後の `Ok(count)` に進まない。
この例は変換前後でエラー型が同じ。他の型へ伝える場合は変換の条件も必要になる。
[`?`の規則](https://doc.rust-lang.org/reference/expressions/operator-expr.html#the-try-propagation-expression)は、
使う型や囲んでいる関数・クロージャに即して確認する。

### 同じ処理で `::`・`::<…>`・`->`・`=>` を読む

早期リターンを書き下すと次のようになる。成功時と失敗時の両方の分岐を残す。

```rust
use std::num::ParseIntError;

fn parse_count(text: &str) -> Result<u32, ParseIntError> {
    let parsed: Result<u32, ParseIntError> = text.parse();
    let count = match parsed {
        Ok(value) => value,
        Err(error) => return Err(error),
    };
    Ok(count)
}

assert_eq!(parse_count("12"), Ok(12));
assert!(parse_count("twelve").is_err());
```

`std::num::ParseIntError` は型の場所を表すパス。前の例の `String::from` は型に属する関連関数の名前を指定する。
`text.parse()` は値に対するメソッド呼び出し。`::` 自体は関数呼び出しではなく、`.` と自由に入れ替えることもできない。
`text.parse::<u32>()` の `::<u32>` は変換先の型を選ぶ部分。`()` 内の実行時の引数はなく、`text` がメソッドの受け手になる。
この書き換えでは `parsed` に型を書くことで、`parse()` の型を推論できる。
前の例も、`Ok(count)` と宣言した戻り値の型から `count` の型が決まるので、`::<u32>` を省ける。
型引数を明記すると、その場で選んだ型を読み取れる。コンパイルのために常に必要とは限らない。

`-> Result<…>` は関数が返す型を呼び出し側に伝える。`match` 内の `=>` は、パターンと一致したときに評価する式を結ぶ。
`Ok(value)` は成功した数値を `value` に束縛し、右側の `value` がその分岐の結果になる。
`=>` だけで関数を抜けるわけではない。エラー側で関数を抜けるのは `return`。
`?` はエラーを捨てずに、この伝播を短く書ける記法。前の例から単に `?` を消すと `count` に `Result` が残り、
`Ok(count)` が宣言した戻り値の型に合わなくなる。`Option` の `?` は同様に `Some` の中身を取り出すか、
`Option` を返す関数・クロージャから `None` を返す。panicや真偽値の判定ではない。

### 名前の後の `!` と値の前の `!`

```rust
let ready = false;
assert!(!ready);
let message = format!("ready={ready}");
assert_eq!(message, "ready=false");
```

`assert!(…)` は条件が偽ならテストを失敗させるマクロの呼び出し。`!ready` は真偽値の反転で、ここでは `true` になる。
`format!` は書式を指定する構文を受け取り、`String` を作る。マクロの目印があるので、そのマクロの入力の規則で読む。
削除すると通常の呼び出しを試みる形になり、任意の句読点を省いただけの同じ処理にはならない。
これらのマクロに `unsafe` ブロックは不要で、`!` は危険の目印ではない。
意味は置かれた位置や、`!value` のような演算子の場合には対象の型によって決まる。

## 代入とシャドーイングを時間の順に読む

```rust
let mut x = 3;
let before = x;
x = x + 1;
assert_eq!((before, x), (3, 4));
assert!(!(x == x + 1));

let x = 3;
let original = &x;
let x = x + 1;
assert_eq!((*original, x), (3, 4));
```

前半では、右辺の `x` が `3` を読み、`+ 1` で `4` を計算し、左辺の `x` が指す場所を代入で更新する。
`before` は `i32` の値をコピーして保持している。`==` は代入せず比較する記号。
`!(…)` は比較結果を反転し、`assert!` はその条件が成り立つか確かめる。この小さな整数の例ではオーバーフローしない。

後半では、`let` ごとに束縛を新しく作る。最後の右辺の `x` はまだ前の束縛を指し、その値に1を足して新しい `x` を初期化する。
`original` は以前の `3` を参照し続け、`*original` でそれを確認できる。
シャドーイングは名前で見つかる束縛を変えるが、既存の参照を別の値へ向け直すものではない。
だから `let x = x + 1` に `mut` は不要であり、代入の別表記ともいえない。
詳しくは[変数とシャドーイングの説明](https://doc.rust-lang.org/book/ch03-01-variables-and-mutability.html)を参照する。

## ライフタイムの関係と、実行時に選ぶ値は別の情報

```rust
fn choose<'a>(take_left: bool, left: &'a str, right: &'a str) -> &'a str {
    if take_left { left } else { right }
}

let left = String::from("left");
let right = String::from("right");
assert_eq!(choose(true, &left, &right), "left");
assert_eq!(choose(false, &left, &right), "right");
```

`<'a>` でライフタイムのパラメータを宣言し、二つの入力の参照と出力で使っている。
この宣言により、呼び出し側は両方の入力が共通の有効期間の条件を満たす間、結果の参照を使える。
どちらの文字列を返すかは関数の本体が選ぶ。`'a` が選ぶのではなく、所有者の寿命が等しいという指定でもない。
`if … { … } else { … }` は式であり、選ばれた分岐の参照を文字列のコピーなしに返す。

同じ関数へ `true` を渡しても、二つ目の所有者の寿命が短い場合はどうなるか。次はE0597になる意図した失敗例。

```compile_fail,E0597
fn choose<'a>(take_left: bool, left: &'a str, right: &'a str) -> &'a str {
    if take_left { left } else { right }
}

let left = String::from("left");
let selected;
{
    let right = String::from("right");
    selected = choose(true, &left, &right);
}
assert_eq!(selected, "left");
```

呼び出し側の検査は、宣言された関係に従う。この `true` に合わせて本体を読み替え、結果を保証してくれるわけではない。
宣言は結果を両方の入力へ結び付けているが、`right` は後の `selected` の使用まで有効でいられない。
この失敗は、この実行で本当に無効な参照が返るという証明ではない。呼び出し側が宣言から確かめられる範囲を示している。
常に最初の入力を返す操作なら、前の `first` のように二つ目のライフタイムを独立させる。
どちらも返し得るなら、所有者を必要な間保持するか、目的に応じて所有する文字列を返す。
`'static` を足してもローカルの所有者の寿命は延びない。
詳しくは[ライフタイムの説明](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html)を参照する。

## 再帰型・間接参照・再帰呼び出しが解く問題は異なる

次はE0072になる意図した失敗例。値の中に同じ型の値全体を直接含めようとしており、
コンパイラが有限のメモリ配置を決められない。

```compile_fail,E0072
enum List {
    End,
    Link(i32, List),
}
```

`Box` を挟むと、各リンクが持つのは次のリストの値を所有する固定サイズのポインタになる。

```rust
enum List {
    End,
    Link(i32, Box<List>),
}

impl List {
    fn sum(&self) -> i32 {
        match self {
            Self::End => 0,
            Self::Link(value, rest) => *value + rest.sum(),
        }
    }
}

let values = List::Link(2, Box::new(List::Link(3, Box::new(List::End))));
assert_eq!(values.sum(), 5);
assert_eq!(values.sum(), 5);
```

各部分を役割から読む。

| 部分 | 役割と、その結果 |
|---|---|
| `Box<List>` | 別に確保した `List` を所有する。`<List>` は型引数で、関数呼び出しではない。間接参照によりenum自体の配置が有限になるが、リスト全体のメモリ量が一定になるわけではない |
| `List::Link(…)` / `Box::new(…)` | バリアントを作る／メモリを確保して中身を所有する。`::` は名前を選び、`()` は値を渡す |
| `&self` / `match self` | リストを読むために借りる。共有参照へこのパターンを照合すると、`value: &i32`、`rest: &Box<List>` と自動的に参照で束縛され、リストを分解して所有権を移さない |
| `*value` / `rest.sum()` | 参照先の整数を読む／次のリストを借りて再帰呼び出しする。メソッドの探索でBoxの参照外しや必要な借用が行われる |
| `Self::End => 0` / `-> i32` | 終端の分岐は0を返し、関数の宣言は整数の結果を示す。もう一方の分岐は現在の数と残りの合計を足す |

このリストの呼び出しは `2 + (3 + 0)` と進む。最初の呼び出しは借りるだけなので、二回目の `sum()` も使える。
`Box` は型のメモリ配置の問題を解き、再帰呼び出しは値をたどり、`End` はこの走査を終える。
型は自分自身を参照するが、この値は有限の連なりであり、ノードが自分自身へ戻るものではない。
再帰型をループで処理することもでき、再帰関数が必ず再帰型を扱うわけでもない。
これは説明用の例で、`Vec` を連結リストに置き換える推奨ではない。深い再帰呼び出しはスタックを使い切る場合があり、
ヒープを経由しても実行の深さが無制限になるわけではない。この小さな合計ではオーバーフローしない。
参照：[Boxによる再帰型](https://doc.rust-lang.org/book/ch15-01-box.html)、
[パターンの束縛モード](https://doc.rust-lang.org/reference/patterns.html#binding-modes)、
[メソッドの探索](https://doc.rust-lang.org/reference/expressions/method-call-expr.html)。

## マクロの展開は構文を扱い、関数呼び出しは評価済みの値を受け取る

```rust
macro_rules! twice {
    ($value:expr) => { $value + $value };
}

let mut calls = 0;
let result = twice!({ calls += 1; calls });
assert_eq!((result, calls), (3, 2));

fn twice_value(value: i32) -> i32 {
    value + value
}

let mut calls = 0;
let result = twice_value({ calls += 1; calls });
assert_eq!((result, calls), (2, 1));
```

`$value:expr` は式の構文を受け取る。`=>` は照合する形と展開内容を分け、各 `$value` が受け取った構文を挿入する。
そのため `twice!(…)` はブロックを二回評価する式へ展開され、一回目は `1`、二回目は `2` になる。
ブロック内の `calls += 1;` が状態を変え、末尾の `calls` が値を返す。
展開そのものが `calls` を増やすのではなく、展開後の式を実行すると増える。

通常の関数呼び出しでは、まず引数のブロックを一回評価して `1` を得てから、関数の中でその整数を二回使って足す。
`value` を二回使っても、呼び出し側のブロックを評価し直すわけではない。
`-> i32` は結果の型を示し、マクロの `=>` のように構文の変換を定義するものではない。

このマクロは、同じ式を繰り返し評価する落とし穴を意図的に見せている。
マクロが必ず引数を二回評価するわけではない。展開先で一度だけ値を束縛すれば避けられるが、この例では動作の変更になる。
`!` や `$` の説明では、作られる構文と、その式が実行されるタイミングを分ける。
感嘆符やこの展開が、再帰呼び出しや、実行時の意味を自由に定義し直せることを表すわけではない。
詳しくは[マクロ展開の規則](https://doc.rust-lang.org/reference/macros-by-example.html)を参照する。

## 関数の宣言から読み、実装で検索方法を確かめる

```rust
struct Record {
    key: String,
    score: u32,
}

fn find_record<'a>(records: &'a [Record], key: &str) -> Option<&'a Record> {
    records.iter().find(|record| record.key.as_str() == key)
}

let records = [
    Record { key: String::from("rust"), score: 1 },
    Record { key: String::from("rust"), score: 2 },
];
let selected;
{
    let key = String::from("rust");
    selected = find_record(&records, &key);
}
let score = match selected {
    Some(record) => Some(record.score),
    None => None,
};
assert_eq!(score, Some(1));
assert!(find_record(&records, "missing").is_none());
```

まず本体を隠して宣言を読む。`&[Record]` はスライスの借用、`&str` は検索キーの借用で、
`Option<&'a Record>` は値がない場合か、`records` と宣言した関係を持つ参照を返す場合を表す。
ローカルのキーは `selected` を使う前に破棄される。そのライフタイムは出力に結び付いていない。
宣言だけでは、重複のどちらを選ぶか、どれくらいの処理で検索するかまでは分からない。

次に本体を読む。`iter()` の要素は `&Record`。`find` は判定のために候補を借りるので、クロージャの引数は `&&Record` になる。
フィールドへのアクセスで参照が自動的にたどられて `key` に届き、`as_str()` が文字列を借り、`==` が検索キーと比較する。
`find` が返すのは最初に一致したイテレータの要素、つまり `&Record` であり、判定時だけ使った `&&Record` ではない。
末尾の式にセミコロンがないため、その `Option` が関数の戻り値になる。

本体からは、先頭から順に探して最初の一致で止まると分かり、重複を含むテストでも確認できる。
検索は文字列のcloneや結果コレクションの確保を行わない。テスト入力を作る `String::from` の確保は検索の外側で起きる。
最悪の場合の処理量はレコード数だけでなく、文字列比較の費用にも左右される。実際の所要時間には測定が必要。
`selected` に対する `match` は `u32` のスコアをコピーし、値がないときにpanicせず `Some` と `None` の違いを保つ。
参照：[`Iterator::find`](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.find)、
[`String::as_str`](https://doc.rust-lang.org/std/string/struct.String.html#method.as_str)。

## エラーを返しても実行済みの作用は取り消されない

状態を変えた後で確認応答が失われる操作を、意図的に失敗する関数で表す。
ローカルな制御の流れを確かめる例であり、ネットワークや永続ストレージの実装ではない。

```rust
#[derive(Debug, PartialEq)]
enum SubmitError {
    AcknowledgementLost,
}

fn submit(applied: &mut u32) -> Result<(), SubmitError> {
    *applied += 1;
    Err(SubmitError::AcknowledgementLost)
}

fn attempt(applied: &mut u32) -> Result<&'static str, SubmitError> {
    submit(applied)?;
    Ok("confirmed")
}

let mut applied = 0;
assert_eq!(attempt(&mut applied), Err(SubmitError::AcknowledgementLost));
assert_eq!(applied, 1);
assert!(attempt(&mut applied).is_err());
assert_eq!(applied, 2);
```

`&mut u32` により、`submit` は呼び出し側のカウンターを変更できる。
`*applied += 1` が作用を起こしてからエラーを作る。`?` は `Ok("confirmed")` に届く前に
`attempt` から戻るが、`Result` も `?` もカウンターを以前の値に戻さない。
再試行するともう一度変更される。このテストは実際の状態を見られるが、外部への呼び出しでは、
応答を失った後にそれを確認できない場合がある。復旧方針には「エラーなら再試行」以上の情報が必要になる。
リクエストの識別、結果の不明、検証範囲には[システムの境界](systems-thinking_ja.md)を使う。

## 検証によって型に狭い意味を与える

ここでは数量の条件を1から100までと決める。`u32` として読み取れることとは異なる条件になる。

```rust
mod order {
    #[derive(Debug, PartialEq)]
    pub struct Quantity(u32);

    #[derive(Debug, PartialEq)]
    pub enum QuantityError {
        OutOfRange,
    }

    impl TryFrom<u32> for Quantity {
        type Error = QuantityError;

        fn try_from(value: u32) -> Result<Self, Self::Error> {
            if (1..=100).contains(&value) {
                Ok(Self(value))
            } else {
                Err(QuantityError::OutOfRange)
            }
        }
    }

    impl Quantity {
        pub fn get(&self) -> u32 {
            self.0
        }
    }
}

use order::{Quantity, QuantityError};

assert_eq!("0".parse::<u32>(), Ok(0));
assert_eq!(Quantity::try_from(0), Err(QuantityError::OutOfRange));
assert_eq!(Quantity::try_from(101), Err(QuantityError::OutOfRange));
assert_eq!(Quantity::try_from(1).map(|q| q.get()), Ok(1));
assert_eq!(Quantity::try_from(100).map(|q| q.get()), Ok(100));
assert!("many".parse::<u32>().is_err());
```

`struct Quantity(u32)` は別の型を定義し、フィールドを `order` の内側だけに公開している。
`pub` が公開するのは型と選んだ操作であり、モジュール外からフィールドを直接指定して作れるわけではない。
`TryFrom<u32>` は失敗し得る変換を表し、`type Error` はそのエラー型を指定する。
`1..=100` は両端を含み、`Self(value)` は検証した型の値を作る。
`get(&self)` は借用を通して読み、`u32` をコピーする。

利点は、範囲の判断を境界に集め、後の処理が制限のない数値を何度も解釈せず `Quantity` を受け取れること。
この性質は、モジュール全体の生成・変更経路に依存する。検証しないコンストラクター、setter、
deriveによるデシリアライズを追加すると、根拠が崩れる場合がある。
また、この型で包んでも在庫の存在や注文する権限までは分からない。
アサーションでは、数値として読めない場合、数量の条件を満たさない場合、受理する両端を別々に確認している。

## 実装を変えても必要な動作を保つ

```rust
fn count_positive_loop(values: &[i32]) -> usize {
    let mut count = 0;
    for value in values {
        if *value > 0 {
            count += 1;
        }
    }
    count
}

fn count_positive_iter(values: &[i32]) -> usize {
    values.iter().filter(|value| **value > 0).count()
}

for (input, expected) in [(&[-1, 0, 2, 3][..], 2), (&[][..], 0), (&[0][..], 0)] {
    assert_eq!(count_positive_loop(input), expected);
    assert_eq!(count_positive_iter(input), expected);
}
```

要求は「正の整数の個数」で、`0` は含まない。二つの実装が一致するだけでなく、要求から決めた期待値と比べる。
`> 0` を `>= 0` に変える誤りは、`0` を含む例で見つかる。
`iter()` の要素は `&i32`、`filter` の判定処理は要素への参照を受けるので `value` は `&&i32`。
`**value` は二段の参照をたどる式になる。この説明がまだ重ければ、先にループ版で型と条件を追う。
この有限個の例の成功を、あらゆる入力や副作用まで同等である証明とは扱わない。
