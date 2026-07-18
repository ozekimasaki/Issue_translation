# Issue Translation

日本語で書かれた GitHub Issue を、ラベルを付けるだけで英語に自動翻訳し、Issue にコメントとして投稿する GitHub Actions ワークフローです。

翻訳には [GitHub Models](https://docs.github.com/en/github-models) を利用しており、外部サービスの API キーを別途用意する必要はありません。

## 概要

このリポジトリは、単一の GitHub Actions ワークフロー（`.github/workflows/issue-translation.yml`）のみで構成されています。アプリケーションコードやビルド成果物は含まれていません。

Issue に `translate:en` ラベルが付与される（または `translate:en` ラベル付きで Issue が作成される）と、ワークフローが起動し、Issue のタイトルと本文を英語に翻訳して、翻訳結果を新しいコメントとして投稿します。

## 主な機能

- **ラベルによる起動**: Issue の作成（`opened`）またはラベル付与（`labeled`）で起動し、`translate:en` ラベルが付いている場合のみ翻訳を実行します。
- **英語への翻訳**: GitHub Models（既定モデル: `openai/gpt-4.1-mini`）を用いて、タイトルと本文を英語に翻訳します。Markdown 構造・コードブロック・リンクは保持されます。
- **翻訳結果のコメント投稿**: 翻訳結果を `## Title` / `## Body` の見出し付き Markdown で Issue にコメントします。
- **重複翻訳の防止**: 翻訳済みコメントには `<!-- translated:en -->` マーカーを埋め込み、`translated:en` ラベルを付与します。既にマーカーまたはラベルが存在する場合は翻訳をスキップします。
- **フォールバック処理**: `actions/ai-inference` の出力が空だった場合、GitHub Models の推論エンドポイント（`https://models.github.ai/inference`）を直接呼び出して翻訳を試みます。
- **同時実行制御**: 同一 Issue に対する処理は `concurrency` グループで直列化し、進行中の実行はキャンセルされます。

## 要件

- 対象の Issue やワークフローを持つ **GitHub リポジトリ**。
- リポジトリで **GitHub Models** が利用可能であること（ワークフローは `models: read` 権限を要求します）。
- 追加の外部シークレットは不要です。ワークフローは自動的に提供される `GITHUB_TOKEN`（`github.token`）を使用します。

## インストール / 導入

このリポジトリ自体をクローンして実行する種類のツールではなく、ワークフロー定義を対象リポジトリに配置して使用します。

1. `.github/workflows/issue-translation.yml` を、翻訳を有効にしたいリポジトリの同じパスにコピーします。
2. 対象リポジトリで GitHub Models が有効になっていることを確認します。
3. リポジトリに `translate:en` ラベルを用意します（存在しない場合は Issue にラベルを付けた時点で作成されます）。

## 使い方

1. 日本語で Issue を作成します。
2. その Issue に `translate:en` ラベルを付与します（または最初から `translate:en` ラベルを付けて作成します）。
3. ワークフローが起動し、英訳結果が Issue に新しいコメントとして投稿されます。
4. 翻訳が完了すると `translated:en` ラベルが付与され、以降の再翻訳はスキップされます。

投稿されるコメントの例:

```markdown
<!-- translated:en -->

### Translation

## Title
- <English title>

## Body
- <English translation of the body>
```

## 開発コマンド

このリポジトリにはビルド・テスト・lint・型チェックのためのツールチェーンやスクリプトは含まれていません（`package.json` 等のマニフェストは存在しません）。

ワークフロー YAML を編集した場合の確認は、実際に対象リポジトリの Issue に `translate:en` ラベルを付けて Actions の実行結果を確認する方法が中心となります。任意で、YAML の構文チェックには `actionlint` などのサードパーティツールを利用できます（本リポジトリには同梱していません）。

## 構成

```
.
├── .github/
│   └── workflows/
│       └── issue-translation.yml   # Issue 翻訳ワークフロー
└── README.md
```

- `.github/workflows/issue-translation.yml`: Issue を英訳して Issue にコメントするワークフロー本体。

### ワークフローのステップ

1. **Check translation status (marker/label)**: `translated:en` ラベルまたは `<!-- translated:en -->` マーカーの有無を調べ、翻訳をスキップするか判定します。
2. **Run AI Inference (translate to English)**: `actions/ai-inference@v2` で翻訳を実行します（`continue-on-error: true`）。
3. **Comment translation to the issue**: 翻訳結果を Issue にコメントします。出力が空の場合は GitHub Models を直接呼び出すフォールバックを行います。
4. **Add translated:en label**: `translated:en` ラベルを付与します。

## ライセンス

ライセンスは指定されていません。

## 補足

- 翻訳の起動条件・モデル・プロンプト・出力形式は、いずれも `.github/workflows/issue-translation.yml` に定義されています。詳細な挙動を確認・変更する場合はこのファイルを参照してください。
