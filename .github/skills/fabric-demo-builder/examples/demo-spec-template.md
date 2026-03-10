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
| ゴール | [例: Fabric の統合分析基盤としての価値を示す] |

---

## 2. アーキテクチャ

### ワークスペース構成

```
[シナリオ名]-demo-ws/
├── Lakehouse: [名前]_lakehouse
│   ├── Tables/
│   │   ├── fact_[xxx]
│   │   ├── dim_[xxx]
│   │   ├── dim_[yyy]
│   │   └── dim_date
│   └── Files/raw/
│       ├── fact_[xxx].csv
│       ├── dim_[xxx].csv
│       └── dim_date.csv
├── Notebook: [名前]_etl
├── Semantic Model: [名前]_model   ← スタースキーマ
└── Data Agent: [名前]_agent       ← 自然言語クエリ
```

### データフロー

```
CSV (Files/raw/)
  ↓  Notebook (PySpark)
Delta Tables (fact_ / dim_)
  ↓
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

## 5. GHCP 実行手順

以下を Copilot チャット内で MCP ツールを使って実行する:

### Step 1: ワークスペース確認
- `onelake_workspace_list` でワークスペース一覧を表示
- ユーザーにデプロイ先を選んでもらう

### Step 2: Lakehouse 作成
- `onelake_item_create` — display-name: `[名前]_lakehouse`, item-type: `Lakehouse`

### Step 3: CSV アップロード
- `onelake_upload_file` で `demo-output/data/` 内の CSV を Lakehouse にアップロード
  - file-path: `Files/raw/fact_[xxx].csv`
  - file-path: `Files/raw/dim_[xxx].csv`
  - file-path: `Files/raw/dim_date.csv`

### Step 4: Notebook 作成
- `onelake_item_create` — display-name: `[名前]_etl`, item-type: `Notebook`

### Step 5: Semantic Model 作成
- `onelake_item_create` — display-name: `[名前]_model`, item-type: `SemanticModel`

### Step 6: Data Agent 作成
- `onelake_item_create` — display-name: `[名前]_agent`, item-type: `DataAgent`

---

## 6. ETL Notebook コード

```python
# --- CSV を読み込んで Delta テーブルに変換 ---

# ファクトテーブル
df_fact = spark.read.option("header", True).option("inferSchema", True) \
    .csv("Files/raw/fact_[xxx].csv")
df_fact.write.mode("overwrite").format("delta").saveAsTable("fact_[xxx]")

# ディメンション
df_dim = spark.read.option("header", True).option("inferSchema", True) \
    .csv("Files/raw/dim_[xxx].csv")
df_dim.write.mode("overwrite").format("delta").saveAsTable("dim_[xxx]")

# 日付ディメンション
df_date = spark.read.option("header", True).option("inferSchema", True) \
    .csv("Files/raw/dim_date.csv")
df_date.write.mode("overwrite").format("delta").saveAsTable("dim_date")

# --- 確認 ---
display(spark.sql("SELECT * FROM fact_[xxx] LIMIT 5"))
display(spark.sql("SELECT * FROM dim_[xxx] LIMIT 5"))
```

---

## 付録

### 調査に使った MCP ツール

- [publicapis_list / publicapis_get で取得した情報のサマリー]
- [bestpractices_itemdefinition_get で取得したスキーマ情報]
