---
title: Getting Started
---

# Getting Started

Misora Connect API を使い始めるための手順を説明します。

## 前提条件

- Misora Connect のアカウント
- API キー
- HTTP クライアント（curl、Postman、各言語の HTTP ライブラリなど）

## ステップ 1: API キーを確認する

Misora Connect API へのアクセスには API キーが必要です。
API キーの発行や確認については、Misora Connect の管理者にお問い合わせください。

## ステップ 2: 最初のリクエストを送信する

API キーを取得したら、まず SIM 一覧の取得を試します。

### curl の場合

```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/sims"
```

### Python の場合

```python
import requests

headers = {"x-api-key": "YOUR_API_KEY"}

response = requests.get(
    "https://api.misora-connect.com/v1/sims",
    headers=headers,
)
print(response.json())
```

### レスポンス例

```json
[
  {
    "sim_id": "sim-001",
    "customer_code": "110139801",
    "iccid": "8981100000000000001",
    "msisdn": "09012345678",
    "status": "active",
    "session_status": "online",
    "apn": "misora.io",
    "active_plan_name": "plan-s",
    "ip_address": "10.0.0.1"
  }
]
```

## ステップ 3: SIM のサマリーを確認する

SIM 全体のステータス別集計は次のエンドポイントで取得します。

```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/sims/summary"
```

```json
{
  "total": 150,
  "active": 120,
  "suspended": 25,
  "terminated": 5
}
```

## ステップ 4: 通信量を確認する

特定の SIM の月別通信量は次のエンドポイントで取得します。

```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/stats/sims/sim-001/monthly_usage"
```

```json
[
  {
    "year_month": "202606",
    "downlink_bytes": 1073741824,
    "uplink_bytes": 268435456,
    "plan_name": "plan-s",
    "last_updated_at": "2026-06-16T00:00:00Z"
  }
]
```

## 認証の仕組み

### API キー

API へのアクセスには、リクエストヘッダーに `x-api-key` を含める必要があります。

```
x-api-key: YOUR_API_KEY
```

API キーが不正、または未指定の場合は `403 Forbidden` が返ります。

## レート制限

API にはレート制限が設定されています。

| 項目 | 値 |
|---|---|
| スロットル | 1 リクエスト/秒（バースト: 1） |
| 日次クォータ | 1,500 リクエスト/日 |

制限を超えた場合は `429 Too Many Requests` が返ります。

## エラーハンドリング

API はエラー時に、対応する HTTP ステータスコードと JSON レスポンスを返します。

```json
{
  "detail": "Resource not found"
}
```

| ステータスコード | 説明 |
|---|---|
| `400` | リクエストパラメータの不正 |
| `403` | 認証失敗、またはアクセス権限なし |
| `404` | リソースが見つからない |
| `429` | レート制限超過 |
| `500` | サーバー内部エラー |

## 次のステップ

- [User Guide](user-guide.md) で各機能の詳しい使い方を確認する
- [API Reference](reference.md) で全エンドポイントの仕様を確認する
