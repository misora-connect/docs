---
title: API Reference
---

# API Reference

すべてのエンドポイントで認証が必要です。
リクエストヘッダーに `x-api-key` を含めてください。

ベース URL は `https://api.misora-connect.com` です。

---

## SIMs

### ListSims

`GET /v1/sims`

SIM の一覧を取得します。

**クエリパラメータ**

| パラメータ | 型 | 必須 | デフォルト | 説明 |
|---|---|---|---|---|
| `limit` | integer | No | 100 | 取得件数（最大 1,000） |
| `offset` | integer | No | 0 | 取得開始位置 |
| `order_by` | string | No | `sim_id` | ソートキー。`sim_id` / `customer_code` / `status` |
| `order_direction` | string | No | `asc` | ソート方向。`asc` / `desc` |
| `include_terminated` | boolean | No | `false` | 解約済み SIM を含めるか |
| `filter_nulls` | boolean | No | `true` | null フィールドをレスポンスから除外するか |

`sim_id` / `msisdn` の前方一致による絞り込みなど、すべてのパラメータは [OpenAPI Spec](spec/) を参照してください。

**レスポンス** `200 OK`

```json
[
  {
    "sim_id": "8981080000000012345",
    "customer_code": "110000001",
    "imsi": "440090000000001",
    "msisdn": "812000000001",
    "tel": "02000000001",
    "status": "active",
    "session_status": "online",
    "session_created_at": "2026-08-18T22:54:11Z",
    "apn": "example.invalid",
    "active_plan_name": "5GB/月 ライト/フル 制限時256kbps",
    "sim_category": "マルチSIM",
    "ip_address": "192.0.2.1",
    "ip_address_type": "動的IP",
    "opening_date": "2026-01-06",
    "puk1": "12345678",
    "user_id": "example-user",
    "password": "example-password",
    "lines_state_ss_for_mypage": "開通",
    "product_code": "EXAMPLE-001",
    "sim_stock_status": "黒"
  }
]
```

（例は主なフィールドのみ抜粋しています）

**SIM オブジェクトの主なフィールド**

| フィールド | 型 | 説明 |
|---|---|---|
| `sim_id` | string | SIM 識別子。**ICCID とは限りません**（19 桁の ICCID と、英大文字 2 文字 + 数字 13 桁の識別子が混在します） |
| `customer_code` | string | 顧客コード（9 桁） |
| `imsi` | string | IMSI |
| `msisdn` | string | 電話番号（国際表記） |
| `tel` | string | 電話番号（国内表記） |
| `status` | string | 回線状態。`stock` / `ready` / `active` / `inactive` / `terminated` / `unknown` |
| `session_status` | string | セッション接続状態。`online` / `offline` |
| `session_created_at` | string | 最新セッションの開始時刻（UTC） |
| `session_terminated_at` | string | 最新セッションの終了時刻（UTC）。継続中は `null` |
| `apn` | string | APN |
| `active_plan_name` | string | 現在適用中のプラン名 |
| `product_name` | string | 商品名 |
| `sim_category` | string | SIM 種別 |
| `ip_address` | string | IP アドレス |
| `ip_address_type` | string | IP アドレス種別 |
| `opening_date` | string | 開通日 |
| `puk1` | string | PUK1（PIN ロック解除コード）。**取り扱いにご注意ください** |
| `user_id` | string | APN 認証のユーザ ID。`apn` / `password` と組で使います。**取り扱いにご注意ください** |
| `password` | string | APN 認証のパスワード。`apn` と対で使います。**取り扱いにご注意ください** |
| `lines_state_ss_for_mypage` | string | 回線状態（マイページの表示値）。画面表示にはこちらを使ってください |
| `product_code` | string | 商品コード |
| `sim_stock_status` | string | SIM 状態（`白` / `半黒` / `黒`）。回線状態の `status` とは別物です |

すべてのフィールド（45 項目）と各フィールドの詳細は [OpenAPI Spec](spec/) の `Sim` スキーマを参照してください。

`puk1` / `password` は 2026 年 9 月、`user_id` は 2026 年 10 月の更新で返すようになりました。

**エラーレスポンス**

| ステータス | 条件 |
|---|---|
| `400` | `limit` が 1,000 超、不正なソートパラメータ |
| `403` | API キー不正 |

---

### GetSim

`GET /v1/sims/{sim_id}`

特定の SIM の詳細情報を取得します。

**パスパラメータ**

| パラメータ | 型 | 説明 |
|---|---|---|
| `sim_id` | string | SIM の一意識別子 |

**レスポンス** `200 OK`

SIM オブジェクト（単体）。一覧取得のフィールドに加えて、次のフィールドを返します。

| フィールド | 型 | 説明 |
|---|---|---|
| `note` | string | 備考。備考が無い SIM では `filter_nulls=true`（既定）ならキーごと省略、`filter_nulls=false` なら `null` |

`note` は SIM 明細でのみ返り、一覧取得と CSV エクスポートには含まれません。

**エラーレスポンス**

| ステータス | 条件 |
|---|---|
| `404` | 指定した `sim_id` が見つからない |

---

### GetSimsSummary

`GET /v1/sims/summary`

SIM のステータス別サマリーを取得します。

**レスポンス** `200 OK`

```json
{
  "customer_code": "110000001",
  "total": 439,
  "subscribed": 426,
  "ready": 330,
  "active": 2,
  "online": 16,
  "terminated": 13,
  "unknown": 94,
  "updated_at": "2026-08-12T15:04:31Z"
}
```

| フィールド | 型 | 説明 |
|---|---|---|
| `customer_code` | string | 顧客コード |
| `total` | integer | 全件 |
| `subscribed` | integer | 解約以外（在庫・停止を含む） |
| `ready` | integer | 開通済みで出荷済み |
| `active` | integer | 稼働中 |
| `online` | integer | セッション接続中 |
| `terminated` | integer | 解約済み |
| `unknown` | integer | 上記のいずれにも該当しない |
| `updated_at` | string\|null | 集計元データの最終更新時刻 |

---

## Stats

### ListMonthlyUsage

`GET /v1/stats/sims/monthly_usage`

全 SIM の月別通信量を取得します。
直近 12 か月分が返ります。

**レスポンス** `200 OK`

```json
[
  {
    "year_month": "202606",
    "downlink_bytes": 1073741824,
    "uplink_bytes": 268435456
  }
]
```

**集計月別通信量オブジェクトのフィールド**

| フィールド | 型 | 説明 |
|---|---|---|
| `year_month` | string | 年月（`YYYYMM` 形式） |
| `downlink_bytes` | integer | 下り通信量の合計（バイト、SUM 集計） |
| `uplink_bytes` | integer | 上り通信量の合計（バイト、SUM 集計） |

---

### GetSimMonthlyUsage

`GET /v1/stats/sims/{sim_id}/monthly_usage`

特定 SIM の月別通信量を取得します。

**パスパラメータ**

| パラメータ | 型 | 説明 |
|---|---|---|
| `sim_id` | string | SIM の一意識別子 |

**レスポンス** `200 OK`

```json
[
  {
    "sim_id": "sim-001",
    "customer_code": "110139801",
    "operator_id": "op-001",
    "imsi": "440101234567890",
    "msisdn": "09012345678",
    "year_month": "202606",
    "plan_name": "plan-s",
    "downlink_bytes": 1073741824,
    "uplink_bytes": 268435456,
    "last_updated_at": "2026-06-16T00:00:00Z"
  }
]
```

**SIM 別月別通信量オブジェクトのフィールド**

| フィールド | 型 | 説明 |
|---|---|---|
| `sim_id` | string | SIM の一意識別子 |
| `customer_code` | string | 顧客コード |
| `operator_id` | string | オペレーター識別子 |
| `imsi` | string | IMSI |
| `msisdn` | string | MSISDN |
| `year_month` | string | 年月（`YYYYMM` 形式） |
| `plan_name` | string | プラン名 |
| `downlink_bytes` | integer | 下り通信量（バイト） |
| `uplink_bytes` | integer | 上り通信量（バイト） |
| `last_updated_at` | string | 最終更新日時（ISO 8601） |

---

### GetSimUsageDetails

`GET /v1/stats/sims/{sim_id}/details`

特定 SIM の 5 分間隔の詳細通信量（CDR データ）を取得します。
最大 288 レコード（約 24 時間分）が返ります。

**パスパラメータ**

| パラメータ | 型 | 説明 |
|---|---|---|
| `sim_id` | string | SIM の一意識別子 |

**レスポンス** `200 OK`

```json
[
  {
    "sim_id": "sim-001",
    "customer_code": "110139801",
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
| `customer_code` | string | カスタマーコード |
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

### ListCumulativeUsage

`GET /v1/stats/sims/cumulative_usage`

全 SIM の累積通信量を取得します。

**レスポンス**

レコード数が 30,000 件未満の場合:

`200 OK`

```json
[
  {
    "customer_code": "110139801",
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

`302 Found` を返し、S3 の署名付きダウンロード URL（NDJSON、`Content-Type: application/x-ndjson`、有効期限 5 分）へリダイレクトします。実体は gzip 圧縮されており、S3 側で `Content-Encoding: gzip` を付与するため、`curl --compressed` / requests / ブラウザなど標準的な HTTP クライアントは自動的に解凍します（自前で gunzip する必要はありません）。

---

### GetSimCumulativeUsage

`GET /v1/stats/sims/{sim_id}/cumulative_usage`

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

### ListCurrentPlanUsage

`GET /v1/stats/sims/current_plan_usage`

全 SIM の「現プランでの消費量」を取得します。

累積通信量（`total_bytes`）が SIM の生涯通算であるのに対し、`current_plan_used_bytes` は最新の実行済みリチャージ以降に使った量です。「今のプランであとどれだけ使えるか」を知りたい場合はこちらを参照してください。

**レスポンス** `200 OK`

```json
[
  {
    "sim_id": "8981100000000000001",
    "customer_code": "110139801",
    "msisdn": "09012345678",
    "downlink_bytes": 10737418240,
    "uplink_bytes": 2684354560,
    "total_bytes": 13421772800,
    "checked_at": "2026-06-16T00:00:00Z",
    "latest_record_time": "2026-06-16T00:00:00Z",
    "plan_code": "OI071522",
    "plan_started_at": "2026-06-10T01:00:00Z",
    "usage_offset_bytes": 10737418240,
    "current_plan_used_bytes": 2684354560
  }
]
```

---

### GetSimCurrentPlanUsage

`GET /v1/stats/sims/{sim_id}/current_plan_usage`

特定 SIM の「現プランでの消費量」を取得します。レスポンスは要素 1 件の配列です。

**パスパラメータ**

| パラメータ | 型 | 説明 |
|---|---|---|
| `sim_id` | string | SIM の一意識別子 |

**レスポンス** `200 OK`

```json
[
  {
    "sim_id": "8981100000000000001",
    "customer_code": "110139801",
    "msisdn": "09012345678",
    "downlink_bytes": 10737418240,
    "uplink_bytes": 2684354560,
    "total_bytes": 13421772800,
    "checked_at": "2026-06-16T00:00:00Z",
    "latest_record_time": "2026-06-16T00:00:00Z",
    "plan_code": "OI071522",
    "plan_started_at": "2026-06-10T01:00:00Z",
    "usage_offset_bytes": 10737418240,
    "current_plan_used_bytes": 2684354560
  }
]
```

**レスポンスフィールド**

| フィールド | 型 | 説明 |
|---|---|---|
| `sim_id` | string | SIM の一意識別子（ICCID） |
| `customer_code` | string | 顧客コード |
| `msisdn` | string | 電話番号 |
| `downlink_bytes` | integer | 累積下り通信量（bytes） |
| `uplink_bytes` | integer | 累積上り通信量（bytes） |
| `total_bytes` | integer | 累積通信量（bytes、生涯通算） |
| `checked_at` | string | 集計時刻 |
| `latest_record_time` | string | 集計に含まれる最新レコードの時刻 |
| `plan_code` | string \| null | 現プランのプランコード（最新の実行済みリチャージ予約のもの） |
| `plan_started_at` | string \| null | 現プランの開始時刻（最新の実行済みリチャージ予約の実行日時） |
| `usage_offset_bytes` | integer \| null | 現プラン適用時点の累積通信量 |
| `current_plan_used_bytes` | integer \| null | 現プランでの消費量（bytes、= `total_bytes − usage_offset_bytes`） |

`usage_offset_bytes` が記録されていない SIM では、`plan_code` / `plan_started_at` / `usage_offset_bytes` / `current_plan_used_bytes` が `null` になります。一度もリチャージを実行していない SIM が該当します。

**主要エラー**

| ステータス | 条件 |
|---|---|
| 400 | `sim_id` の形式が不正 |
| 404 | 指定 SIM の利用実績データが存在しない |
| 503 | 集計基盤が一時的に利用できない |

---

## Exports

### ListExports

`GET /v1/exports`

利用可能なエクスポート種別の一覧を取得します。

**レスポンス** `200 OK`

```json
{
  "exports": ["sims"]
}
```

---

### CreateSimsExport

`POST /v1/exports/sims`

SIM データのエクスポートをリクエストします。
解約済み SIM はエクスポートに含まれません。

**CSV の列構成**

CSV は次の **42 列** を、この順で出力します（列名は SIM オブジェクトのフィールド名）。

`sim_id`, `customer_code`, `application_start_date`, `cancellation_mobile_core_date`,
`carrier_opening_date`, `carrier_withdrawal_date`, `communication_generation`,
`dns_primary`, `dns_secondary`, `extension_end_date`, `first_communicating_date`,
`imsi`, `ip_address`, `ip_address_type`, `line_category`, `line_status_from_mc`,
`line_stoped_date`, `lines_status`, `linestarted_date`, `opening_date`,
`puk1`, `user_id`, `password`, `msisdn`, `realm`,
`service_category`, `use_start_expiration_date`, `shipping_date`, `apn`,
`active_plan_name`, `product_name`, `sim_category`, `product_name_id`,
`status`, `session_status`, `session_created_at`, `session_terminated_at`,
`lines_state_ss_for_mypage`, `product_code`, `quotation_application_number`, `tel`,
`sim_stock_status`

`ipv6_address` / `ipv6_address_dns_primary` / `ipv6_address_dns_secondary` の 3 列は CSV には出力しません（`GET /v1/sims` の応答には含まれます）。

> **2026 年 9 月の更新で列構成が変わりました**（37 列 → 41 列）。
>
> - ipv6 系 3 列を削除（旧 CSV の 15〜17 列目）
> - `puk1` / `password` を `opening_date` の直後に追加
> - `lines_state_ss_for_mypage` 以降の 5 列を末尾に追加
>
> 続く 2026 年 10 月の更新で、`user_id`（APN 認証のユーザ ID）を `password` の直前に追加しました（41 列 → 42 列）。`password` と組で使う値のため、例外的に途中に置いています。**`password` 以降の列は位置が 1 つ後ろにずれます。**
>
> 列を位置で読んでいる場合は追随してください。列名で読んでいる場合は、削除した 3 列を参照していなければ影響ありません。今後列を追加する場合は原則として末尾に追加します。
>
> CSV には `puk1`（PIN ロック解除コード）、`user_id` と `password`（APN 認証のユーザ ID とパスワード）が含まれます。ダウンロードしたファイルの取り扱いにご注意ください。

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

### ListRechargeableSims

`GET /v1/recharges/sims`

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
      "sim_id": "8981100000000000001",
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
| `sim_id` | string | SIM の一意識別子（ICCID） |
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

### GetSimBalance

`GET /v1/recharges/sims/{sim_id}/balance`

特定 SIM の使える総量・残り容量・使用量を取得します。マイページ等で「◯GB 中 ◯GB 残り」を描画する用途を想定しています。

**繰越（Banked Data）の扱い**

- 使える総量 = 現プラン容量 + 繰越（未消化分 = Banked Data）
- 消費順序は 現プラン → 繰越。現プランの枯渇または期間満了時に、繰越分は自動的に払い出されます（お客様の操作は不要で、通信は途切れません）。したがって残量は現プランと繰越をまたいで連続して減ります
- 繰越は無期限ではなく、払い出し用の内部プランの期間終了時に消滅します

**daily プラン（1GB/day 等）の当日枠**

- 通算残量の概念を持たないため容量 3 値は `null` のまま、当日枠を `daily_limit_bytes` / `daily_used_bytes` / `daily_remaining_bytes` で返します
- 当日利用量は**日本時間の 0:00 以降**の実績の合計です。日付が変わると 0 に戻ります
- 日次上限のないプラン（無制限）や、リセット周期が 1 日でないプランでは上限・残量が `null` になり、**当日利用量のみ**を返します

レスポンスは全プランタイプで同一のスキーマです。capacity プランでは `daily_*` が、daily プランでは容量 3 値が、それぞれ `null` として含まれます。

**パスパラメータ**

| パラメータ | 型 | 説明 |
|---|---|---|
| `sim_id` | string | SIM の一意識別子（ICCID、19-20 桁の数字） |

**レスポンス** `200 OK`

capacity プラン（容量型）:

```json
{
  "sim_id": "8981080321000912770",
  "plan_type": "capacity",
  "total_granted_bytes": 6442450944,
  "total_remaining_bytes": 5368709120,
  "used_bytes": 1073741824,
  "banked_data_bytes": 1073741824,
  "filler_state": "NORMAL",
  "checked_at": "2026-08-26T05:30:00Z",
  "latest_record_time": "2026-08-25T15:00:00Z",
  "daily_limit_bytes": null,
  "daily_used_bytes": null,
  "daily_remaining_bytes": null
}
```

daily プラン（日次型・1GB/day）:

```json
{
  "sim_id": "8981080321000912788",
  "plan_type": "daily",
  "total_granted_bytes": null,
  "total_remaining_bytes": null,
  "used_bytes": null,
  "banked_data_bytes": null,
  "filler_state": "NORMAL",
  "checked_at": "2026-09-03T05:30:00Z",
  "latest_record_time": "2026-09-03T05:18:00Z",
  "daily_limit_bytes": 1073741824,
  "daily_used_bytes": 268435456,
  "daily_remaining_bytes": 805306368
}
```

**レスポンスフィールド**

| フィールド | 型 | 説明 |
|---|---|---|
| `sim_id` | string | SIM の一意識別子（ICCID） |
| `plan_type` | string \| null | プランタイプ（`capacity` / `daily`）。プランマスタ未登録時は `null` |
| `total_granted_bytes` | integer \| null | 付与容量（bytes）＝「◯GB 中 ◯GB 残り」の**前者**。**現プラン容量 + 繰越（Banked Data）の合計**。capacity プランのみ、それ以外は `null` |
| `total_remaining_bytes` | integer \| null | 利用可能残量（bytes）＝「◯GB 中 ◯GB 残り」の**後者**。**繰越分を含みます**。サーバ側で `0 ≤ remaining ≤ total_granted_bytes` および `used = granted − remaining` の整合を保証するため、**利用者側での追加補正は不要**です。capacity プランのみ |
| `used_bytes` | integer \| null | 使用量（bytes、= `total_granted_bytes − total_remaining_bytes`）。capacity プランのみ |
| `banked_data_bytes` | integer \| null | 繰越（未消化データ）残高（bytes）。**`total_granted_bytes` / `total_remaining_bytes` に既に含まれる内訳であり、加算しないでください**。繰越の払い出し中（`filler_state != NORMAL`）は 0 になりますが、総枠・残量は繰越分を含んだままなので表示は不連続になりません。capacity プランのみ |
| `filler_state` | string | Filler 状態（`NORMAL` / `FILLER_LARGE` / `FILLER_SMALL`） |
| `checked_at` | string | 残量を確認した時刻（UTC、RFC 3339 / `...Z` 形式） |
| `latest_record_time` | string \| null | 残量算出に使った使用量データの最新記録時刻（UTC、RFC 3339 / `...Z` 形式）。通信実績が無い SIM や、使用量を参照しない状態（総枠未確定など）では `null`。daily プランでは当日分の集計に使った最新実績の時刻を返します |
| `daily_limit_bytes` | integer \| null | 1 日あたりの上限（bytes）。**daily プランのみ**。日次上限のないプラン（無制限）、リセット周期が 1 日でないプラン、およびプラン情報から上限を判定できない場合は `null` |
| `daily_used_bytes` | integer \| null | 当日利用量（bytes）。**日本時間 0:00 以降**の実績の合計で、日付が変わると 0 に戻ります。**daily プランのみ**。通信実績が無い SIM は `0` |
| `daily_remaining_bytes` | integer \| null | 当日残量（bytes）＝ `daily_limit_bytes − daily_used_bytes`（0 未満にはなりません）。**daily プランのみ**。`daily_limit_bytes` が `null` の場合は `null` |

**主要エラー**

| ステータス | 条件 |
|---|---|
| 400 | `sim_id` 未指定、ICCID 形式不正、解約済み SIM（`message: SIM is terminated`） |
| 403 | 指定 SIM を当該顧客が所有していない |
| 404 | SIM が見つからない |
| 503 | 累積 usage 取得に失敗（古い値・キャッシュは返さない） |

解約済み SIM の残量は返しません。`GET /v1/sims` で `include_terminated=true` を指定して取得した SIM や、解約直後で一覧にまだ残っている SIM に対して呼ぶと 400 になります（解約済み SIM は `GET /v1/recharges/sims` の一覧には含まれません）。

---

### ListSimRechargePlans

`GET /v1/recharges/sims/{sim_id}/plans`

特定 SIM で利用可能なリチャージプランを取得します。

**パスパラメータ**

| パラメータ | 型 | 説明 |
|---|---|---|
| `sim_id` | string | SIM の一意識別子（ICCID） |

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

### ListRechargePlans

`GET /v1/recharges/plans`

利用可能な全リチャージプランを取得します。

**レスポンス** `200 OK`

プラン一覧オブジェクト（フィールドは上記と同一）。

---

### GetRechargePlan

`GET /v1/recharges/plans/{plan_code}`

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

### CreateRechargeReservation

`POST /v1/recharges/reservations`

リチャージの予約を作成します。

**リクエストボディ**

```json
{
  "sim_id": "8981100000000000001",
  "plan_code": "PLAN-001"
}
```

| フィールド | 型 | 必須 | 説明 |
|---|---|---|---|
| `sim_id` | string | Yes | リチャージ対象の SIM ID（ICCID） |
| `plan_code` | string | Yes | 適用するプランコード |

**レスポンス** `200 OK`

```json
{
  "reservation_id": "RR-00012",
  "status": "Reserved",
  "reserved_at": "2026-06-16T10:00:00Z"
}
```

`executed_at` は予約作成の応答には含まれません。実行日時は `GET /v1/recharges/reservations` で取得してください。

#### 予約できる件数

1 つの SIM に対して予約できるのは **最大 3 件** です。上限にカウントされるのは `status` が
`Reserved`（未実行）の予約のみで、`Executed`（実行済み）や取り消し済みの予約は含みません。

そのため「通算 4 件目」ではなく「未実行の予約が同時に 4 件になる」タイミングで
`LIMIT_EXCEEDED` になります。3 件予約済みでも、そのうち 1 件が実行されれば次の予約を作成できます。

**エラーレスポンス**

| ステータス | `errorCode` | 条件 |
|---|---|---|
| `400` | `BAD_REQUEST` | 不正な SIM ID やプランコード、解約済み SIM、リチャージ対象外プラン、受付期限の超過 |
| `400` | `LIMIT_EXCEEDED` | 未実行の予約が上限（3 件）に到達している |
| `400` | `PLAN_MISMATCH` | 容量上限型と日次上限型の混在、または日次上限型で 1 日あたり容量・リセット周期が現行プランと不一致 |
| `400` | `REALM_MISMATCH` | プランのレルムが SIM の現行レルムと不一致 |
| `403` | `FORBIDDEN` | アクセス権限なし |
| `404` | `NOT_FOUND` | 指定した SIM が登録されていない |
| `400` | `VALIDATION_ERROR` | `plan_code` の不足・型不正などリクエストボディの不正 |
| `502` | - | 下流サービスのエラー |

`sim_id` が未指定の場合と、`sim_id` と `iccid` を異なる値で両方指定した場合は、`errorCode` を持たない
400（`detail` が文字列の形式 A。後述の「共通エラーレスポンス」を参照）になります。

リチャージサービスのエラーレスポンスは次の形式です。

```json
{
  "detail": {
    "errorCode": "FORBIDDEN",
    "message": "Customer does not have permission for this ICCID"
  }
}
```

`errorCode` が `BAD_REQUEST` になる条件は複数あるため、条件の切り分けには `message` を参照してください。
主なものは次のとおりです。

| `message` | 条件 |
|---|---|
| `Invalid ICCID format` | SIM ID が 19-20 桁の数字でない |
| `Invalid planCode` | 存在しない、または有効期限切れのプランコード |
| `SIM is terminated` | 解約済みの SIM を指定した |
| `CPFR plans are not eligible for recharge` | リチャージ対象外のプラン（定額・容量無制限）を指定した |
| `Reservation is not accepted after 19:00 on the last day` | 利用終了日当日の 19:00（JST）以降に予約しようとした（後述の「予約の受付期限」を参照） |

#### 予約の受付期限

**利用終了日の当日に限り、19:00（JST）以降の予約は受け付けません。** 当日の 19:00 以降に
予約を作成しようとすると `400` / `BAD_REQUEST` /
`Reservation is not accepted after 19:00 on the last day` を返します。

利用終了日より前であれば、時刻による制限はありません。当日分の予約が必要な場合は、
19:00（JST）までに作成してください。

この締切は、利用終了日当日に行われるリチャージの実行処理（後述の 20:00 / 21:00）に
予約を確実に間に合わせるためのものです。

#### リチャージの実行タイミング

予約したリチャージが実際に適用されるタイミングは、プランタイプによって異なります。

| プランタイプ | 実行条件 |
|---|---|
| `capacity`（容量上限型） | 残データ量が 500MB 未満になった時点、または利用終了日の 21:00（JST）以降 |
| `daily`（日次上限型） | 利用終了日に到達後、毎日 20:00（JST）の日次処理で実行（残データ量による判定なし） |

※ 上限が 500MB 以下の小容量プランでは、上限に到達した時点で実行されます。

予約直後は `status` が `Reserved` となり、実行後に `Executed` へ変わります。

---

### ListRechargeReservations

`GET /v1/recharges/reservations`

リチャージ予約の一覧を取得します。

**クエリパラメータ**

| パラメータ | 型 | 必須 | デフォルト | 説明 |
|---|---|---|---|---|
| `sim_id` | string | No | - | 特定 SIM でフィルタ |
| `status` | string | No | - | ステータスでフィルタ。`Reserved` / `Executed` / `Failed`。大文字始まりのみ有効で、値が一致しない場合は `400`。取り消し済み（`Cancelled`）の予約は一覧に含まれないため、フィルタ値としても指定できません |
| `page_size` | integer | No | 200 | 1ページあたりの件数 |
| `cursor` | string | No | - | ページネーションカーソル |

**レスポンス** `200 OK`

```json
{
  "reservations": [
    {
      "reservation_id": "rsv-001",
      "sim_id": "8981100000000000001",
      "plan_code": "PLAN-001",
      "status": "Executed",
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
| `sim_id` | string | SIM の一意識別子（ICCID） |
| `plan_code` | string | プランコード |
| `status` | string | ステータス。`Reserved`（予約済み）/ `Executed`（実行済み）/ `Failed`（失敗）。取り消し済みの予約は一覧に含まれません |
| `reserved_at` | string | 予約日時（ISO 8601） |
| `executed_at` | string\|null | 実行日時（ISO 8601） |

---

### CreateImmediateRecharge

`POST /v1/recharges/immediate`

プランの書き換えと残量リセットを、予約を挟まずその場で実行します。予約レコードは作成と同時に `Executed` になります。

容量上限型（`capacity`）のプランのみが対象です。日次上限型プラン、CPFR プラン、社内専用プランを指定した場合はエラーになります。

**リクエストボディ**

| フィールド | 型 | 必須 | 説明 |
|---|---|---|---|
| `sim_id` | string | ○ | 対象 SIM の一意識別子（ICCID、19-20 桁の数字） |
| `plan_code` | string | ○ | 書き換え先のプランコード（容量上限型のみ） |
| `force` | boolean | | `true` のとき、既存の予約済みリチャージを取り消して前倒し実行します。既定値は `false` |

```json
{
  "sim_id": "8981080301011327143",
  "plan_code": "OI071522",
  "force": false
}
```

**レスポンス** `200 OK`

```json
{
  "sim_id": "8981080301011327143",
  "plan_code": "OI071522",
  "status": "Executed",
  "reservation_id": "a1XRB000005P7Q12AK",
  "sync_status": "実行中",
  "superseded_reservation_ids": [],
  "message": "Immediate recharge executed. PCRF sync triggered."
}
```

**レスポンスフィールド**

| フィールド | 型 | 説明 |
|---|---|---|
| `sim_id` | string | SIM の一意識別子（ICCID） |
| `plan_code` | string | 書き換え先のプランコード |
| `status` | string \| null | 予約ステータス。成功時は `Executed` |
| `reservation_id` | string \| null | 生成された予約 ID |
| `sync_status` | string \| null | ネットワーク側への反映状況。成功直後は `実行中` で、反映完了後に `実行済` へ変わります |
| `superseded_reservation_ids` | array | `force` により取り消した既存予約の ID |
| `message` | string | 処理結果メッセージ |

**主要エラー**

エラー時のボディは成功時と形が異なり、`detail` の中に `errorCode` と `message` が入ります
（後述の「共通エラーレスポンス」形式 B）。成功時のレスポンスに `error_code` フィールドは含まれません。

```json
{
  "detail": {
    "errorCode": "INVALID_PLAN",
    "message": "Plan is not eligible for immediate recharge"
  }
}
```

| ステータス | `detail.errorCode` | 条件 |
|---|---|---|
| 400 | `INVALID_REQUEST` | 必須項目が不足している |
| 400 | `INVALID_ICCID` | `sim_id` が未指定、または ICCID の形式が不正 |
| 400 | `INVALID_PLAN` | プランコードが不正、解約済み SIM、日次上限型 / CPFR / 社内専用プランを指定した |
| 400 | `REALM_MISMATCH` | SIM とプランの接続先ネットワークが一致しない |
| 403 | `FORBIDDEN_TENANT` | 指定 SIM を当該顧客が所有していない |
| 404 | `NOT_FOUND` | SIM が見つからない |
| 409 | `ALREADY_SYNCING` | ネットワーク側へ反映中。`force` を指定しても実行できません |
| 409 | `RESERVATION_EXISTS` | 実行待ちの予約が存在する。`force=true` で前倒し実行できます |
| 500 | `EXECUTION_FAILED` | 実行処理に失敗した |
| 503 | `USAGE_UNAVAILABLE` | 通信量の取得に失敗した（プランは書き換えていません） |

`ALREADY_SYNCING` は `force` の指定によらず発生します。直前の書き換えがネットワークへ反映されるまで待ってから再試行してください。

---

## 共通エラーレスポンス

エラー応答のボディは、エンドポイントとエラーの発生箇所によって次の 2 形式のいずれかになります。
クライアント側では両方を扱えるようにしてください。

**形式 A — `detail` が文字列**

SIMs / Stats / Exports の各エンドポイントと、Recharges のパラメータ検証エラーで返します。

```json
{
  "detail": "SIM not found"
}
```

**形式 B — `detail` がオブジェクト**

Recharges のうちリチャージサービスへ委譲するエンドポイント（残量取得、プラン一覧、
予約の作成・一覧、即時リチャージ）で返します。`errorCode` / `message` はキャメルケースです。

```json
{
  "detail": {
    "errorCode": "BAD_REQUEST",
    "message": "SIM is terminated"
  }
}
```

リクエストボディの形式が不正な場合（必須キーの欠落・型不正）のみ、`message` の代わりに詳細の配列
`errors` が入ります（`errorCode` は `VALIDATION_ERROR`）。値が空文字の場合など、同じ `VALIDATION_ERROR`
でも `message` で返る場合があります。

`errorCode` は各エンドポイントの表に記載したもの以外に、下流サービスの障害時には `SF_AUTH_ERROR` /
`SF_API_ERROR` / `INTERNAL_ERROR` / `UNKNOWN_ERROR` なども返ります。未知の値も扱えるようにしてください。

なお、API キーが不正または未指定の場合は API Gateway が応答するため、上記いずれとも異なる
`{"message": "Forbidden"}` が返ります。

| ステータスコード | 説明 |
|---|---|
| `400 Bad Request` | パラメータ不正（未指定、フォーマットエラー、上限超過など） |
| `403 Forbidden` | 認証失敗（API キー不正）、または指定したリソースへのアクセス権限がない（他社所有の SIM を指定した場合など） |
| `404 Not Found` | 指定されたリソースが存在しない |
| `429 Too Many Requests` | レート制限を超過。時間を置いてから再試行してください |
| `500 Internal Server Error` | サーバー内部エラー |
| `502 Bad Gateway` | 下流サービスの通信エラー（Recharges サービスで発生） |
| `503 Service Unavailable` | サービス利用不可（例: Snowflake 未設定時の累積通信量） |
