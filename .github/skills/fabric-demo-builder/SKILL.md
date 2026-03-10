---
name: fabric-demo-builder
description: >
  Microsoft Fabric のデモ環境を GHCP 内で構築するスキル。
  Fabric MCP の OneLake ツールでワークスペース確認・Lakehouse 作成・
  データ投入・Semantic Model・Data Agent まで Copilot 内で実行する。
  データモデルはスタースキーマで設計する。
  CSV は変換済み（ETL 不要）の状態で生成し、そのまま Lakehouse にアップロードする。
  WHEN: Fabric デモ、デモ環境、ワークスペース構成、サンプルデータ、デモシナリオ。
---

# Fabric デモ構築スキル

Fabric MCP の OneLake ツールを使って、Data Agent デモ環境を構築するスキル。
CSV データは **変換済み（スタースキーマ形式）** で生成するため ETL は不要。

## 全体フロー

```
1. ヒアリング（業種・目的・規模）
   ↓
2. MCP で API 仕様を調査
   ↓
3. デモ設計書 + 変換済みサンプルデータを生成（ローカル）
   ↓
4. Fabric 環境を構築（MCP OneLake ツールで実行）
   ├── ワークスペース確認
   ├── Lakehouse 作成
   ├── CSV アップロード（変換済み）
   ├── Semantic Model 作成
   └── Data Agent 作成
   ↓
5. Fabric ポータルで初期設定
   ├── CSV → Delta テーブル変換（Load to Tables）
   ├── Semantic Model のテーブル・リレーションシップ設定
   └── Data Agent のデータソース設定
```

## ワークロード構成

すべてのデモで以下を **標準構成** とする:

| ワークロード | 用途 | 作成方法 |
|---|---|---|
| **Lakehouse** | データ保存（ファクト＋ディメンション） | MCP `onelake_item_create` |
| **SQL Analytics Endpoint** | Delta テーブルへの SQL アクセス（Direct Lake） | Lakehouse と同時に自動作成 |
| **Semantic Model** | スタースキーマの分析モデル | MCP `onelake_item_create` + ポータルで設定 |
| **Data Agent** | 自然言語でデータに質問できるエージェント | MCP `onelake_item_create` + ポータルで設定 |

> **Notebook は作成しない** — CSV は変換済みの状態で生成するため、ETL 処理は不要。
> Lakehouse ポータルの「Load to Tables」で CSV → Delta テーブルに変換する。

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

## サンプルデータ（変換済み CSV）

- ファクト: 100〜500 行 / ディメンション: 10〜50 行
- 日本語の値を含める（カテゴリ名、地域名 等）
- 日付範囲: 直近 3〜6 ヶ月
- 集計・フィルタで差が出るデータにする
- **CSV はスタースキーマ形式で生成済み** — そのまま Delta テーブルとしてロード可能

## MCP 実行手順（GHCP 内）

### Step 1: ワークスペース確認
`onelake_workspace_list` → ユーザーに使用するワークスペースを選んでもらう

### Step 2: Lakehouse 作成
`onelake_item_create` — item-type: `Lakehouse`
（SQL Analytics Endpoint は自動でプロビジョニングされる）

### Step 3: CSV アップロード
`onelake_upload_file` — 変換済みの CSV を `Files/` にアップロード
- `Files/fact_[xxx].csv`
- `Files/dim_[xxx].csv`
- `Files/dim_date.csv`

### Step 4: Semantic Model 作成
`onelake_item_create` — item-type: `SemanticModel`

### Step 5: Data Agent 作成
`onelake_item_create` — item-type: `DataAgent`

## Fabric ポータルでの初期設定（MCP 実行後）

MCP でアイテム作成後、以下の設定を Fabric ポータルで行う:

### 1. CSV → Delta テーブル変換
Lakehouse を開き、`Files/` 内の各 CSV を右クリック → **「Load to Tables」** で Delta テーブルに変換。

### 2. Semantic Model 設定
SQL Analytics Endpoint で **モデル レイアウト** を開き:
- テーブル（fact_ / dim_）を選択して追加
- リレーションシップを設定（ファクトの `_key` → ディメンションの `_key`）
- 必要なメジャーを定義

### 3. Data Agent 設定
Data Agent を開き:
- データソース（Semantic Model または Lakehouse）を選択
- インストラクション（任意）を設定

## 出力ファイル

```
demo-output/
├── demo-spec.md        — デモ設計書
└── data/
    ├── fact_xxx.csv    — ファクトテーブル（変換済み）
    ├── dim_xxx.csv     — ディメンションテーブル（変換済み）
    └── dim_date.csv    — 日付ディメンション（変換済み）
```

テンプレート: [examples/demo-spec-template.md](./examples/demo-spec-template.md)
