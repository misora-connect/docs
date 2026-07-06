# docs

misora-connect のユーザーおよび開発者向けドキュメントを管理するリポジトリです。

## 本番サイト

<https://misora-connect.github.io/docs/>

`main` ブランチへ push すると GitHub Pages へ自動デプロイされます。

## リポジトリ構成

```
docs/                  ドキュメント本体 (Jekyll)
├── index.md           トップページ
├── _config.yml        Jekyll 設定
└── api/               API ドキュメント
    ├── getting-started.md
    ├── user-guide.md
    ├── reference.md
    └── spec/          OpenAPI 仕様 (YAML + Swagger UI)
.github/workflows/     CI/CD
├── pages.yml          GitHub Pages デプロイ
├── preview.yml        PR プレビュー (S3 + CloudFront)
└── claude.yml         Claude Code 自動対応
CLAUDE.md              Claude Code 向けリポジトリガイド
LICENSE                Apache License 2.0
```

## `@claude` ワークフロー

Issue や PR のコメントに `@claude` を含めると、Claude Code が自動で対応します。

### 仕組み

1. Issue またはコメントに `@claude` を記載して投稿します。
2. GitHub Actions が起動し、Claude Code がリクエストを分析します。
3. 必要に応じてコードを変更し、作業ブランチを作成します。
4. 変更がある場合、Issue にリンクする形で PR が自動作成されます。

### 使い方の例

- Issue 本文に `@claude` を書いて新規作成すると、内容に応じて対応します。
- PR コメントで `@claude ここをレビューして` と書くと、コードレビューを実施します。
- Issue コメントで `@claude この修正をして` と書くと、修正を実施して PR を作成します。

### 注意事項

- Claude はコメント内の指示に従って動作するため、指示は具体的に書いてください。
- 対応結果は GitHub コメントで報告されます。
- 作成された PR は人間がレビューし、マージします。
