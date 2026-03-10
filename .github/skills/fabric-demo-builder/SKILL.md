---
name: fabric-demo-builder
description: >
  Microsoft Fabric のデモ環境を GHCP 内で構築するスキル。
  Fabric MCP の OneLake ツールでワークスペース確認・Lakehouse 作成・
  データ投入・Semantic Model・Data Agent まですべて Copilot 内で完結する。
  データモデルはスタースキーマで設計する。
  WHEN: Fabric デモ、デモ環境、ワークスペース構成、サンプルデータ、デモシナリオ。
---

# Fabric デモ構築スキル

Fabric MCP の OneLake ツールを使って、**GHCP 内ですべて完結** するデモ構築スキル。

## 全体フロー

```
1. ヒアリング（業種・目的・規模）
   ↓
2. MCP で API 仕様を調査
   ↓
3. デモ設計書 + サンプルデータを生成（ローカル）
   ↓
4. Fabric 環境を構築（MCP OneLake ツールで実行）
   ├── ワークスペース確認
   ├── Lakehouse 作成
   ├── CSV アップロード
   ├── Notebook 作成
   ├── Semantic Model 作成（スタースキーマ）
   └── Data Agent 作成
```

## ワークロード構成

すべてのデモで以下を **標準構成** とする:

| ワークロード | 用途 | MCP ツール |
|---|---|---|
| **Lakehouse** | データ保存（ファクト＋ディメンション） | `onelake_item_create` |
| **Notebook** | ETL（CSV → Delta テーブル変換） | `onelake_item_create` |
| **Semantic Model** | スタースキーマの分析モデル | `onelake_item_create` |
| **Data Agent** | 自然言語でデータに質問できるエージェント | `onelake_item_create` |

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

## サンプルデータ

- ファクト: 100〜500 行 / ディメンション: 10〜50 行
- 日本語の値を含める（カテゴリ名、地域名 等）
- 日付範囲: 直近 3〜6 ヶ月
- 集計・フィルタで差が出るデータにする

## MCP 実行手順

### Step 1: ワークスペース確認
`onelake_workspace_list` → ユーザーに使用するワークスペースを選んでもらう

### Step 2: Lakehouse 作成
`onelake_item_create` — item-type: `Lakehouse`

### Step 3: CSV アップロード
`onelake_upload_file` — ファクト・ディメンションの CSV を `Files/raw/` にアップロード

### Step 4: Notebook 作成
`onelake_item_create` — item-type: `Notebook`

### Step 5: Semantic Model 作成
`onelake_item_create` — item-type: `SemanticModel`

### Step 6: Data Agent 作成
`onelake_item_create` — item-type: `DataAgent`

## 出力ファイル

```
demo-output/
├── demo-spec.md        — デモ設計書
├── data/
│   ├── fact_xxx.csv    — ファクトテーブル
│   ├── dim_xxx.csv     — ディメンションテーブル
│   └── dim_date.csv    — 日付ディメンション
└── notebooks/
    └── etl_notebook.py — CSV → Delta テーブル変換コード
```

テンプレート: [examples/demo-spec-template.md](./examples/demo-spec-template.md)
