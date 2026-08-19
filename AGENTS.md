# AGENTS.md

このリポジトリで作業するコーディングエージェント向けのガイドです。作業前に一読してください。

## プロジェクト概要

日本語の GitHub Issue を英語に自動翻訳し、Issue にコメントとして投稿する GitHub Actions ワークフローのみで構成されるリポジトリです。アプリケーションコードやパッケージマニフェスト（`package.json` など）は存在しません。

翻訳には GitHub Models を利用し、追加の外部シークレットは不要です（ワークフローは自動提供される `GITHUB_TOKEN` を使用します）。

## プロジェクト構成 / エントリポイント

```
.
├── .github/
│   └── workflows/
│       └── issue-translation.yml   # 唯一の実体。Issue 翻訳ワークフロー
├── README.md
└── AGENTS.md
```

- 実質的なエントリポイントは `.github/workflows/issue-translation.yml` です。ロジックの変更・調査はまずこのファイルを確認してください。
- ワークフローの起動トリガーは `issues` イベントの `opened` / `labeled` です。
- 起動条件（`jobs.translate.if`）: `translate:en` ラベルが対象で、かつ `translated:en` ラベルが無いこと。

## セットアップ

- 特別なローカルセットアップは不要です。依存関係のインストール手順やパッケージマネージャの設定は存在しません。
- ワークフローは GitHub 上で実行されます。ローカルでアプリを起動する仕組みはありません。
- 必要な権限はワークフロー内で定義されています: `contents: read`, `issues: write`, `models: read`。

## ビルド / テスト / lint / 型チェック

- **ビルド**: なし（ビルド対象のコードやビルドスクリプトはありません）。
- **テスト**: 自動テストスイートはありません。動作確認は、対象リポジトリの Issue に `translate:en` ラベルを付けて GitHub Actions の実行結果を確認する形で行います。
- **lint / 型チェック**: リポジトリに設定・スクリプトはありません。ワークフロー YAML を検証する場合は、任意で `actionlint` などのサードパーティツールを利用できます（同梱していません。実在するリポジトリ内コマンドとして紹介しないでください）。

実在しないコマンド（`npm test`、`npm run lint` など）を README や作業説明に記載しないでください。

## コーディング規約

- 変更対象は主に YAML（`.github/workflows/*.yml`）です。既存のインデント（スペース 2）とスタイルに合わせてください。
- ワークフロー内の JavaScript（`actions/github-script`）は Node.js ランタイムで実行されます。既存の記述スタイル（`const`、`await`、`context.repo` などの利用）に合わせてください。
- 翻訳済み判定に使うマーカー文字列 `<!-- translated:en -->` と `translated:en` ラベルは、重複翻訳防止の要です。名称を変更する場合は precheck ステップ・コメント本文・ラベル付与の全箇所を整合させてください。
- コミットメッセージは既存の履歴に倣い、`type(scope): 説明` 形式（例: `fix(workflow): ...`、`feat(build): ...`）を用いてください。説明は日本語で記述されています。

## 注意点

- GitHub Models のモデル名は現在 `openai/gpt-4.1-mini` を使用しています。過去に別モデル（`gpt-5-nano` など）を試した経緯が履歴に残っています。モデルを変更する場合は、`Run AI Inference` ステップの `model` と、フォールバック用の `INFERENCE_MODEL` env の両方を更新してください。
- `actions/ai-inference` の出力が空になるケースに備え、GitHub Models エンドポイントへの直接呼び出しフォールバックが実装されています。片方だけを変更しないよう注意してください。
- 同時実行制御（`concurrency`）により同一 Issue の処理は直列化されます。
- ドキュメント（README.md / AGENTS.md）の記述言語は日本語です。既存のコミットメッセージや文脈に合わせてください。
- 実際のコードと設定から裏付けを取り、憶測で機能やコマンドを追記しないでください。
