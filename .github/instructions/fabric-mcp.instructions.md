---
name: 'Fabric MCP ツール利用ガイド'
description: 'Microsoft Fabric MCP サーバーのツールを適切に使い分けるためのルール'
applyTo: '**'
---

# Fabric MCP サーバーの使い方

Microsoft Fabric の API 仕様、アイテム定義、ベストプラクティスに関する質問には、`fabric-mcp-server` のツールを使用すること。

## 利用可能なツール

### API 仕様関連

| ツール名 | 用途 |
|---|---|
| `publicapis_list` | 利用可能な Fabric ワークロードタイプの一覧を取得する（最初に呼ぶ） |
| `publicapis_get` | 特定のワークロードの OpenAPI/Swagger 仕様を取得する |
| `publicapis_platform_get` | Fabric プラットフォーム共通の API 仕様を取得する |

### ベストプラクティス・定義関連

| ツール名 | 用途 |
|---|---|
| `publicapis_bestpractices_get` | 特定トピックのベストプラクティスやガイダンスを取得する |
| `publicapis_bestpractices_examples_get` | 特定ワークロードの API リクエスト/レスポンス例を取得する |
| `publicapis_bestpractices_itemdefinition_get` | Fabric アイテムの JSON スキーマ定義を取得する |

### OneLake 関連

OneLake ツールは、Fabric 環境に直接接続して操作する場合に使用する:

- `onelake_workspace_list` — ワークスペース一覧
- `onelake_item_list` / `onelake_item_create` — アイテムの管理
- `onelake_file_list` / `onelake_upload_file` / `onelake_download_file` — ファイル操作
- `onelake_table_list` / `onelake_table_get` — テーブル操作

## ツール利用の判断基準

- 「Fabric でどんなワークロードが使えるか」→ `publicapis_list`
- 「Notebook / Lakehouse / Pipeline の作り方」→ `publicapis_get` + `publicapis_bestpractices_itemdefinition_get`
- 「API のベストプラクティス（認証・リトライ・ページング）」→ `publicapis_bestpractices_get`
- 「API リクエストの具体例」→ `publicapis_bestpractices_examples_get`
- 「プラットフォーム共通の操作（ワークスペース・キャパシティ管理）」→ `publicapis_platform_get`
