---
name: 'Fabric-Explore'
description: 'Microsoft Fabric のワークロード、API、機能を調査する'
agent: agent
argument-hint: '調査したい Fabric の機能やワークロードを入力してください'
tools:
  - fabric-mcp-server/*
  - microsoft-learn/*
---

# Fabric の機能調査

ユーザーが知りたい Fabric の機能やワークロードについて、MCP ツールを使って正確な情報を提供してください。

## 調査の進め方

1. **ワークロード一覧の確認**: #tool:publicapis_list で利用可能なワークロードを確認する
2. **API 仕様の取得**: #tool:publicapis_get で特定ワークロードの OpenAPI 仕様を取得する
3. **プラットフォーム API の確認**: #tool:publicapis_platform_get でプラットフォーム共通の操作を確認する
4. **公式ドキュメントの補完**: #tool:microsoft_docs_search で概念説明やチュートリアルを検索する

## 回答の形式

- 調査結果は **表形式** や **箇条書き** で見やすくまとめる
- API エンドポイントは **メソッド + パス + 説明** を記載する
- 具体的なコード例がある場合は提示する
- 情報ソース（MCP ツール名）を明記する
