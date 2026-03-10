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
主要な構築作業は GHCP 内で完結。Semantic Model ・ Data Agent の設定補完はポータルで行う。

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
   ├── ワークスペース選択 or 作成（REST API）
   ├── Lakehouse 作成（MCP）
   ├── CSV アップロード（MCP）
   ├── CSV → Delta 変換（REST API）
   ├── Semantic Model 作成（MCP）
   └── Data Agent 作成（MCP）
   ↓
5. Fabric ポータルで Semantic Model 設定補完 + Data Agent データソース選択
```

---

## ワークロード構成

すべてのデモで以下を **標準構成** とする:

| ワークロード | 用途 | 作成方法 |
|---|---|---|
| **Lakehouse** | データ保存（ファクト＋ディメンション） | MCP `onelake_item_create` |
| **SQL Analytics Endpoint** | Delta テーブルへの SQL アクセス（Direct Lake） | Lakehouse と同時に自動作成 |
| **Semantic Model** | スタースキーマの分析モデル | MCP `onelake_item_create` → ポータルで設定補完 |
| **Data Agent** | 自然言語でデータに質問できるエージェント | MCP `onelake_item_create` → ポータルでデータソース選択 |

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

| 設定 | 内容 | 設定場所 |
|---|---|---|
| データソース | Semantic Model を追加（最大5つ） | ポータル: Explorer → + Data source |
| エージェントインストラクション | 回答ルール・業務用語・日付の定義 | ポータル: Agent instructions |
| データソースインストラクション | テーブル構造・リレーション・クエリロジック | ポータル: データソース選択後 |
| 代表質問セット | 動作検証用の質問例 | 設計書に記載し、ポータルで検証 |

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
3. `publicapis_bestpractices_itemdefinition_get` → アイテム定義スキーマ

> **ワークロード名や item-type を固定文字列で決め打ちしない。**
> 必ず上記のツールで最新値を確認してから実行する。

### Step 1: ワークスペース選択または作成
`onelake_workspace_list` で既存ワークスペースを一覧表示し、ユーザーに使用するワークスペースを選んでもらう。
新規作成したい場合は、Fabric REST API をターミナルから呼び出す:
```powershell
$token = (az account get-access-token --resource https://api.fabric.microsoft.com --query accessToken -o tsv)
Invoke-RestMethod -Uri "https://api.fabric.microsoft.com/v1/workspaces" `
  -Method POST -Headers @{ Authorization = "Bearer $token" } `
  -ContentType "application/json" -Body '{"displayName": "demo-[scenario]-[YYYYMMDD]"}'
```
> **注意**: MCP ツールにワークスペース作成機能はない。既存選択 or REST API で作成。

### Step 2: Lakehouse 作成
`onelake_item_create` — パラメータ: `display-name`, `item-type`, `workspace`
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
`onelake_item_create` — パラメータ: `display-name`, `item-type: SemanticModel`, `workspace`

> **注意**: MCP の `onelake_item_create` には `definition` パラメータがない。
> TMDL 定義の適用は Fabric REST API `updateDefinition` で試みる。失敗した場合はスキップ。

### Step 6: Data Agent 作成
`onelake_item_create` — パラメータ: `display-name`, `item-type`, `workspace`
**Semantic Model 作成後に実行すること。**

> **注意**: Data Agent のデータソース設定は MCP ではできない。
> 完了報告でポータルでの設定を案内する。

---

## 完了後の設定（Fabric ポータル）

MCP + REST API で主要なアイテム作成・データ投入は完了するが、以下は Fabric ポータルでの設定が必要:

1. **Semantic Model** — TMDL 自動適用が失敗した場合: モデル レイアウトでテーブル追加 + リレーションシップ設定
2. **Data Agent** — データソースとして Semantic Model を選択
3. **動作検証** — 代表質問セットで回答品質を確認

---

## 失敗時の対処

| 失敗ケース | 対処 |
|---|---|
| `onelake_item_create` がエラー | `publicapis_list` で item-type が正しいか再確認。ワークスペース権限を確認。Fabric 容量が起動済み（アクティブ）か確認 |
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
