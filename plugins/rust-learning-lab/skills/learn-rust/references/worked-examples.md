# 実行できる説明例

引数順・借用・早期リターンを具体的に説明するときの対比例。
Rust 2024で `rustdoc --test --edition 2024 <このファイル>` を実行すると、成功例と意図したコンパイル失敗を確認できる。
`assert_eq!` は二つの値が等しいことを確認し、違う場合にテストを失敗させるマクロ。
例の値や説明量は学習者の質問へ合わせる。

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
