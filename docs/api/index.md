---
title: Misora Connect API ドキュメント
---

# Misora Connect API ドキュメント

Misora Connect API は REST API です。
SIM カードの管理、通信量の統計、データエクスポート、リチャージ（容量追加）の各機能を提供します。

## ドキュメント構成

| ドキュメント | 内容 |
|---|---|
| [Getting Started](getting-started.md) | API キーの取得から最初の API 呼び出しまで |
| [User Guide](user-guide.md) | 各機能の使い方と実践的なユースケース |
| [API Reference](reference.md) | 全エンドポイントの詳細仕様 |
| [OpenAPI Spec (Swagger UI)](spec/) | OpenAPI 3.0 仕様と対話的リファレンス |

## API の概要

Misora Connect API は次の 4 つのサービスで構成されています。

| サービス | ベースパス | 概要 |
|---|---|---|
| **SIMs** | `/v1/sims` | SIM カード情報の取得と一覧 |
| **Stats** | `/v1/stats` | 通信量の統計と CDR データ |
| **Exports** | `/v1/exports` | SIM データの CSV や JSON へのエクスポート |
| **Recharges** | `/v1/recharges` | リチャージプランの照会と予約 |

## ベース URL

```
https://api.misora-connect.com
```

## 認証

すべての API リクエストで、`x-api-key` ヘッダーによる API キー認証が必要です。

```
x-api-key: your-api-key-here
```

詳細は [Getting Started](getting-started.md) を参照してください。
