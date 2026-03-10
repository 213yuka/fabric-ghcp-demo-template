---
name: 'MS Learn MCP ツール利用ガイド'
description: 'Microsoft Learn MCP サーバーのツールを適切に使い分けるためのルール'
applyTo: '**'
---

# Microsoft Learn MCP サーバーの使い方

Microsoft の公式ドキュメントやコードサンプルを参照するときは、`microsoft-learn` MCP サーバーのツールを使用すること。
この MCP サーバーは Microsoft Learn の最新の公式ドキュメントにアクセスできるため、トレーニングデータよりも新しい情報が含まれている可能性がある。

## 利用可能なツール

| ツール名 | 用途 | 使いどころ |
|---|---|---|
| `microsoft_docs_search` | ドキュメントの検索 | トピックを広く調べたいとき。キーワードで関連ドキュメントを探す |
| `microsoft_docs_fetch` | 特定ページの取得 | URL が分かっているドキュメントの全文を取得したいとき |
| `microsoft_code_sample_search` | コードサンプルの検索 | 実装例やサンプルコードを探したいとき |

## ツール利用の判断基準

- **概念・チュートリアル・設定方法の調査** → `microsoft_docs_search` で検索し、必要に応じて `microsoft_docs_fetch` で詳細取得
- **API リファレンスやコードサンプル** → `microsoft_code_sample_search` で具体例を検索
- **Fabric 固有の API 仕様** → MS Learn MCP ではなく **Fabric MCP サーバー** を使う（より詳細な OpenAPI 定義が取得可能）

## 使い分けの原則

- **Fabric の API 仕様・スキーマ・ベストプラクティス** → Fabric MCP を優先
- **Fabric の概念説明・チュートリアル・設定手順** → MS Learn MCP を使用
- **Fabric 以外の Microsoft 技術（Azure, Power BI, .NET 等）** → MS Learn MCP を使用
- 両方の MCP で得た情報を組み合わせて、正確で包括的な回答を構築する
