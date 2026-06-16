# Misora Connect API ドキュメント

Misora Connect API は、SIM カードの管理、通信量の統計、データエクスポート、リチャージ（容量追加）機能を提供する REST API です。

## ドキュメント構成

| ドキュメント | 内容 |
|---|---|
| [Getting Started](getting-started.md) | API キーの取得から最初の API 呼び出しまで |
| [User Guide](user-guide.md) | 各機能の使い方と実践的なユースケース |
| [API Reference](reference.md) | 全エンドポイントの詳細仕様 |

## API の概要

Misora Connect API は以下の4つのサービスで構成されています。

| サービス | ベースパス | 概要 |
|---|---|---|
| **SIMs** | `/v1/sims` | SIM カード情報の取得・一覧 |
| **Stats** | `/v1/stats` | 通信量の統計・CDR データ |
| **Exports** | `/v1/exports` | SIM データの CSV/JSON エクスポート |
| **Recharges** | `/v1/recharges` | リチャージプランの照会・予約 |

## ベース URL

```
https://api.misora-connect.com
```

## 認証

すべての API リクエストには `x-api-key` ヘッダーによる API キー認証が必要です。

```
x-api-key: your-api-key-here
```

詳細は [Getting Started](getting-started.md) を参照してください。
