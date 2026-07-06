---
title: User Guide
---

# User Guide

このガイドでは、Misora Connect API の各機能を実践的なユースケースとともに説明します。

## SIM 管理

### SIM 一覧の取得

`GET /v1/sims` で SIM の一覧を取得できます。

```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/sims"
```

#### ページネーション

大量の SIM がある場合は `limit` と `offset` でページネーションできます。

```bash
# 最初の 50 件
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/sims?limit=50&offset=0"

# 次の 50 件
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/sims?limit=50&offset=50"
```

- `limit`: 1回のレスポンスで返す最大件数（デフォルト: 100、最大: 1,000）
- `offset`: 取得開始位置（デフォルト: 0）

#### ソート

`order_by` と `order_direction` でソート順を指定できます。

```bash
# 更新日時の新しい順
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/sims?order_by=updated_at&order_direction=desc"
```

**ソート可能なフィールド**: `sim_id`、`status`、`created_at`、`updated_at`

#### 解約済み SIM の表示

デフォルトでは解約済み（terminated）の SIM は除外されます。含める場合は `include_terminated=true` を指定します。

```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/sims?include_terminated=true"
```

#### null フィールドの除外

デフォルトでは値が null のフィールドはレスポンスから除外されます。すべてのフィールドを含める場合は `filter_nulls=false` を指定します。

```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/sims?filter_nulls=false"
```

### 個別 SIM の取得

`GET /v1/sims/{sim_id}` で特定の SIM の詳細情報を取得できます。

```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/sims/8981100000000000001"
```

### SIM サマリーの取得

`GET /v1/sims/summary` でステータス別の SIM 件数を取得できます。

```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/sims/summary"
```

---

## 通信量統計

### 月別通信量

#### 全 SIM の月別通信量

`GET /v1/stats/sims/monthly_usage` で全 SIM の月別合計通信量（直近12か月分）を取得できます。

```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/stats/sims/monthly_usage"
```

#### 個別 SIM の月別通信量

```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/stats/sims/8981100000000000001/monthly_usage"
```

レスポンスには `downlink_bytes`（下り）と `uplink_bytes`（上り）がバイト単位で含まれます。

### 詳細通信量（CDR データ）

`GET /v1/stats/sims/{sim_id}/details` で 5 分間隔の詳細な CDR（Call Detail Record）データを取得できます。最大 288 レコード（約24時間分）が返ります。

```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/stats/sims/8981100000000000001/details"
```

レスポンスには接続情報（IMSI、IMEI、APN、IP アドレス）とパケット数も含まれます。

### 累積通信量

#### 個別 SIM の累積通信量

```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/stats/sims/8981100000000000001/cumulative_usage"
```

#### 全 SIM の累積通信量

```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/stats/sims/cumulative_usage"
```

**注意**: レコード数が 30,000 件を超える場合、レスポンスは `302` リダイレクトとなり、S3 の署名付き URL（gzip 圧縮 NDJSON、有効期限 5 分）へ転送されます。HTTP クライアントがリダイレクトに自動追従する設定になっているか確認してください。

---

## データエクスポート

### エクスポート可能なデータ

`GET /v1/exports` でエクスポート可能なデータ種別を確認できます。

```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/exports"
```

```json
{
  "exports": ["sims"]
}
```

### SIM データのエクスポート

`POST /v1/exports/sims` で SIM データをエクスポートできます。解約済み SIM はエクスポートに含まれません。

#### 非同期エクスポート（デフォルト）

大量データの場合に推奨される非同期モードです。リクエスト後すぐにダウンロード URL が返り、バックグラウンドでファイルが生成されます。

```bash
curl -X POST -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/exports/sims?format=csv"
```

```json
{
  "file_name": "sims_export_110139801_01KABC.csv",
  "download_url": "https://mc-prod-exports.s3.amazonaws.com/...",
  "message": "Export started. File will be available at the download URL. Estimated wait: 10 seconds.",
  "expires_in_seconds": 86400
}
```

**ポーリング**: レスポンスの `download_url` に対して GET リクエストを繰り返し、ファイルが利用可能になるまで待ちます。推定待ち時間は `message` に含まれます。

ファイル生成に失敗した場合、同じ URL にプレーンテキストのエラーファイルが配置されます。ダウンロードしたファイルの内容を確認してください。

#### 同期エクスポート

少量データの場合は同期モードを使用できます。

```bash
curl -X POST -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/exports/sims?format=csv&async=false"
```

```json
{
  "file_name": "sims_export_110139801_01KABC.csv",
  "row_count": 150,
  "file_size_bytes": 45678,
  "download_url": "https://mc-prod-exports.s3.amazonaws.com/..."
}
```

同期モードのダウンロード URL の有効期限は 5 分です。

#### エクスポート形式

| 形式 | `format` パラメータ | 説明 |
|---|---|---|
| CSV | `csv`（デフォルト） | カンマ区切りテキスト |
| JSON | `json` | JSON 配列 |
| JSON Lines | `jsonl` | 1行1レコードの JSON |

---

## リチャージ（容量追加）

### リチャージ可能な SIM の一覧

`GET /v1/recharges/sims` でリチャージ可能な SIM の一覧を取得できます。

```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/recharges/sims"
```

```json
{
  "sims": [
    {
      "iccid": "8981100000000000001",
      "product_code": "PROD-001",
      "plan_type": "capacity",
      "extension_end_date": "2026-07-31",
      "reservation_count": 0,
      "recharge_available": true,
      "usage_offset_bytes": 0
    }
  ],
  "total_count": 50,
  "page_size": 200,
  "has_more": false,
  "next_cursor": null
}
```

カーソルベースのページネーションに対応しています。`has_more` が `true` の場合、`next_cursor` の値を `cursor` パラメータに指定して次のページを取得します。

### リチャージプランの確認

#### 特定 SIM で利用可能なプラン

```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/recharges/sims/8981100000000000001/plans"
```

#### 全プランの一覧

```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/recharges/plans"
```

#### 特定プランの詳細

```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/recharges/plans/PLAN-001"
```

```json
{
  "plan_code": "PLAN-001",
  "plan_type": "capacity",
  "capacity_limit": 1,
  "daily_limit": null,
  "duration_days": 30
}
```

**プランタイプ**:
- `capacity`: 容量上限型（`capacity_limit` に GB 単位の上限）
- `daily`: 日次上限型（`daily_limit` に GB 単位の日次上限）

### リチャージの予約

`POST /v1/recharges/reservations` でリチャージを予約します。

```bash
curl -X POST -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"iccid": "8981100000000000001", "plan_code": "PLAN-001"}' \
  "https://api.misora-connect.com/v1/recharges/reservations"
```

```json
{
  "reservation_id": "rsv-001",
  "status": "reserved",
  "reserved_at": "2026-06-16T10:00:00Z"
}
```

### 予約一覧の確認

```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/recharges/reservations"
```

フィルタリングオプション:
- `iccid`: 特定 SIM の予約のみ
- `status`: ステータスで絞り込み（`reserved`、`executed`、`failed`）

```bash
# 特定 SIM の実行済み予約のみ
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/recharges/reservations?iccid=8981100000000000001&status=executed"
```

---

## 共通仕様

### バイト単位の通信量

通信量はすべてバイト単位で返されます。表示用に変換する例:

```python
def format_bytes(bytes_value):
    if bytes_value >= 1_073_741_824:
        return f"{bytes_value / 1_073_741_824:.2f} GB"
    elif bytes_value >= 1_048_576:
        return f"{bytes_value / 1_048_576:.2f} MB"
    elif bytes_value >= 1024:
        return f"{bytes_value / 1024:.2f} KB"
    return f"{bytes_value} B"
```

### 日時フォーマット

日時は ISO 8601 形式で返されます。月次指定は `YYYYMM` 形式です。

```
2026-06-16T10:00:00Z    # ISO 8601
202606                   # 年月（year_month フィールド）
```
