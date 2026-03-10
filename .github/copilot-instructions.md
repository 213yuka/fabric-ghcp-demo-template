# Fabric GHCP デモテンプレート

GitHub Copilot (Agent モード) + Fabric MCP サーバーで、Fabric のデモ環境を **GHCP 内からすべて構築** するテンプレート。

## 基本方針

- 応答は **日本語** で行う
- Fabric の情報は **Fabric MCP サーバー** のツールで取得する（推測しない）
- Fabric 環境の構築は **MCP OneLake ツールで GHCP 内から直接実行** する
- セマンティックモデルは **スタースキーマ** で設計する
- Microsoft ドキュメントは **MS Learn MCP サーバー** で検索する

## MCP サーバー

| サーバー名 | 用途 |
|---|---|
| `fabric-mcp-server` | API 仕様の参照 + OneLake ツールで Fabric 環境を直接操作 |
| `microsoft-learn` | Microsoft 公式ドキュメントの検索・取得 |

## デモの標準構成

| ワークロード | 用途 |
|---|---|
| Lakehouse | データ保存（ファクト＋ディメンション） |
| Notebook | ETL（CSV → Delta テーブル） |
| Semantic Model | スタースキーマの分析モデル |
| Data Agent | 自然言語でデータに質問 |

## 使い方

1. `/fabric-demo-create` を実行して 3 つの質問に答える
2. Copilot が設計書 + サンプルデータを生成し、MCP ツールで Fabric に直接デプロイする
