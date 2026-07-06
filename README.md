# docs

misora-connect のユーザー・開発者向けドキュメントリポジトリ。

## 本番サイト

<https://misora-connect.github.io/docs/>

`main` ブランチに push すると GitHub Pages へ自動デプロイされる。

## リポジトリ構成

```
docs/                   # ドキュメント本体 (Jekyll)
├── index.md            # トップページ
├── _config.yml         # Jekyll 設定
└── api/                # API ドキュメント
    ├── index.md        # API トップ
    ├── getting-started.md
    ├── user-guide.md
    ├── reference.md
    └── spec/           # OpenAPI 仕様
        ├── openapi.yaml
        └── index.html  # Swagger UI
.github/workflows/
├── pages.yml           # GitHub Pages デプロイ
├── preview.yml         # PR プレビュー (S3 + CloudFront)
└── claude.yml          # Claude Code 自動対応
```

## 開発フロー

### ローカルプレビュー

Jekyll でビルドされるため、ローカルで確認する場合:

```bash
cd docs
bundle exec jekyll serve
```

### PR プレビュー

PR を作成すると、自動で `https://docs-pr-<PR番号>.dev.misora-connect.com` にプレビューがデプロイされる。PR コメントにプレビュー URL が投稿される。

## `@claude` ワークフロー

このリポジトリでは [Claude Code Action](https://github.com/anthropics/claude-code-action) を使い、Issue や PR のコメントで `@claude` とメンションすることで AI にタスクを依頼できる。

### 使い方

1. **Issue で依頼する** — Issue 本文またはコメントに `@claude` を含めると、Claude がコードを変更し、自動で PR を作成する。
2. **PR で依頼する** — PR のコメントに `@claude` を含めると、その PR のブランチに直接コミットする。

### 動作の流れ

1. `@claude` を検知すると GitHub Actions が起動
2. Claude がリポジトリを読み、指示に従ってファイルを編集
3. 変更をブランチにプッシュし、Issue の場合は PR を自動作成

### 対応できること

- ドキュメントの加筆・修正
- コードレビューや質問への回答
- 新規ドキュメントの作成

## ライセンス

[Apache License 2.0](LICENSE)
