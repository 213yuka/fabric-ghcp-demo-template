# Fabric GHCP デモテンプレート

GitHub Copilot (Agent モード) + Fabric MCP サーバーで、Fabric のデモ環境を構築するテンプレート。

## 基本方針

- 応答は **日本語** で行う
- Fabric の情報は **Fabric MCP サーバー** のツールで取得する（推測しない）
- Fabric 環境の構築は **MCP + Fabric REST API で GHCP 内から直接実行** する
- **Fabric ポータルを一切開かずに完結** させる
- セマンティックモデルは **スタースキーマ** で設計する
- CSV データは **変換済み（スタースキーマ形式）** で生成する（ETL 不要）
- Microsoft ドキュメントは **MS Learn MCP サーバー** で検索する
- MCP で参照した公式資料は **タイトル・URL・要点** を回答に残す

## MCP サーバー

| サーバー名 | 用途 |
|---|---|
| `fabric-mcp-server` | API 仕様の参照 + OneLake ツールで Fabric 環境を直接操作 |
| `microsoft-learn` | Microsoft 公式ドキュメントの検索・取得 |

## デモの標準構成

| ワークロード | 用途 |
|---|---|
| Lakehouse | データ保存（ファクト＋ディメンション） |
| SQL Analytics Endpoint | Delta テーブルへの SQL アクセス（Lakehouse と同時に自動作成） |
| Semantic Model | スタースキーマの分析モデル（Direct Lake） |
| Data Agent | 自然言語でデータに質問 |

## 使い方

1. `/fabric-demo-create` を実行して質問に答える（追加データの添付も可能）
2. Copilot が設計書 + 変換済みサンプルデータを生成し、MCP ツール + Fabric REST API で Fabric に自動デプロイする
3. 完了 — Fabric ポータルを開く必要なし
