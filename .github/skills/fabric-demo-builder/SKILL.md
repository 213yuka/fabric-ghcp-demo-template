---
name: fabric-demo-builder
description: >
  Microsoft Fabric のデモ環境を GHCP 内で構築するスキル。
  Fabric MCP の OneLake ツール + Fabric REST API で、Lakehouse 作成からデータ投入・Data Agent 構築まで
  GHCP 内で実行する。
  データモデルはスタースキーマで設計する。
  CSV は変換済み（ETL 不要）の状態で生成し、REST API Tables_LoadTable で Delta に変換する。
  `onelake_item_create` に `definition` パラメータはない。TMDL は REST API updateDefinition で別途適用する。
  MCP ツールにない API はターミナルで Fabric REST API を直接呼び出す。
  WHEN: Fabric デモ、デモ環境、ワークスペース構成、サンプルデータ、デモシナリオ。
---

# Fabric デモ構築スキル

Fabric MCP の OneLake ツール + Fabric REST API を使って、Data Agent デモ環境を構築するスキル。
CSV データは **変換済み（スタースキーマ形式）** で生成するため ETL は不要。
全アイテムの作成・設定が GHCP 内で完結する。ポータルでの手動設定は不要。
各フェーズ開始時に `現在の進捗: フェーズ X/5 - [フェーズ名]` を表示し、全 5 フェーズ中の現在位置をユーザーに明示する。

---

## 入力契約

このスキルを実行する前に、以下の情報が揃っていること:

| 入力 | 必須 | 説明 |
|---|---|---|
| 業種・シナリオ | ✅ | 例: 製造業の品質管理、小売の売上分析 |
| デモの目的・対象者 | ✅ | 例: 経営層向け、エンジニア向けハンズオン |
| 追加データ | ❌ | ユーザーが添付した Excel / CSV 等 |

## 出力契約

| 出力 | 形式 | 保存先 |
|---|---|---|
| デモ設計書 | Markdown | `demo-output/demo-spec.md` |
| ファクトテーブル CSV | CSV（変換済み） | `demo-output/data/fact_[xxx].csv` |
| ディメンションテーブル CSV | CSV（変換済み） | `demo-output/data/dim_[xxx].csv` |
| 日付ディメンション CSV | CSV（変換済み） | `demo-output/data/dim_date.csv` |
| 代表質問セット | Markdown（設計書内） | `demo-output/demo-spec.md` の「代表質問」節 |
| 活用ガイド | Markdown | `demo-output/usage-guide.md` |
| Fabric アイテム | Lakehouse / SemanticModel / DataAgent | 選択したワークスペース |

---

## 全体フロー

```
1. ヒアリング（業種・目的 + 追加データの有無）
   ↓
2. MCP で API 仕様を調査（publicapis_list → publicapis_get → item definition）
   ↓
3. デモ設計書 + 変換済みサンプルデータを生成（ローカル）
   ※ 追加データがあればスタースキーマに組み込む
   ↓
4. Fabric 環境を構築（MCP + Fabric REST API）
   ├── 新規ワークスペース作成（REST API）
   ├── Lakehouse 作成（MCP）
   ├── CSV アップロード（MCP）
   ├── CSV → Delta 変換（REST API）
   ├── Semantic Model 作成（REST API、定義付き）
   └── Data Agent 作成（REST API、定義付き: データソース + AI インストラクション）
```

---

## ワークロード構成

すべてのデモで以下を **標準構成** とする:

| ワークロード | 用途 | 作成方法 |
|---|---|---|
| **Lakehouse** | データ保存（ファクト＋ディメンション） | REST API `POST /v1/workspaces/{id}/lakehouses` |
| **SQL Analytics Endpoint** | Delta テーブルへの SQL アクセス（Direct Lake） | Lakehouse と同時に自動作成 |
| **Semantic Model** | スタースキーマの分析モデル | REST API `POST /v1/workspaces/{id}/semanticModels`（定義付き） |
| **Data Agent** | 自然言語でデータに質問できるエージェント | REST API `POST /v1/workspaces/{id}/DataAgents`（定義付き: データソース + AI インストラクション） |

> **Notebook は作成しない** — CSV は変換済みの状態で生成するため、ETL 処理は不要。
> CSV → Delta 変換は Fabric REST API `Tables_LoadTable` でターミナルから実行する（MCP ツールにはない）。

---

## スタースキーマ設計ルール

セマンティックモデルは必ず **スタースキーマ** で設計する。

### 構造

```
        dim_date
           │
dim_xxx ── fact_xxx ── dim_yyy
           │
        dim_zzz
```

### ファクトテーブル（`fact_` プレフィックス）

- ビジネスイベント（売上、計測、注文 等）を記録する
- 数値メジャー（金額、数量、値 等）を持つ
- 各ディメンションへの外部キー（`_key` サフィックス）を持つ

### ディメンションテーブル（`dim_` プレフィックス）

- マスタデータ（商品、顧客、日付、拠点 等）
- テキスト属性（名前、カテゴリ、地域 等）を持つ
- サロゲートキー（`_key`）を主キーにする

### 日付ディメンション（全デモ共通・必須）

| カラム | 型 | 例 |
|---|---|---|
| date_key | int | 20250101 |
| date | date | 2025-01-01 |
| year | int | 2025 |
| month | int | 1 |
| month_name | string | 1月 |
| quarter | string | Q1 |
| day_of_week | string | 月曜日 |

---

## サンプルデータ（変換済み CSV）

- ファクト: 100〜500 行 / ディメンション: 10〜50 行
- 日本語の値を含める（カテゴリ名、地域名 等）
- 日付範囲: 直近 3〜6 ヶ月
- 集計・フィルタで差が出るデータにする
- **CSV はスタースキーマ形式で生成済み** — そのまま Delta テーブルとしてロード可能

---

## Data Agent 構築の前提条件

Data Agent を作成する前に、以下が完了していること:

- [ ] Lakehouse が作成済みで、CSV がアップロードされている
- [ ] CSV → Delta テーブル変換が Tables_LoadTable API で完了している
- [ ] Semantic Model が作成済み（リレーションシップ定義含む）
- [ ] スタースキーマのリレーションシップ設計が完了している
- [ ] テーブル名・カラム名が業務用語で明確に命名されている（AI-ready）
- [ ] 代表質問セット（5〜10 個）が用意されている
- [ ] エージェントインストラクションが準備されている
- [ ] データソースインストラクション（テーブル構造・リレーション・業務用語）が準備されている

### AI-ready データ準備

Data Agent が正確なクエリを生成するために、以下のベストプラクティスに従う:

| カテゴリ | ルール |
|---|---|
| テーブル命名 | `fact_sales`, `dim_product` のように意味が明確な名前 |
| カラム命名 | `customer_segment`, `order_date` のように説明的（`col1` は NG） |
| メジャー定義 | よく使う集計（売上合計、平均単価等）は Semantic Model のメジャーとして定義 |
| テーブル数 | 1 データソースあたり **25 テーブル以下** を推奨 |
| リレーションシップ | ファクト → ディメンションの多対一を TMDL で明示的に定義 |
| 値の形式 | 略称 vs フルネーム等をインストラクションに明記 |

> 参考: [Best practices for configuring your data agent](https://learn.microsoft.com/fabric/data-science/data-agent-configuration-best-practices)

### Data Agent 設定項目

| 設定 | 内容 | 設定方法 |
|---|---|---|
| データソース | Lakehouse または Semantic Model を追加（最大5つ） | REST API: definition の datasource.json で指定 |
| エージェントインストラクション | 回答ルール・業務用語・日付の定義 | REST API: definition の stage_config.json で指定 |
| データソースインストラクション | テーブル構造・リレーション・クエリロジック | REST API: datasource.json の dataSourceInstructions で指定 |
| 代表質問セット | 動作検証用の質問例 | REST API: fewshots.json で指定 + 設計書に記載 |

### Data Agent ベストプラクティス

| カテゴリ | ルール |
|---|---|
| データソース | Semantic Model を優先的に使う。Lakehouse は補完用途 |
| スコープ | 1 エージェントに複数ドメインを詰め込みすぎない |
| 命名 | テーブル名・列名・メジャー名を業務用語で整える |
| KPI 定義 | 集計粒度、日付の定義（年度 vs 暦年等）を明示する |
| 同義語 | 曖昧語や同義語に対応できるよう説明・別名を整備する |
| インストラクション | 具体的に何をすべきかを書く（「～しない」だけでは不十分） |
| 検証 | デモ前に代表質問セットで回答品質を検証する |
| ガバナンス | 機密データは対象外にする。権限は最小化 |

---

## MCP 実行手順（GHCP 内）

### 事前チェック（必須）

実行系ツールを使う前に、必ず調査系ツールで最新仕様を確認する:

1. `publicapis_list` → ワークロード名・item-type の確認
2. `publicapis_get` → Lakehouse / SemanticModel / DataAgent の API 仕様
3. `publicapis_bestpractices_examples_get` → リクエスト/レスポンス例
4. `publicapis_bestpractices_itemdefinition_get` → アイテム定義スキーマ（取得できる workload の場合のみ）

> **ワークロード名や item-type を固定文字列で決め打ちしない。**
> 必ず上記のツールで最新値を確認してから実行する。
> `publicapis_bestpractices_itemdefinition_get` が 404 の workload では、`publicapis_get` の definitions と examples を正とする。

### Step 1: 新規ワークスペース作成
既存ワークスペースは使わず、Fabric REST API をターミナルから呼び出して毎回新規作成する:
```powershell
$token = (az account get-access-token --resource https://api.fabric.microsoft.com --query accessToken -o tsv)
Invoke-RestMethod -Uri "https://api.fabric.microsoft.com/v1/workspaces" `
  -Method POST -Headers @{ Authorization = "Bearer $token" } `
  -ContentType "application/json" -Body '{"displayName": "demo-[scenario]-[YYYYMMDD]"}'
```
> **注意**: MCP ツールにワークスペース作成機能はないため、ワークスペースは REST API で新規作成する。

### Step 2: Lakehouse 作成
Fabric REST API で Lakehouse を作成。Step 1 で取得した `$workspaceId` をそのまま使う。
```powershell
Invoke-RestMethod -Uri "https://api.fabric.microsoft.com/v1/workspaces/$workspaceId/lakehouses" `
  -Method POST -Headers @{ Authorization = "Bearer $token" } `
  -ContentType "application/json" -Body '{"displayName": "[name]_lakehouse"}'
```
（SQL Analytics Endpoint は自動でプロビジョニングされる）

### Step 3: CSV アップロード
`onelake_upload_file` — 変換済みの CSV を `Files/` にアップロード
**必須パラメータ:**
- `file-path`: OneLake 上のパス（例: `Files/fact_sales.csv`）
- `local-file-path`: ローカル CSV ファイルの絶対パス（例: `demo-output/data/fact_sales.csv`）
- `item`: Lakehouse 名 + 型サフィックス（例: `myLakehouse.Lakehouse`）
- `workspace`: ワークスペース名
- `overwrite`: true

### Step 4: CSV → Delta テーブル変換
MCP ツールには Load Table API がないため、**Fabric REST API をターミナルから直接呼び出す**。
```powershell
$body = @{
  relativePath = "Files/[tableName].csv"
  pathType = "File"
  mode = "Overwrite"
  formatOptions = @{ format = "Csv"; header = $true; delimiter = "," }
} | ConvertTo-Json -Depth 3

Invoke-RestMethod `
  -Uri "https://api.fabric.microsoft.com/v1/workspaces/$workspaceId/lakehouses/$lakehouseId/tables/$tableName/load" `
  -Method POST -Headers @{ Authorization = "Bearer $token" } `
  -ContentType "application/json" -Body $body
```
これは LRO（非同期）。202 が返ったらステータスを確認して完了を待つ。全 CSV に対して順番に実行する。

### Step 5: Semantic Model 作成
Fabric REST API で **定義付き**（definition.pbism + model.bim）で作成する。MCP の `onelake_item_create` は使わない。

> **注意**: MCP の `onelake_item_create` には `definition` パラメータがない。
> Semantic Model は空では作成できないため、REST API で定義付きで作成する。

### Step 6: Data Agent 作成（定義付き: データソース + AI インストラクション）
Fabric REST API で **定義付き** で Data Agent を作成する。
```
POST https://api.fabric.microsoft.com/v1/workspaces/{workspaceId}/DataAgents
```
definition に以下の parts を含める:
- `Files/Config/data_agent.json` — `{"$schema": "2.1.0"}`
- `Files/Config/draft/stage_config.json` — AI インストラクション
- `Files/Config/draft/{type}-{name}/datasource.json` — データソース設定（Lakehouse の artifactId, workspaceId, type を指定）
- `Files/Config/draft/{type}-{name}/fewshots.json` — 代表質問セット

**Semantic Model が作成済みであること。**
Data Agent のデータソースには Lakehouse（type: `lakehouse-tables`）または Semantic Model（type: `semantic_model`）を指定できる。

> 参考: [Data Agent definition](https://learn.microsoft.com/rest/api/fabric/articles/item-management/definitions/data-agent-definition)

---

## 完了確認

MCP + REST API で全アイテムの作成・設定が完了する。ポータルでの手動設定は不要:

1. **Semantic Model** — REST API で定義付きで作成済み。リレーションシップ・メジャーが設定済み。
2. **Data Agent** — REST API で定義付きで作成済み。データソース・AI インストラクション・代表質問が設定済み。
3. **動作検証** — 代表質問セットで回答品質を確認

---

## 失敗時の対処

| 失敗ケース | 対処 |
|---|---|
| `onelake_upload_file` がエラー | Lakehouse が正常に作成されたか確認。ファイルパスを確認 |
| `onelake_upload_file` がエラー | Lakehouse が正常に作成されたか `onelake_item_list` で確認。ファイルパスを確認 |
| `Tables_LoadTable` がエラー | CSV ファイル名のパターン・パスを確認。LRO 完了前にタイムアウトした場合はリトライ |
| Semantic Model TMDL エラー | TMDL 構文を確認。テーブル名が Delta テーブルと一致しているか確認 |
| MCP サーバー接続エラー | VS Code で MCP サーバーを再起動。認証の有効期限を確認 |
| CSV のキー不整合 | ファクトテーブルのキーがディメンションに存在するか検証 |

---

## 出力ファイル

```
demo-output/
├── demo-spec.md        — デモ設計書（代表質問セット含む）
├── usage-guide.md      — 活用ガイド（質問例・デモシナリオ・トラブルシューティング）
└── data/
    ├── fact_xxx.csv    — ファクトテーブル（変換済み）
    ├── dim_xxx.csv     — ディメンションテーブル（変換済み）
    └── dim_date.csv    — 日付ディメンション（変換済み）
```

テンプレート:
- 設計書: [examples/demo-spec-template.md](./examples/demo-spec-template.md)
- 活用ガイド: [examples/usage-guide-template.md](./examples/usage-guide-template.md)
