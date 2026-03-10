---
name: 'Fabric ベストプラクティス'
description: 'Microsoft Fabric API のベストプラクティスやパターンを取得する'
agent: agent
argument-hint: '認証、リトライ、ページング、エラー処理など知りたいトピックを入力'
tools:
  - fabric-mcp-server/*
  - microsoft-learn/*
---

# Fabric ベストプラクティスの取得

Fabric API を使ったアプリケーション開発のベストプラクティスを、MCP ツールで取得して提供してください。

## よくあるトピック

- **認証**: Fabric API への認証方法、トークン管理
- **リトライ / スロットリング**: API レート制限時のリトライ戦略、バックオフパターン
- **ページング**: 大量データ取得時のページネーション実装
- **エラー処理**: エラーレスポンスの解釈と対処法
- **長時間実行操作**: 非同期オペレーションのポーリングパターン

## 調査手順

1. #tool:publicapis_bestpractices_get でトピック別のベストプラクティスを取得する
2. #tool:publicapis_bestpractices_examples_get で具体的なリクエスト/レスポンス例を取得する
3. 必要に応じて #tool:microsoft_docs_search で関連ドキュメントを補完する

## 回答の形式

- **推奨パターン** と **アンチパターン** を明確に区別する
- 実装例はコードブロックで示す
- API の具体的なレスポンス例を含める
