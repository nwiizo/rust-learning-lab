# Rust Learning Lab

[English](README.md) | 日本語

**構文の意味から、自分で確かめる力まで。**

Rustのコードを、記号の読み方・引数の対応・型と所有権・その仕組みが必要な理由から説明する、
Codex / Claude Code向けの学習・レビュー支援プラグインです。読むだけで終わらず、小さな変更・実装・デバッグへつなげます。
レビューでは問題の場所・条件・影響と修正理由を示し、詳しい設計解説は読み返せるMarkdownに残します。
スキルの指示と8種類すべてのリファレンスを日英で用意しています。回答には会話から判断した言語を使い、文書は指定や既存の慣例に合わせます。

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

$learn-rust owner/repo の PR #123 から教育向けのレビュー文書を作ってください。
変更前後の動作、構文、引数の順番、設計判断まで説明してください。
docs/rust-learning-review/pr-123-parser.md に保存し、ディレクトリの目次も更新してください。

$learn-rust コミット <sha>（または範囲 <base>..<head>）から学習文書を作ってください。
比較した版を記録し、所有権とエラー処理を説明して、問題がないかもレビューしてください。
docs/rust-learning-review/ の下に保存し、ディレクトリの目次も更新してください。
```

通常の質問には先に答えます。練習を強制したり、説明の依頼だけでファイルを変更したりしません。
コードやエラー全文、分かっている範囲があれば、一緒に渡してください。
例のリポジトリ、PR番号、コミットの指定は実際の対象へ置き換えてください。PRのURLでも指定できます。
指定した差分とその版の関連コードを読み、説明された意図と確認できた動作を分けます。

詳しい学習レビューは、既定で `docs/rust-learning-review/` の下にMarkdownを分けて保存します。
作成・更新のたびに同ディレクトリの `README.md`（英語）と `README_ja.md`（日本語）の目次へ、
文書の相対リンク、参照元のPR・コミット、本文の言語、更新日を反映します。
目次には、特定の版を教材にした教育向けの資料であり、製品の文書やリポジトリの指示ではないことを記載します。
保存先の指定や「チャットのみ」「読み取り専用」を優先します。
ソースの修正は修正も依頼されたときに行います。
[生成するレビュー文書の短い例](plugins/rust-learning-lab/skills/learn-rust/references/review-example_ja.md)も参照できます。

最初の文書作成時に、既存設定を保ちながら `.gitignore` と `.rgignore` にディレクトリを追加し、
プロジェクトのエージェント向け指示にも通常のコンテキストへ読み込まない方針を残します。目次も除外対象です。
通常の作業では参照せず、エージェントのメモリ・ルール・要約・製品の文書へも自動転記しない方針です。
指定された学習作業では該当文書を読めます。あらゆる直接読み取りを禁止する設定ではありません。
既に追跡されているファイルは追跡されたままで、設定変更を禁止する指示があればそちらを優先します。
常駐の目次更新機能は追加せず、スキルがレビュー文書を書く際に目次を更新します。

## 学べること

- **構文と背景**：記号の役割、関数の形、省略された型、解決する問題。仕様・慣習・推測を分けます。
- **引数と流れ**：`self`、値と参照、クロージャ、引数順、戻り値、途中終了を具体値で追います。
- **理解から実践へ**：追跡・予測・部分補完・自力での変更から、つまずきに合う練習を選びます。
- **レビューと検証**：具体的な不具合や変更の影響を調べ、構文の背景・型・値の流れから指摘の理由を説明します。
- **継続学習**：説明済みと自力で確認できたことを分け、次に試す一点を残します。

10件のプログラミング教育・AI支援研究と、2件の記憶研究を参考にしています。
対象・結果・限界を[研究ノート](plugins/rust-learning-lab/skills/learn-rust/references/learning-evidence_ja.md)に記載しています。
他言語や授業の結果をRustの個別学習へそのまま一般化せず、このプラグイン自体の学習効果は未測定です。

## 開発と確認

`plugins/rust-learning-lab/skills/learn-rust/` が両ホストで共用するスキルです。
補助資料は質問に必要なときだけ、今作る説明・文書の言語に合わせて読み込みます。
英語版と `_ja.md` の日本語版は相互リンクし、日本語版内の参照も日本語の資料へつなぎます。
その他の回答言語では英語の資料を参照し、回答言語は変えません。通常は両言語を重複して読み込みません。
MCPサーバー、フック、独自ランタイムは含みません。
プラグイン自身によるデータ収集はありません。会話やツール実行には利用するホストの設定が適用されます。

```sh
rustdoc --test --edition 2024 plugins/rust-learning-lab/skills/learn-rust/references/worked-examples_ja.md
rustdoc --test --edition 2024 plugins/rust-learning-lab/skills/learn-rust/references/rust-2024_ja.md
rustdoc --test --edition 2024 plugins/rust-learning-lab/skills/learn-rust/references/review-example_ja.md
claude plugin validate .
claude plugin validate plugins/rust-learning-lab
```

英語版にも同じdoctestを実行してください。CIは配布メタデータ、日英のファイル対応、Rustコードの一致、両言語の実行例を確認します。
[対話の確認課題](docs/evaluation_ja.md)は手動評価用です。
構文検査やコード例の成功だけでは、モデルの回答品質や学習効果を実証できません。
変更時は両ホストのmanifestのversionを揃えて更新してください。
意味を変えた場合は、英語版と日本語版を合わせて更新してください。
日英資料、レビュー文書、検証、公開の方針は[リポジトリのルール](AGENTS_ja.md)に記載し、Claude Codeからも同じ方針を読み込みます。
資料の使い分けは[スキルの説明](plugins/rust-learning-lab/skills/learn-rust/SKILL_ja.md)を参照してください。

MIT License。リンク先の論文・外部資料にはそれぞれの利用条件が適用されます。
