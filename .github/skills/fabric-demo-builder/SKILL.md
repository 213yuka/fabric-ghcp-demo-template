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

---

## 入力契約

このスキルを実行する前に、以下の情報が揃っていること:

| 入力 | 必須 | 説明 |
|---|---|---|
| 業種・シナリオ | ✅ | 例: 製造業の品質管理、小売の売上分析 |
| デモの目的・対象者 | ✅ | 例: 経営層向け、エンジニア向けハンズオン |
| デモの規模 | ✅ | ミニマル / 標準 / フル |
| 追加データ | ❌ | ユーザーが添付した Excel / CSV 等 |

## 出力契約

| 出力 | 形式 | 保存先 |
|---|---|---|
| デモ設計書 | Markdown | `demo-output/demo-spec.md` |
| ファクトテーブル CSV | CSV（変換済み） | `demo-output/data/fact_[xxx].csv` |
| ディメンションテーブル CSV | CSV（変換済み） | `demo-output/data/dim_[xxx].csv` |
| 日付ディメンション CSV | CSV（変換済み） | `demo-output/data/dim_date.csv` |
| 代表質問セット | Markdown（設計書内） | `demo-output/demo-spec.md` の「代表質問」節 |
| Fabric アイテム | Lakehouse / SemanticModel / DataAgent | 新規作成されたワークスペース |

---

## 全体フロー

```
1. ヒアリング（業種・目的・規模 + 追加データの有無）
   ↓
2. MCP で API 仕様を調査（publicapis_list → publicapis_get → item definition）
   ↓
3. デモ設計書 + 変換済みサンプルデータを生成（ローカル）
   ※ 追加データがあればスタースキーマに組み込む
   ↓
4. Fabric 環境を構築（MCP OneLake ツールで実行）
   ├── 新規ワークスペース作成
   ├── Lakehouse 作成
   ├── CSV アップロード（変換済み）
   ├── Semantic Model 作成 ← Lakehouse の後に作成
   └── Data Agent 作成    ← Semantic Model の後に作成
   ↓
5. Fabric ポータルで初期設定
   ├── CSV → Delta テーブル変換（Load to Tables）
   ├── Semantic Model のテーブル・リレーションシップ設定
   └── Data Agent のデータソース設定
```

---

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
- [ ] CSV → Delta テーブル変換の手順が設計書に記載されている
- [ ] Semantic Model が作成済み（ポータル設定は後続）
- [ ] スタースキーマのリレーションシップ設計が完了している
- [ ] テーブル名・カラム名が業務用語で明確に命名されている
- [ ] 代表質問セット（5〜10 個）が用意されている

### Data Agent ベストプラクティス

| カテゴリ | ルール |
|---|---|
| データソース | Semantic Model を優先的に使う。Lakehouse は補完用途 |
| スコープ | 1 エージェントに複数ドメインを詰め込みすぎない |
| 命名 | テーブル名・列名・メジャー名を業務用語で整える |
| KPI 定義 | 集計粒度、日付の定義（年度 vs 暦年等）を明示する |
| 同義語 | 曖昧語や同義語に対応できるよう説明・別名を整備する |
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

### Step 1: ワークスペース作成
デモ用の **新しいワークスペースを作成** する。
ワークスペース名: `demo-[シナリオ]-[YYYYMMDD]`（例: `demo-retail-sales-20260310`）
`onelake_workspace_list` で既存ワークスペースとの重複がないことを確認した上で作成する。

### Step 2: Lakehouse 作成
`onelake_item_create` — item-type: (publicapis_list で確認した値)
（SQL Analytics Endpoint は自動でプロビジョニングされる）

### Step 3: CSV アップロード
`onelake_upload_file` — 変換済みの CSV を `Files/` にアップロード
- `Files/fact_[xxx].csv`
- `Files/dim_[xxx].csv`
- `Files/dim_date.csv`

### Step 4: Semantic Model 作成
`onelake_item_create` — item-type: (publicapis_list で確認した値)

### Step 5: Data Agent 作成
`onelake_item_create` — item-type: (publicapis_list で確認した値)
**Semantic Model 作成後に実行すること。**

---

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
- データソース（Semantic Model を優先）を選択
- インストラクション（任意）を設定
- 代表質問セットで動作検証

---

## 失敗時の対処

| 失敗ケース | 対処 |
|---|---|
| `onelake_item_create` がエラー | `publicapis_list` で item-type が正しいか再確認。ワークスペース権限を確認 |
| `onelake_upload_file` がエラー | Lakehouse が正常に作成されたか `onelake_item_list` で確認。ファイルパスを確認 |
| MCP サーバー接続エラー | VS Code で MCP サーバーを再起動。認証の有効期限を確認 |
| CSV のキー不整合 | ファクトテーブルのキーがディメンションに存在するか検証 |

---

## 出力ファイル

```
demo-output/
├── demo-spec.md        — デモ設計書（代表質問セット含む）
└── data/
    ├── fact_xxx.csv    — ファクトテーブル（変換済み）
    ├── dim_xxx.csv     — ディメンションテーブル（変換済み）
    └── dim_date.csv    — 日付ディメンション（変換済み）
```

テンプレート: [examples/demo-spec-template.md](./examples/demo-spec-template.md)
