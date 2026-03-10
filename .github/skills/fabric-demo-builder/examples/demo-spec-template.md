# デモ環境指示書テンプレート

このテンプレートに沿って `demo-output/demo-spec.md` を生成してください。
`[...]` の部分をユーザーの回答と MCP 調査結果で埋めてください。

---

# デモ環境指示書: [シナリオ名]

> 生成日: [日付]
> 生成元: fabric-ghcp-demo-template / fabric-demo-create

---

## 1. デモ概要

| 項目 | 内容 |
|---|---|
| シナリオ名 | [例: 製造ライン品質管理ダッシュボード] |
| 業種 | [例: 製造] |
| ユースケース | [例: IoT センサーデータによるリアルタイム品質監視] |
| 対象者 | [例: 経営層・品質管理チーム] |
| デモ時間 | [例: 30分] |
| デモ規模 | [ミニマル / 標準 / フル] |
| デモのゴール | [例: Fabric の統合データ分析基盤としての価値を示す] |

---

## 2. アーキテクチャ設計

### 使用ワークロード

| ワークロード | 用途 | 選定理由 |
|---|---|---|
| [例: Lakehouse] | [データ保存] | [構造化・非構造化データの統合管理] |
| [例: Notebook] | [データ加工] | [PySpark による柔軟な変換処理] |
| [例: Data Pipeline] | [データ取込] | [スケジュール実行によるバッチ取込] |
| [例: Semantic Model] | [分析モデル] | [Power BI 向けデータモデリング] |
| [例: Power BI Report] | [可視化] | [インタラクティブなダッシュボード] |

### ワークスペース構成

```
ワークスペース: [シナリオ名]-demo-ws
├── Lakehouse: [名前]_lakehouse
│   ├── Tables/
│   │   ├── [テーブル1]
│   │   ├── [テーブル2]
│   │   └── [テーブル3]
│   └── Files/
│       └── raw/
├── Notebook: [名前]_etl
├── Data Pipeline: [名前]_pipeline
├── Semantic Model: [名前]_model
└── Power BI Report: [名前]_dashboard
```

### データフロー

```
[データソース]  ──→  Data Pipeline  ──→  Lakehouse (raw)
                                            │
                                        Notebook (ETL)
                                            │
                                        Lakehouse (curated tables)
                                            │
                                        Semantic Model
                                            │
                                        Power BI Report
```

---

## 3. データ設計

### テーブル一覧

#### テーブル: [テーブル名1]

| カラム名 | 型 | 説明 |
|---|---|---|
| [例: id] | string | 一意識別子 |
| [例: timestamp] | datetime | 記録日時 |
| [例: value] | float | 計測値 |
| [例: category] | string | カテゴリ |

**サンプルデータ (3-5行):**

| id | timestamp | value | category |
|---|---|---|---|
| [データ] | [データ] | [データ] | [データ] |

#### テーブル: [テーブル名2]

(同様の形式で繰り返す)

### データソース

| ソース名 | 形式 | 件数 | 更新頻度 |
|---|---|---|---|
| [例: sensor_data] | CSV | 1,000 | バッチ (日次) |
| [例: master_data] | CSV | 50 | 静的 |

---

## 4. デモストーリー

| # | ステップ | 内容 | 時間 | 見せるもの | 話すポイント |
|---|---|---|---|---|---|
| 1 | 導入 | [課題とゴールの説明] | [3分] | スライド | [なぜ Fabric か] |
| 2 | データ取込 | [Pipeline でデータを投入] | [5分] | Pipeline 画面 | [統合されたデータ基盤] |
| 3 | データ加工 | [Notebook で変換処理] | [7分] | Notebook 実行 | [Spark の柔軟性] |
| 4 | 分析・可視化 | [Power BI レポートを表示] | [10分] | ダッシュボード | [リアルタイムな意思決定] |
| 5 | まとめ | [Fabric の価値を要約] | [5分] | スライド | [次のステップ] |

---

## 5. 構築手順

### Step 1: ワークスペースの作成

[Fabric API によるワークスペース作成手順]

```python
# API リクエスト例
import requests

url = "https://api.fabric.microsoft.com/v1/workspaces"
headers = {"Authorization": "Bearer <token>"}
body = {"displayName": "[ワークスペース名]"}

response = requests.post(url, headers=headers, json=body)
```

### Step 2: Lakehouse の作成

[Lakehouse 作成の API 呼び出し]

### Step 3: データの投入

[サンプルデータのアップロード手順]

### Step 4: Notebook の作成と実行

[ETL Notebook の作成手順]

### Step 5: Semantic Model と Power BI レポート

[レポート構築の手順]

### 注意事項

- [認証・権限に関する注意]
- [キャパシティ要件]
- [その他の前提条件]

---

## 6. サンプルコード

### ETL Notebook (`demo-output/notebooks/etl_notebook.py`)

```python
# [Notebook のコード全体をここに記載]
```

---

## 付録

### 使用した Fabric API 仕様

- [調査した API のサマリー — MCP ツールの調査結果に基づく]

### 参考ドキュメント

- [MS Learn から取得した関連ドキュメントへのリンク]
