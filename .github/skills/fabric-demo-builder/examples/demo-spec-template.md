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
│   ├── Tables/          ← ポータルで Load to Tables
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
  ↓  ポータル: Load to Tables
Delta Tables (fact_ / dim_)
  ↓  自動
SQL Analytics Endpoint
  ↓  Direct Lake
Semantic Model (スタースキーマ)
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

## 5. 構築手順

### Part A: GHCP 内で実行（MCP ツール）

| Step | 操作 | MCP ツール |
|---|---|---|
| 1 | ワークスペース確認 | `onelake_workspace_list` |
| 2 | Lakehouse 作成 | `onelake_item_create` (Lakehouse) |
| 3 | CSV アップロード | `onelake_upload_file` |
| 4 | Semantic Model 作成 | `onelake_item_create` (SemanticModel) |
| 5 | Data Agent 作成 | `onelake_item_create` (DataAgent) |

### Part B: Fabric ポータルで設定

| Step | 操作 | 場所 |
|---|---|---|
| 1 | CSV → Delta テーブル変換 | Lakehouse → Files → 右クリック → Load to Tables |
| 2 | Semantic Model 設定 | SQL Analytics Endpoint → モデル レイアウト |
|   | - テーブル追加 | fact_ / dim_ テーブルを選択 |
|   | - リレーションシップ設定 | ファクトの _key → ディメンションの _key |
|   | - メジャー定義 | [必要なメジャーを列挙] |
| 3 | Data Agent 設定 | Data Agent → データソース選択 |

---

## 付録

### 調査に使った MCP ツール

- [publicapis_list / publicapis_get で取得した情報のサマリー]
- [bestpractices_itemdefinition_get で取得したスキーマ情報]
