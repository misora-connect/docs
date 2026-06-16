---
title: API Reference
---

# API Reference

すべてのエンドポイントは認証が必要です。リクエストヘッダーに `x-api-key` を含めてください。

**ベース URL**: `https://api.misora-connect.com`

---

## SIMs

### GET /v1/sims

SIM の一覧を取得します。

**クエリパラメータ**

| パラメータ | 型 | 必須 | デフォルト | 説明 |
|---|---|---|---|---|
| `limit` | integer | No | 100 | 取得件数（最大 1,000） |
| `offset` | integer | No | 0 | 取得開始位置 |
| `order_by` | string | No | `sim_id` | ソートキー。`sim_id` / `status` / `created_at` / `updated_at` |
| `order_direction` | string | No | `asc` | ソート方向。`asc` / `desc` |
| `include_terminated` | boolean | No | `false` | 解約済み SIM を含めるか |
| `filter_nulls` | boolean | No | `true` | null フィールドをレスポンスから除外するか |

**レスポンス** `200 OK`

```json
[
  {
    "sim_id": "sim-001",
    "iccid": "8981100000000000001",
    "imsi": "440101234567890",
    "msisdn": "09012345678",
    "status": "active",
    "session_status": "online",
    "apn": "misora.io",
    "active_plan_name": "plan-s",
    "product_name": "IoT Plan S",
    "sim_category": "iot",
    "ip_address": "10.0.0.1",
    "ip_address_type": "static",
    "opening_date": "2025-01-15",
    "line_status_from_mc": "active",
    "created_at": "2025-01-15T00:00:00Z",
    "updated_at": "2026-06-16T00:00:00Z"
  }
]
```

**SIM オブジェクトのフィールド**

| フィールド | 型 | 説明 |
|---|---|---|
| `sim_id` | string | SIM の一意識別子 |
| `iccid` | string | IC カード識別番号 |
| `imsi` | string | 国際移動体加入者識別番号 |
| `msisdn` | string | 電話番号 |
| `status` | string | SIM ステータス。`active` / `suspended` / `terminated` / `ready` |
| `session_status` | string | セッション状態。`online` / `offline` |
| `apn` | string | Access Point Name |
| `active_plan_name` | string | 適用中のプラン名 |
| `product_name` | string | プロダクト名 |
| `sim_category` | string | SIM カテゴリ |
| `ip_address` | string | IP アドレス |
| `ip_address_type` | string | IP アドレス種別 |
| `opening_date` | string | 開通日 |
| `line_status_from_mc` | string | MC 上の回線ステータス |
| `created_at` | string | 作成日時（ISO 8601） |
| `updated_at` | string | 更新日時（ISO 8601） |

**エラーレスポンス**

| ステータス | 条件 |
|---|---|
| `400` | `limit` が 1,000 超、不正なソートパラメータ |
| `403` | API キー不正 |

---

### GET /v1/sims/{sim_id}

特定の SIM の詳細情報を取得します。

**パスパラメータ**

| パラメータ | 型 | 説明 |
|---|---|---|
| `sim_id` | string | SIM の一意識別子 |

**レスポンス** `200 OK`

SIM オブジェクト（単体）。フィールドは一覧取得と同一です。

**エラーレスポンス**

| ステータス | 条件 |
|---|---|
| `404` | 指定した `sim_id` が見つからない |

---

### GET /v1/sims/summary

SIM のステータス別サマリーを取得します。

**レスポンス** `200 OK`

```json
{
  "total": 150,
  "active": 120,
  "suspended": 25,
  "terminated": 5
}
```

---

## Stats

### GET /v1/stats/sims/monthly_usage

全 SIM の月別通信量を取得します。直近 12 か月分が返ります。

**レスポンス** `200 OK`

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

**月別通信量オブジェクトのフィールド**

| フィールド | 型 | 説明 |
|---|---|---|
| `year_month` | string | 年月（`YYYYMM` 形式） |
| `downlink_bytes` | integer | 下り通信量（バイト） |
| `uplink_bytes` | integer | 上り通信量（バイト） |
| `plan_name` | string | プラン名 |
| `last_updated_at` | string | 最終更新日時（ISO 8601） |

---

### GET /v1/stats/sims/{sim_id}/monthly_usage

特定 SIM の月別通信量を取得します。

**パスパラメータ**

| パラメータ | 型 | 説明 |
|---|---|---|
| `sim_id` | string | SIM の一意識別子 |

**レスポンス** `200 OK`

月別通信量オブジェクトの配列（フィールドは全 SIM 版と同一）。

---

### GET /v1/stats/sims/{sim_id}/details

特定 SIM の 5 分間隔の詳細通信量（CDR データ）を取得します。最大 288 レコード（約 24 時間分）。

**パスパラメータ**

| パラメータ | 型 | 説明 |
|---|---|---|
| `sim_id` | string | SIM の一意識別子 |

**レスポンス** `200 OK`

```json
[
  {
    "sim_id": "sim-001",
    "imsi": "440101234567890",
    "imei": "353456789012345",
    "APN": "misora.io",
    "ue_ip_address": "10.0.0.1",
    "is_dynamic_address": false,
    "record_begins_at": "2026-06-16T10:00:00Z",
    "msisdn": "09012345678",
    "session_deleted_at": null,
    "session_created_at": "2026-06-16T00:00:00Z",
    "downlink_bytes": 10485760,
    "downlink_packets": 8000,
    "uplink_bytes": 2621440,
    "uplink_packets": 2000
  }
]
```

**CDR オブジェクトのフィールド**

| フィールド | 型 | 説明 |
|---|---|---|
| `sim_id` | string | SIM ID |
| `imsi` | string | IMSI |
| `imei` | string | IMEI |
| `APN` | string | Access Point Name |
| `ue_ip_address` | string | 端末 IP アドレス |
| `is_dynamic_address` | boolean | 動的アドレスかどうか |
| `record_begins_at` | string | レコード開始日時（ISO 8601） |
| `msisdn` | string | 電話番号 |
| `session_deleted_at` | string\|null | セッション削除日時 |
| `session_created_at` | string | セッション作成日時 |
| `downlink_bytes` | integer | 下り通信量（バイト） |
| `downlink_packets` | integer | 下りパケット数 |
| `uplink_bytes` | integer | 上り通信量（バイト） |
| `uplink_packets` | integer | 上りパケット数 |

---

### GET /v1/stats/sims/cumulative_usage

全 SIM の累積通信量を取得します。

**レスポンス**

レコード数が 30,000 件未満の場合:

`200 OK`

```json
[
  {
    "sim_id": "sim-001",
    "msisdn": "09012345678",
    "downlink_bytes": 10737418240,
    "uplink_bytes": 2684354560,
    "total_bytes": 13421772800,
    "checked_at": "2026-06-16T00:00:00Z",
    "latest_record_time": "2026-06-16T00:00:00Z"
  }
]
```

レコード数が 30,000 件以上の場合:

`302 Found` — S3 署名付き URL へリダイレクト（gzip 圧縮 NDJSON、有効期限 5 分）

---

### GET /v1/stats/sims/{sim_id}/cumulative_usage

特定 SIM の累積通信量を取得します。

**パスパラメータ**

| パラメータ | 型 | 説明 |
|---|---|---|
| `sim_id` | string | SIM の一意識別子 |

**レスポンス** `200 OK`

```json
{
  "last_updated_at": "2026-06-16T00:00:00Z",
  "downlink_bytes": 10737418240,
  "uplink_bytes": 2684354560
}
```

---

## Exports

### GET /v1/exports

利用可能なエクスポート種別の一覧を取得します。

**レスポンス** `200 OK`

```json
{
  "exports": ["sims"]
}
```

---

### POST /v1/exports/sims

SIM データのエクスポートをリクエストします。解約済み SIM はエクスポートに含まれません。

**クエリパラメータ**

| パラメータ | 型 | 必須 | デフォルト | 説明 |
|---|---|---|---|---|
| `format` | string | No | `csv` | 出力形式。`csv` / `json` / `jsonl` |
| `async` | boolean | No | `true` | 非同期処理を使用するか |

**レスポンス（非同期: `async=true`）** `200 OK`

```json
{
  "file_name": "sims_export_110139801_01KABC.csv",
  "download_url": "https://mc-prod-exports.s3.amazonaws.com/sims_export_110139801_01KABC.csv?...",
  "message": "Export started. File will be available at the download URL. Estimated wait: 10 seconds.",
  "expires_in_seconds": 86400
}
```

| フィールド | 型 | 説明 |
|---|---|---|
| `file_name` | string | エクスポートファイル名 |
| `download_url` | string | S3 署名付きダウンロード URL（有効期限 24 時間） |
| `message` | string | ステータスメッセージ（推定待ち時間を含む） |
| `expires_in_seconds` | integer | URL の有効期限（秒） |

**レスポンス（同期: `async=false`）** `200 OK`

```json
{
  "file_name": "sims_export_110139801_01KABC.csv",
  "row_count": 150,
  "file_size_bytes": 45678,
  "download_url": "https://mc-prod-exports.s3.amazonaws.com/sims_export_110139801_01KABC.csv?..."
}
```

| フィールド | 型 | 説明 |
|---|---|---|
| `file_name` | string | エクスポートファイル名 |
| `row_count` | integer | エクスポートされたレコード数 |
| `file_size_bytes` | integer | ファイルサイズ（バイト） |
| `download_url` | string | S3 署名付きダウンロード URL（有効期限 5 分） |

---

## Recharges

### GET /v1/recharges/sims

リチャージ可能な SIM の一覧を取得します。

**クエリパラメータ**

| パラメータ | 型 | 必須 | デフォルト | 説明 |
|---|---|---|---|---|
| `page_size` | integer | No | 200 | 1ページあたりの件数 |
| `cursor` | string | No | - | ページネーションカーソル |

**レスポンス** `200 OK`

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

**リチャージ SIM オブジェクトのフィールド**

| フィールド | 型 | 説明 |
|---|---|---|
| `iccid` | string | IC カード識別番号 |
| `product_code` | string | プロダクトコード |
| `plan_type` | string | プランタイプ（`capacity` / `daily`） |
| `extension_end_date` | string | 延長期限日 |
| `reservation_count` | integer | 予約数 |
| `recharge_available` | boolean | リチャージ可能かどうか |
| `usage_offset_bytes` | integer | 使用量オフセット（バイト） |

**ページネーション**

| フィールド | 型 | 説明 |
|---|---|---|
| `total_count` | integer | 総件数 |
| `page_size` | integer | 1ページの件数 |
| `has_more` | boolean | 次のページがあるか |
| `next_cursor` | string\|null | 次ページのカーソル値 |

---

### GET /v1/recharges/sims/{iccid}/plans

特定 SIM で利用可能なリチャージプランを取得します。

**パスパラメータ**

| パラメータ | 型 | 説明 |
|---|---|---|
| `iccid` | string | IC カード識別番号 |

**レスポンス** `200 OK`

```json
{
  "plans": [
    {
      "plan_code": "PLAN-001",
      "plan_type": "capacity",
      "capacity_limit": 1,
      "daily_limit": null,
      "duration_days": 30
    }
  ],
  "total_count": 3
}
```

---

### GET /v1/recharges/plans

利用可能な全リチャージプランを取得します。

**レスポンス** `200 OK`

プラン一覧オブジェクト（フィールドは上記と同一）。

---

### GET /v1/recharges/plans/{plan_code}

特定のリチャージプランの詳細を取得します。

**パスパラメータ**

| パラメータ | 型 | 説明 |
|---|---|---|
| `plan_code` | string | プランコード |

**レスポンス** `200 OK`

```json
{
  "plan_code": "PLAN-001",
  "plan_type": "capacity",
  "capacity_limit": 1,
  "daily_limit": null,
  "duration_days": 30
}
```

**プランオブジェクトのフィールド**

| フィールド | 型 | 説明 |
|---|---|---|
| `plan_code` | string | プランコード |
| `plan_type` | string | プランタイプ。`capacity`（容量上限型）/ `daily`（日次上限型） |
| `capacity_limit` | number\|null | 容量上限（GB）。`capacity` タイプの場合に設定 |
| `daily_limit` | number\|null | 日次上限（GB）。`daily` タイプの場合に設定 |
| `duration_days` | integer | プラン有効日数 |

---

### POST /v1/recharges/reservations

リチャージの予約を作成します。

**リクエストボディ**

```json
{
  "iccid": "8981100000000000001",
  "plan_code": "PLAN-001"
}
```

| フィールド | 型 | 必須 | 説明 |
|---|---|---|---|
| `iccid` | string | Yes | リチャージ対象の ICCID |
| `plan_code` | string | Yes | 適用するプランコード |

**レスポンス** `200 OK`

```json
{
  "reservation_id": "rsv-001",
  "status": "reserved",
  "reserved_at": "2026-06-16T10:00:00Z"
}
```

**エラーレスポンス**

| ステータス | 条件 |
|---|---|
| `400` | 必須フィールドの不足、不正な ICCID やプランコード |
| `403` | アクセス権限なし |
| `502` | 下流サービスのエラー |

リチャージサービスのエラーレスポンスは以下の形式です:

```json
{
  "detail": {
    "errorCode": "FORBIDDEN",
    "message": "Access denied for the specified resource"
  }
}
```

---

### GET /v1/recharges/reservations

リチャージ予約の一覧を取得します。

**クエリパラメータ**

| パラメータ | 型 | 必須 | デフォルト | 説明 |
|---|---|---|---|---|
| `iccid` | string | No | - | 特定 SIM でフィルタ |
| `status` | string | No | - | ステータスでフィルタ。`reserved` / `executed` / `failed` |
| `page_size` | integer | No | 200 | 1ページあたりの件数 |
| `cursor` | string | No | - | ページネーションカーソル |

**レスポンス** `200 OK`

```json
{
  "reservations": [
    {
      "reservation_id": "rsv-001",
      "iccid": "8981100000000000001",
      "plan_code": "PLAN-001",
      "status": "executed",
      "reserved_at": "2026-06-16T10:00:00Z",
      "executed_at": "2026-06-16T10:05:00Z"
    }
  ],
  "total_count": 10,
  "page_size": 200,
  "has_more": false,
  "next_cursor": null
}
```

**予約オブジェクトのフィールド**

| フィールド | 型 | 説明 |
|---|---|---|
| `reservation_id` | string | 予約 ID |
| `iccid` | string | 対象の ICCID |
| `plan_code` | string | プランコード |
| `status` | string | ステータス。`reserved`（予約済み）/ `executed`（実行済み）/ `failed`（失敗） |
| `reserved_at` | string | 予約日時（ISO 8601） |
| `executed_at` | string\|null | 実行日時（ISO 8601） |

---

## 共通エラーレスポンス

すべてのエンドポイントで共通のエラー形式:

```json
{
  "detail": "Error message"
}
```

| ステータスコード | 説明 |
|---|---|
| `400 Bad Request` | パラメータ不正（未指定、フォーマットエラー、上限超過など） |
| `403 Forbidden` | 認証失敗 |
| `404 Not Found` | 指定されたリソースが存在しない |
| `429 Too Many Requests` | レート制限超過。時間を置いてリトライしてください |
| `500 Internal Server Error` | サーバー内部エラー |
| `502 Bad Gateway` | 下流サービスの通信エラー（Recharges サービスで発生） |
| `503 Service Unavailable` | サービス利用不可（例: Snowflake 未設定時の累積通信量） |
