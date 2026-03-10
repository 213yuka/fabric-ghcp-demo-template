# デモ設計書テンプレート

`[...]` をユーザー回答と MCP 調査結果で埋めて `demo-output/demo-spec.md` を生成する。

---

# デモ設計書: [シナリオ名]

> 生成日: [日付]

---

## 1. 概要

| 項目 | 内容 |
|---|---|
| シナリオ | [例: 製造ライン品質管理] |
| 業種 | [例: 製造] |
| 対象者 | [例: 経営層] |
| ゴール | [例: Data Agent で自然言語からデータ分析できることを示す] |

---

## 2. アーキテクチャ

### ワークスペース構成

```
[シナリオ名]-demo-ws/
├── Lakehouse: [名前]_lakehouse
│   ├── Tables/          ← Tables_LoadTable API で自動変換
│   │   ├── fact_[xxx]
│   │   ├── dim_[xxx]
│   │   ├── dim_[yyy]
│   │   └── dim_date
│   ├── Files/
│   │   ├── fact_[xxx].csv   ← MCP でアップロード
│   │   ├── dim_[xxx].csv
│   │   └── dim_date.csv
│   └── SQL Analytics Endpoint  ← 自動プロビジョニング
├── Semantic Model: [名前]_model   ← スタースキーマ (Direct Lake)
└── Data Agent: [名前]_agent       ← 自然言語クエリ
```

### データフロー

```
変換済み CSV（スタースキーマ形式）
  ↓  MCP: onelake_upload_file
Lakehouse Files/
  ↓  REST API: Tables_LoadTable
Delta Tables (fact_ / dim_)
  ↓  自動
SQL Analytics Endpoint
  ↓  Direct Lake
Semantic Model (スタースキーマ、TMDL 定義付き)
  ↓
Data Agent (自然言語で質問)
```

---

## 3. データ設計（スタースキーマ）

### ER 図

```
           dim_date
              │
dim_[xxx] ── fact_[xxx] ── dim_[yyy]
              │
           dim_[zzz]
```

### ファクトテーブル: fact_[xxx]

| カラム | 型 | 説明 |
|---|---|---|
| [xxx]_key | int | サロゲートキー |
| date_key | int | → dim_date |
| [dim]_key | int | → dim_[xxx] |
| [メジャー名] | float/int | [説明] |

### ディメンション: dim_[xxx]

| カラム | 型 | 説明 |
|---|---|---|
| [xxx]_key | int | サロゲートキー |
| [属性名] | string | [説明] |

### ディメンション: dim_date

| カラム | 型 | 説明 |
|---|---|---|
| date_key | int | 20250101 形式 |
| date | date | 日付 |
| year | int | 年 |
| month | int | 月 |
| month_name | string | 月名（日本語） |
| quarter | string | Q1〜Q4 |
| day_of_week | string | 曜日（日本語） |

---

## 4. サンプルデータ

### fact_[xxx] (抜粋 3-5行)

| [key] | date_key | [dim_key] | [メジャー] |
|---|---|---|---|
| ... | ... | ... | ... |

### dim_[xxx] (抜粋 3-5行)

| [key] | [属性] |
|---|---|
| ... | ... |

---

## 5. Data Agent 設計

### 基本設計

| 項目 | 内容 |
|---|---|
| エージェント名 | [名前]_agent |
| データソース | Semantic Model: [名前]_model |
| スコープ | [例: 製造ラインの品質データに関する質問] |
| 対象外 | [例: 人事データ、財務詳細] |

### インストラクション案

```
[Data Agent に設定するインストラクションのドラフト]
例:
- このエージェントは [業種] の [シナリオ] データを分析します
- 日付は日本の会計年度（4月〜3月）で集計してください
- 金額は日本円で表示してください
```

### 代表質問セット

デモで使用する質問例。Data Agent 設定後にこれらで動作検証する:

| # | 質問 | 期待される回答の概要 |
|---|---|---|
| 1 | [例: 今月の売上合計は？] | [期待値の概要] |
| 2 | [例: 売上トップ5の商品は？] | [期待値の概要] |
| 3 | [例: 前月比で売上が増えたカテゴリは？] | [期待値の概要] |
| 4 | [例: 地域別の売上比較を見せて] | [期待値の概要] |
| 5 | [例: 品質不良率が高いラインは？] | [期待値の概要] |

---

## 6. 構築手順

### GHCP 内で自動実行

| Step | 操作 | ツール / API |
|---|---|---|
| 1 | ワークスペース選択 or 作成 | MCP `onelake_workspace_list` + REST API |
| 2 | Lakehouse 作成 | MCP `onelake_item_create` |
| 3 | CSV アップロード | MCP `onelake_upload_file` |
| 4 | CSV → Delta 変換 | ターミナルから Fabric REST API `Tables_LoadTable` |
| 5 | Semantic Model 作成 | MCP `onelake_item_create` |
| 6 | Data Agent 作成 | MCP `onelake_item_create` |

### Fabric ポータルで設定補完

| Step | 操作 |
|---|---|
| 1 | Semantic Model → モデル レイアウトでテーブル追加 + リレーションシップ設定（TMDL 自動適用成功時は不要） |
| 2 | Data Agent → データソースとして Semantic Model を選択 |
| 3 | 代表質問セットで動作検証 |

---

## 付録

### 調査に使った MCP ツール

- [publicapis_list / publicapis_get で取得した情報のサマリー]
- [bestpractices_itemdefinition_get で取得したスキーマ情報]

### 参照した公式ドキュメント

| 資料タイトル | URL | 要点 |
|---|---|---|
| [タイトル] | [URL] | [この設計に関連する要点] |
