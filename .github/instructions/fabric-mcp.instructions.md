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
| `publicapis_list` | 利用可能な Fabric ワークロードタイプの一覧を取得する（**最初に必ず呼ぶ**） |
| `publicapis_get` | 特定のワークロードの OpenAPI/Swagger 仕様を取得する |
| `publicapis_platform_get` | Fabric プラットフォーム共通の API 仕様を取得する |

### ベストプラクティス・定義関連

| ツール名 | 用途 |
|---|---|
| `publicapis_bestpractices_get` | 特定トピックのベストプラクティスやガイダンスを取得する |
| `publicapis_bestpractices_examples_get` | 特定ワークロードの API リクエスト/レスポンス例を取得する |
| `publicapis_bestpractices_itemdefinition_get` | Fabric アイテムの JSON スキーマ定義を取得する |

### OneLake 関連（実行系）

OneLake ツールは、Fabric 環境に直接接続して操作する場合に使用する:

- `onelake_workspace_list` — ワークスペース一覧
- `onelake_item_list` / `onelake_item_create` — アイテムの管理
- `onelake_file_list` / `onelake_upload_file` / `onelake_download_file` — ファイル操作
- `onelake_table_list` / `onelake_table_get` — テーブル操作

## ツール利用の判断基準

### 調査系（API 仕様の参照）
- 「Fabric でどんなワークロードが使えるか」→ `publicapis_list`
- 「Lakehouse / SemanticModel / DataAgent の作り方」→ `publicapis_get` + `publicapis_bestpractices_itemdefinition_get`
- 「API のベストプラクティス（認証・リトライ・ページング）」→ `publicapis_bestpractices_get`
- 「API リクエストの具体例」→ `publicapis_bestpractices_examples_get`
- 「プラットフォーム共通の操作（ワークスペース・キャパシティ管理）」→ `publicapis_platform_get`

### 実行系（Fabric 環境の直接操作）
- ワークスペース確認 → `onelake_workspace_list`
- Lakehouse / SemanticModel / DataAgent の作成 → `onelake_item_create`
- CSV データのアップロード → `onelake_upload_file`
- ファイル一覧の確認 → `onelake_file_list`
- テーブル一覧の確認 → `onelake_table_list`

## 必須ルール

1. **調査完了後に実行**: 実行系ツール（`onelake_*`）は、調査系ツールで仕様を確認した後にのみ使用する
2. **`publicapis_list` は必須**: デモ構築時は最初に必ず `publicapis_list` を実行し、ワークロード名・item-type の最新値を取得する
3. **Data Agent 作成前の事前確認**: `publicapis_get` と `publicapis_bestpractices_itemdefinition_get` で Data Agent の最新仕様を必ず確認する
4. **固定値の禁止**: ワークロード名や item-type をハードコードしない。必ず MCP ツールの結果に基づく値を使う

## MCP ツールの制約

以下の操作は MCP ツールでは実行 **できない**。必要に応じてターミナルで Fabric REST API を直接呼び出すこと:

| 操作 | MCP ツール | 代替手段 |
|---|---|---|
| ワークスペース作成 | ❌ なし（`onelake_workspace_list` は一覧のみ） | REST API `POST /v1/workspaces` |
| CSV → Delta テーブル変換 | ❌ なし | REST API `Tables_LoadTable` |
| Semantic Model に TMDL 定義を適用 | ❌ `onelake_item_create` に `definition` パラメータなし | REST API `updateDefinition` |
| Data Agent のデータソース設定 | ❌ なし | Fabric ポータルで手動設定 |

> REST API 呼び出し時のトークン取得: `az account get-access-token --resource https://api.fabric.microsoft.com`

デモ構築時は **調査系で仕様を確認 → MCP 実行系で構築 → 不足分は REST API で補完** する流れとする。
