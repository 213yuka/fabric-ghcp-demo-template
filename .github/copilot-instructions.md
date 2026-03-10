# Fabric GHCP デモテンプレート

このワークスペースは **GitHub Copilot (Agent モード) + MCP サーバー** を使って、Microsoft Fabric のデモ環境を構築するためのテンプレートです。

## 基本方針

- 応答は **日本語** で行う
- Microsoft Fabric に関する情報は、必ず **Fabric MCP サーバー** のツールを使って最新の公式情報を取得する
- Microsoft 技術全般のドキュメントは **MS Learn MCP サーバー** のツールを使って参照する
- 推測で回答せず、MCP ツールで得た情報に基づいて回答する

## 利用可能な MCP サーバー

| サーバー名 | 種類 | 用途 |
|---|---|---|
| `fabric-mcp-server` | ローカル (npx) | Fabric の API 仕様、アイテム定義、ベストプラクティスの取得 |
| `microsoft-learn` | リモート (HTTP) | Microsoft 公式ドキュメントの検索・取得、コードサンプル検索 |

## 前提条件

このテンプレートを使うには以下が必要です:

1. **Visual Studio Code** (最新版)
2. **GitHub Copilot** 拡張機能 (Copilot Chat 含む)
3. **Node.js 20 LTS 以降** — Fabric MCP サーバーの実行に必要
   - `node --version` で確認
   - `npx --version` で確認
4. **GitHub Copilot のライセンス** (Individual, Business, Enterprise いずれか)

## デモ構築の進め方

1. Agent モードでチャットを開く
2. `/fabric-demo-create` プロンプトを使い、シナリオ（製造・教育・小売など）を指定する
3. Copilot が MCP ツールを使って Fabric の API 仕様やベストプラクティスを取得しながら、デモ設計を支援する
4. 必要に応じて `/fabric-explore` や `/fabric-best-practices` で追加調査する
