# docs

misora-connect のユーザー・開発者向けドキュメントリポジトリ。

## 本番サイト

<https://misora-connect.github.io/docs/>

`main` ブランチに push すると GitHub Pages へ自動デプロイされる。

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

Issue や PR コメントに `@claude` を含めると、AI (Claude Code) が自動で対応する。

### 仕組み

1. Issue / コメントに `@claude` を記載して投稿する
2. GitHub Actions が起動し、Claude Code がリクエストを分析する
3. 必要に応じてコード変更を行い、ブランチを作成する
4. 変更がある場合は自動で PR が作成される (Issue にリンク)

### 使い方の例

- Issue 本文に `@claude` を書いて新規作成 → Issue の内容に応じて対応
- PR のコメントで `@claude ここをレビューして` → コードレビューを実施
- Issue コメントで `@claude この修正をして` → 修正を実施して PR を作成

### 注意事項

- Claude はコメント内の指示に従って動作する。指示は具体的に書くとよい
- 対応結果は GitHub コメントで報告される
- 作成された PR は人間がレビュー・マージする
