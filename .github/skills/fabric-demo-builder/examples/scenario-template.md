# シナリオテンプレート

デモシナリオを定義する際に、以下のテンプレートを埋めてください。

---

## シナリオ基本情報

| 項目 | 内容 |
|---|---|
| シナリオ名 | (例: 製造ライン品質管理ダッシュボード) |
| 業種 | (例: 製造) |
| ユースケース | (例: IoT センサーデータによるリアルタイム品質監視) |
| 対象者 | (例: 製造業の経営層・品質管理チーム) |
| デモ時間 | (例: 30分) |
| デモ形式 | (例: スライド + ライブデモ) |

## データソース

| データソース名 | 形式 | 件数 | 更新頻度 | 説明 |
|---|---|---|---|---|
| (例: sensor_readings) | CSV / JSON / API | 1000 | リアルタイム | センサー計測値 |
| (例: product_master) | CSV | 50 | 静的 | 製品マスタ |

## テーブル設計

### テーブル名: (例: sensor_readings)

| カラム名 | 型 | 説明 |
|---|---|---|
| (例: timestamp) | datetime | 計測日時 |
| (例: sensor_id) | string | センサー ID |
| (例: temperature) | float | 温度（℃） |
| (例: pressure) | float | 圧力（hPa） |
| (例: status) | string | 正常 / 警告 / 異常 |

## Fabric ワークスペース構成

```
ワークスペース: demo-[シナリオ名]-[YYYYMMDD]
├── Lakehouse: [名前]_lakehouse
│   ├── Tables/
│   │   ├── fact_[テーブル1]
│   │   ├── dim_[テーブル2]
│   │   └── dim_date
│   └── Files/
│       └── *.csv（アップロード済みの変換済みCSV）
├── SQL Analytics Endpoint（Lakehouse と同時に自動作成）
├── Semantic Model: [名前]_model（Direct Lake、スタースキーマ）
└── Data Agent: [名前]_agent（自然言語クエリ）
```

> **Notebook・Pipeline・Power BI Report は作成しない。** CSV は変換済みで生成するため ETL 不要。分析は Data Agent で自然言語クエリを使用する。

## デモストーリー

| ステップ | 内容 | 所要時間 | 見せるもの |
|---|---|---|---|
| 1. 導入 | 課題とゴールの説明 | 3分 | スライド |
| 2. 環境紹介 | Fabric ワークスペースと Lakehouse の構成を紹介 | 5分 | Fabric ポータル |
| 3. データ確認 | Delta テーブルのスタースキーマ構成を確認 | 5分 | SQL Analytics Endpoint |
| 4. Data Agent デモ | 自然言語でデータに質問し、回答を確認 | 12分 | Data Agent 画面 |
| 5. まとめ | Fabric + Data Agent の価値を要約 | 5分 | スライド |

## 補足事項

- (デモ環境の制約、事前準備が必要な項目などを記載)
