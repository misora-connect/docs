---
title: User Guide
---

# User Guide

Misora Connect API の各機能について、実践的なユースケースを交えて説明します。

## SIM 管理

### SIM 一覧の取得

`GET /v1/sims` で SIM の一覧を取得します。

```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/sims"
```

#### ページネーション

大量の SIM がある場合は、`limit` と `offset` でページネーションを行います。

```bash
# 最初の 50 件
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/sims?limit=50&offset=0"

# 次の 50 件
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/sims?limit=50&offset=50"
```

- `limit`：1 回のレスポンスで返す最大件数（デフォルト: 100、最大: 1,000）
- `offset`：取得開始位置（デフォルト: 0）

#### ソート

`order_by` と `order_direction` でソート順を指定します。

```bash
# 更新日時の新しい順
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/sims?order_by=updated_at&order_direction=desc"
```

ソート可能なフィールドは `sim_id`、`status`、`created_at`、`updated_at` です。

#### 解約済み SIM の表示

デフォルトでは、解約済み（`terminated`）の SIM は除外されます。
含める場合は `include_terminated=true` を指定します。

```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/sims?include_terminated=true"
```

#### null フィールドの除外

デフォルトでは、値が `null` のフィールドはレスポンスから除外されます。
すべてのフィールドを含める場合は `filter_nulls=false` を指定します。

```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/sims?filter_nulls=false"
```

### 個別 SIM の取得

`GET /v1/sims/{sim_id}` で特定の SIM の詳細情報を取得します。

```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/sims/8981100000000000001"
```

### SIM サマリーの取得

`GET /v1/sims/summary` でステータス別の SIM 件数を取得します。

```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/sims/summary"
```

## 通信量統計

### 月別通信量

#### 全 SIM の月別通信量

`GET /v1/stats/sims/monthly_usage` で、全 SIM の月別合計通信量（直近 12 か月分）を取得します。

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

`GET /v1/stats/sims/{sim_id}/details` で 5 分間隔の詳細な CDR（Call Detail Record）データを取得します。
最大 288 レコード（約 24 時間分）が返ります。

```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/stats/sims/8981100000000000001/details"
```

レスポンスには、接続情報（IMSI、IMEI、APN、IP アドレス）とパケット数も含まれます。

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

レコード数が 30,000 件を超える場合、レスポンスは `302` リダイレクトとなり、S3 の署名付き URL（gzip 圧縮 NDJSON、有効期限 5 分）へ転送されます。
HTTP クライアントがリダイレクトに自動追従する設定になっているか、あらかじめ確認してください。

### 現プランでの消費量

累積通信量が SIM の生涯通算であるのに対し、`current_plan_usage` は「最新の実行済みリチャージ以降に使った量」を返します。
「今のプランであとどれだけ使えるか」を知りたい場合はこちらを使います。

#### 個別 SIM の現プラン消費量

```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/stats/sims/8981100000000000001/current_plan_usage"
```

```json
[
  {
    "sim_id": "8981100000000000001",
    "total_bytes": 13421772800,
    "plan_code": "OI071522",
    "plan_started_at": "2026-06-10T01:00:00Z",
    "usage_offset_bytes": 10737418240,
    "current_plan_used_bytes": 2684354560
  }
]
```

`current_plan_used_bytes` は `total_bytes − usage_offset_bytes` で求まります。
`usage_offset_bytes` は現プランが適用された時点の累積通信量です。

#### 全 SIM の現プラン消費量

```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/stats/sims/current_plan_usage"
```

一度もリチャージを実行していない SIM では、`plan_code` や `current_plan_used_bytes` が `null` になります。
残量をそのまま画面に出す用途では、リチャージ側の残量取得 API（後述の「残量の確認」）のほうが扱いやすい場合があります。

## データエクスポート

### エクスポート可能なデータ

`GET /v1/exports` で、エクスポート可能なデータ種別を確認できます。

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

`POST /v1/exports/sims` で SIM データをエクスポートします。
解約済み SIM はエクスポートに含まれません。

#### 非同期エクスポート（デフォルト）

大量データを扱う場合に推奨されるモードです。
リクエスト後すぐにダウンロード URL が返り、ファイルはバックグラウンドで生成されます。

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

レスポンスの `download_url` に対して GET リクエストを繰り返し、ファイルが利用可能になるまで待ちます（ポーリング）。
推定待ち時間は `message` に含まれます。

ファイル生成に失敗した場合、同じ URL にプレーンテキストのエラーファイルが配置されます。
ダウンロードしたファイルの内容を確認してください。

#### 同期エクスポート

少量のデータであれば、同期モードを利用できます。

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
| JSON Lines | `jsonl` | 1 行 1 レコードの JSON |

## リチャージ（容量追加）

### リチャージ可能な SIM の一覧

`GET /v1/recharges/sims` で、リチャージ可能な SIM の一覧を取得します。

```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/recharges/sims"
```

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

カーソルベースのページネーションに対応しています。
`has_more` が `true` の場合、`next_cursor` の値を `cursor` パラメータに指定して次のページを取得します。

### 残量の確認

`GET /v1/recharges/sims/{sim_id}/balance` で、使える総量・残量・使用量をまとめて取得します。
マイページ等で「◯GB 中 ◯GB 残り」を描画する用途を想定した API です。

```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/recharges/sims/8981100000000000001/balance"
```

```json
{
  "sim_id": "8981100000000000001",
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

「◯GB 中 ◯GB 残り」の前者が `total_granted_bytes`、後者が `total_remaining_bytes` です。
残量は `0 ≤ total_remaining_bytes ≤ total_granted_bytes` の範囲にクランプ済みなので、そのまま表示に使えます。

**繰越（Banked Data）について**: 使える総量 = 現プラン容量 + 繰越（未消化分）です。
消費順序は 現プラン → 繰越で、現プランの枯渇または期間満了時に繰越分は自動的に払い出されます
（お客様の操作は不要で、通信は途切れません）。残量は現プランと繰越をまたいで連続して減ります。
繰越は無期限ではなく、払い出し用の内部プランの期間終了時に消滅します。

`banked_data_bytes` は繰越残高の**内訳表示**で、`total_granted_bytes` / `total_remaining_bytes` に
**既に含まれています**。合計を出す際に加算しないでください。

`checked_at` は残量を確認した時刻、`latest_record_time` は残量算出に使った使用量データの
最新記録時刻です（いずれも UTC の `...Z` 形式）。通信実績が無い SIM では `latest_record_time` が `null` になります。

容量 3 値（`total_granted_bytes` / `total_remaining_bytes` / `used_bytes`）は容量上限型プランでのみ値を持ちます。
総枠が未確定の SIM（初回リチャージ前など）では `null` が返るため、
表示側で「残量非表示」に切り替える分岐を用意してください。

#### 日次上限型プラン（1GB/day 等）の当日枠

日次上限型プランは通算残量の概念を持たないため、容量 3 値は `null` のままで、
代わりに**当日の枠**を返します。

```json
{
  "sim_id": "8981100000000000002",
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

- `daily_used_bytes` は**日本時間の 0:00 以降**の実績の合計です。日付が変わると 0 に戻ります
- `daily_remaining_bytes` は `daily_limit_bytes − daily_used_bytes` で、0 未満にはなりません
- 日次上限のないプラン（無制限）や、リセット周期が 1 日でないプランでは
  `daily_limit_bytes` / `daily_remaining_bytes` が `null` になり、**当日利用量のみ**が返ります

レスポンスは全プランタイプで同一のスキーマです。**容量上限型プランでは `daily_*` が `null` として含まれ、
日次上限型プランでは容量 3 値が `null` として含まれます**。`plan_type` で表示を切り替えてください。

通信量の取得に失敗した場合は `503` を返します。古い値やキャッシュを返すことはありません。

### リチャージプランの確認

#### 特定 SIM で利用可能なプラン

```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/recharges/sims/sim-001/plans"
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

プランタイプは次の 2 種類です。

- `capacity`：容量上限型（`capacity_limit` に GB 単位の上限）
- `daily`：日次上限型（`daily_limit` に GB 単位の日次上限）

### リチャージの予約

`POST /v1/recharges/reservations` でリチャージを予約します。

```bash
curl -X POST -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"sim_id": "8981100000000000001", "plan_code": "PLAN-001"}' \
  "https://api.misora-connect.com/v1/recharges/reservations"
```

```json
{
  "reservation_id": "rsv-001",
  "status": "Reserved",
  "reserved_at": "2026-06-16T10:00:00Z"
}
```

### リチャージの実行タイミング

予約したリチャージが実際に適用されるタイミングは、プランタイプによって異なります。

| プランタイプ | 実行条件 |
|---|---|
| `capacity`（容量上限型） | 残データ量が 500MB 未満になった時点、または利用終了日の 21:00（JST）以降 |
| `daily`（日次上限型） | 利用終了日に到達後、毎日 20:00（JST）の日次処理で実行（残データ量による判定なし） |

※ 上限が 500MB 以下の小容量プランでは、上限に到達した時点で実行されます。

予約直後は `status` が `Reserved` となり、実行後に `Executed` へ変わります。

### 予約一覧の確認

```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/recharges/reservations"
```

次のフィルタリングオプションを利用できます。

- `sim_id`：特定 SIM の予約のみに絞り込みます。
- `status`：ステータスで絞り込みます（`Reserved`、`Executed`、`Failed`）。値は大文字始まりで指定してください。小文字を指定すると `400` になります。

```bash
# 特定 SIM の実行済み予約のみ
curl -H "x-api-key: YOUR_API_KEY" \
  "https://api.misora-connect.com/v1/recharges/reservations?sim_id=8981100000000000001&status=Executed"
```

### 即時リチャージ

`POST /v1/recharges/immediate` は、予約を挟まずにその場でプランを書き換えます。
残量は書き換えと同時にリセットされ、予約は作成と同時に実行済みになります。

通常のリチャージは「残量が少なくなったら適用する」予約方式です。
即時リチャージは残量の有無を問わず**その時点で現プランを置き換える**ため、
残っていたデータ量はリセットされます。運用ツールからの手動操作や、
利用者の求めに応じてその場で容量を追加する用途を想定しています。

対象は容量上限型（`capacity`）プランのみです。
日次上限型プラン、CPFR プラン、社内専用プランは指定できません。

```bash
curl -X POST -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"sim_id": "8981100000000000001", "plan_code": "PLAN-001"}' \
  "https://api.misora-connect.com/v1/recharges/immediate"
```

```json
{
  "sim_id": "8981100000000000001",
  "plan_code": "PLAN-001",
  "status": "Executed",
  "reservation_id": "rsv-001",
  "sync_status": "実行中",
  "superseded_reservation_ids": [],
  "error_code": null,
  "message": "Immediate recharge executed. PCRF sync triggered."
}
```

成功直後の `sync_status` は `実行中` です。
ネットワーク側への反映が完了すると `実行済` に変わります。

#### 実行待ちの予約がある場合

対象 SIM に実行待ちの予約が残っていると `409 RESERVATION_EXISTS` になります。
既存の予約を取り消して前倒し実行する場合は `force` を指定してください。
取り消した予約の ID は `superseded_reservation_ids` に入ります。

```bash
curl -X POST -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"sim_id": "8981100000000000001", "plan_code": "PLAN-001", "force": true}' \
  "https://api.misora-connect.com/v1/recharges/immediate"
```

#### 連続実行時の注意

直前の書き換えがネットワークへ反映されている間は `409 ALREADY_SYNCING` を返します。
これは `force` を指定しても回避できません。`sync_status` が `実行済` になるのを待ってから再試行してください。

なお `503 USAGE_UNAVAILABLE`（通信量の取得失敗）が返った場合、プランの書き換えは行われていません。
そのまま再試行して問題ありません。

## 共通仕様

### バイト単位の通信量

通信量はすべてバイト単位で返されます。
表示用に変換する例を次に示します。

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

日時は ISO 8601 形式で返されます。
月次指定は `YYYYMM` 形式です。

```
2026-06-16T10:00:00Z    # ISO 8601
202606                   # 年月（year_month フィールド）
```
