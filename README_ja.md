# Rust Learning Lab

[English](README.md) | 日本語

**構文の意味から、自分で確かめる力まで。**

Rustのコードを、記号の読み方・引数の対応・型と所有権・その仕組みが必要な理由から説明する、
Codex / Claude Code向けの学習・レビュー支援プラグインです。読むだけで終わらず、小さな変更・実装・デバッグへつなげます。
レビューでは問題の場所・条件・影響と修正理由を示し、詳しい設計解説は読み返せるMarkdownに残します。
指示は日本語で記述していますが、回答には会話から判断した言語を使い、文書は指定や既存の慣例に合わせます。

## インストール

プラグイン機能を備えた各CLIを使用してください。説明だけならRustのインストールは不要です。
コードを実行する場合は対象プロジェクトに合うRust toolchainを使います。
配布検証には Codex CLI 0.157.0、Claude Code 2.1.282、Rust 1.98.1を使用しています。

### Codex

```sh
codex plugin marketplace add nwiizo/rust-learning-lab
codex plugin add rust-learning-lab@rust-learning-lab
```

インストール後は新しい会話を開き、`$learn-rust` を指定します。
スキル選択画面でも `Learn Rust` を選べます。

### Claude Code

```sh
claude plugin marketplace add nwiizo/rust-learning-lab
claude plugin install rust-learning-lab@rust-learning-lab
```

新しい会話で `/rust-learning-lab:learn-rust` に続けて質問します。

配布形式は[Codexの公式仕様](https://developers.openai.com/plugins/build/plugins)と
[Claude Codeの公式仕様](https://code.claude.com/docs/en/plugin-marketplaces)に従っています。
このリポジトリ自身が配布用marketplaceを提供します。公式カタログへの掲載を意味しません。

## 使い方

Codexでの例です。Claude Codeでは先頭を `/rust-learning-lab:learn-rust` に置き換えます。

```text
$learn-rust fold(10, |acc, item| acc - item) は、誰がどの引数を渡しますか？
なぜこの順番なのか、型と実行の流れから説明してください。

$learn-rust このコードの &str と &name はどう違いますか？
記号の意味と、借用が必要になる背景から知りたいです。

$learn-rust Result と ? を練習したいです。まずヒントだけ出し、私の回答を待ってください。

$learn-rust この差分をレビューし、設計判断と構文の背景まで説明してください。
docs/parser-learning-review.md に残してください。
```

通常の質問には先に答えます。練習を強制したり、説明の依頼だけでファイルを変更したりしません。
コードやエラー全文、分かっている範囲があれば、一緒に渡してください。
リポジトリを対象にした詳しい学習レビューでは、既定で `docs/rust-learning-review.md` などに文書を残します。
保存先の指定や「チャットのみ」「読み取り専用」を優先し、ソースの修正は修正も依頼されたときに行います。
[生成するレビュー文書の短い例](plugins/rust-learning-lab/skills/learn-rust/references/review-example.md)も参照できます。

## 学べること

- **構文と背景**：記号の役割、関数の形、省略された型、解決する問題。仕様・慣習・推測を分けます。
- **引数と流れ**：`self`、値と参照、クロージャ、引数順、戻り値、途中終了を具体値で追います。
- **理解から実践へ**：追跡・予測・部分補完・自力での変更から、つまずきに合う練習を選びます。
- **レビューと検証**：具体的な不具合や変更の影響を調べ、構文の背景・型・値の流れから指摘の理由を説明します。
- **継続学習**：説明済みと自力で確認できたことを分け、次に試す一点を残します。

10件のプログラミング教育・AI支援研究と、2件の記憶研究を参考にしています。
対象・結果・限界を[研究ノート](plugins/rust-learning-lab/skills/learn-rust/references/learning-evidence.md)に記載しています。
他言語や授業の結果をRustの個別学習へそのまま一般化せず、このプラグイン自体の学習効果は未測定です。

## 開発と確認

`plugins/rust-learning-lab/skills/learn-rust/` が両ホストで共用するスキルです。
補助資料は質問に必要なときだけ読み込みます。MCPサーバー、フック、独自ランタイムは含みません。
プラグイン自身によるデータ収集はありません。会話やツール実行には利用するホストの設定が適用されます。

```sh
rustdoc --test --edition 2024 plugins/rust-learning-lab/skills/learn-rust/references/worked-examples.md
rustdoc --test --edition 2024 plugins/rust-learning-lab/skills/learn-rust/references/rust-2024.md
rustdoc --test --edition 2024 plugins/rust-learning-lab/skills/learn-rust/references/review-example.md
claude plugin validate .
claude plugin validate plugins/rust-learning-lab
```

CIは配布メタデータの整合とRust例を確認します。[対話の確認課題](docs/evaluation.md)は手動評価用です。
構文検査やコード例の成功だけでは、モデルの回答品質や学習効果を実証できません。
変更時は両ホストのmanifestのversionを揃えて更新してください。
READMEの利用方法を変えた場合は、英語版と日本語版を合わせて更新してください。

MIT License。リンク先の論文・外部資料にはそれぞれの利用条件が適用されます。
