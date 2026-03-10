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
| `publicapis_bestpractices_itemdefinition_get` | Fabric アイテムの JSON スキーマ定義を取得する（一部ワークロードのみ。未提供なら 404） |

### OneLake 関連（実行系）

OneLake ツールは、Fabric 環境に直接接続して操作する場合に使用する:

- `onelake_workspace_list` — ワークスペース一覧
- `onelake_item_list` / `onelake_item_create` — アイテムの管理
- `onelake_file_list` / `onelake_upload_file` / `onelake_download_file` — ファイル操作
- `onelake_table_list` / `onelake_table_get` — テーブル操作

## ツール利用の判断基準

### 調査系（API 仕様の参照）
- 「Fabric でどんなワークロードが使えるか」→ `publicapis_list`
- 「Lakehouse の作り方」→ `publicapis_get` + `publicapis_bestpractices_examples_get` + `publicapis_bestpractices_itemdefinition_get`（取得できる場合のみ）
- 「SemanticModel / DataAgent の作り方」→ `publicapis_get` + `publicapis_bestpractices_examples_get`
- 「API のベストプラクティス（認証・リトライ・ページング）」→ `publicapis_bestpractices_get`
- 「プラットフォーム共通の操作（ワークスペース・キャパシティ管理）」→ `publicapis_platform_get`

### 実行系（Fabric 環境の直接操作）
- ワークスペース確認 → `onelake_workspace_list`
- Lakehouse 作成 → REST API `POST /v1/workspaces/{id}/lakehouses`（MCP の `onelake_item_create` はワークスペース認識のタイミング問題で 400 になることがある）
- Semantic Model 作成 → REST API `POST /v1/workspaces/{id}/semanticModels`（定義付き）
- Data Agent 作成 → REST API `POST /v1/workspaces/{id}/DataAgents`（定義付き）
- CSV データのアップロード → `onelake_upload_file`
- ファイル一覧の確認 → `onelake_file_list`
- テーブル一覧の確認 → `onelake_table_list`

## 必須ルール

1. **調査完了後に実行**: 実行系ツール（`onelake_*`）は、調査系ツールで仕様を確認した後にのみ使用する
2. **`publicapis_list` は必須**: デモ構築時は最初に必ず `publicapis_list` を実行し、ワークロード名・item-type の最新値を取得する
3. **ワークロード差異を前提にする**: `publicapis_bestpractices_itemdefinition_get` が 404 の場合はエラーとして扱わず、`publicapis_get` と `publicapis_bestpractices_examples_get` にフォールバックする
4. **固定値の禁止**: ワークロード名や item-type をハードコードしない。必ず MCP ツールの結果に基づく値を使う

## MCP ツールの制約

以下の操作は MCP ツールでは実行 **できない**。必要に応じてターミナルで Fabric REST API を直接呼び出すこと:

| 操作 | MCP ツール | 代替手段 |
|---|---|---|
| ワークスペース作成 | ❌ なし（`onelake_workspace_list` は一覧のみ） | REST API `POST /v1/workspaces` |
| ワークスペースへの容量割り当て | ❌ なし | REST API `POST /v1/workspaces/{id}/assignToCapacity` |
| Lakehouse 作成 | ⚠ `onelake_item_create` はワークスペース認識のタイミングで 400 になることがある | REST API `POST /v1/workspaces/{id}/lakehouses`（推奨） |
| CSV → Delta テーブル変換 | ❌ なし | REST API `Tables_LoadTable`（LRO） |
| `publicapis_bestpractices_itemdefinition_get` の全 workload 対応 | ❌ 一部 workload は 404 | `publicapis_get` + `publicapis_bestpractices_examples_get` |
| Semantic Model を定義付きで作成 | ❌ `onelake_item_create` に `definition` パラメータなし | REST API `POST /v1/workspaces/{id}/semanticModels`（definition.pbism + model.bim 必須） |
| Semantic Model 定義の更新 | ❌ なし | REST API `updateDefinition`（LRO）— このデモテンプレートでは使わない。定義付きで新規作成する |
| Data Agent を定義付きで作成 | ❌ `onelake_item_create` に `definition` パラメータなし | REST API `POST /v1/workspaces/{id}/DataAgents`（definition 付き） |

> **Semantic Model は空では作成できない。** `definition.pbism`（最小構成: version + $schema のみ）と `model.bim`（TMSL 形式の JSON）が必須。
> **model.bim は TMSL 形式**（単一 JSON ファイル）を使う。TMDL 形式（複数ファイル）は使わない。
> `definition.pbism` に `datasetReference` や `connections` を含めると `Workload_FailedToParseFile` エラーになる。
> **model.bim の構築**: PowerShell ハッシュテーブルで構築し `ConvertTo-Json -Depth 10` で JSON に変換する。`-Depth` が浅いとネスト構造が切り捨てられてエラーになる。
> `publicapis_bestpractices_itemdefinition_get` は `semanticModel` で 404 になることがある。その場合は `publicapis_get(semanticModel)` の definitions / examples を正とする。

> REST API 呼び出し時のトークン取得: `az account get-access-token --resource https://api.fabric.microsoft.com`

> **SQL Analytics Endpoint の FQDN 取得**: `GET /v1/workspaces/{id}/lakehouses/{id}` → `properties.sqlEndpointProperties.connectionString` で取得する。Power BI API での検索は不要。`provisioningStatus` が `Success` になるまでポーリングすること。

> **新規ワークスペースには Fabric 容量の割り当てが必須。** 割り当て前に Lakehouse 等を作成すると `FeatureNotAvailable` エラーになる。

> **Data Agent はポータルではなく REST API で定義付きで作成する。** `Files/Config/data_agent.json`（`$schema: "2.1.0"`）+ `Files/Config/draft/stage_config.json`（AI instructions）+ `Files/Config/draft/{type}-{name}/datasource.json`（データソース設定）を Base64 エンコードして definition.parts に含める。
> **Data Agent のデータソースには必ず Semantic Model（type: `semantic_model`）を使う。** `lakehouse-tables` は絶対に使わない（メジャー・リレーションシップが無視され回答精度が低下するため）。
> `datasource.json` には `artifactId`（Semantic Model の ID）、`workspaceId`、`type: "semantic_model"` を必ず指定する。
> 参考: [Data Agent definition](https://learn.microsoft.com/rest/api/fabric/articles/item-management/definitions/data-agent-definition)

デモ構築時は **調査系で仕様を確認 → MCP 実行系で構築 → 不足分は REST API で補完** する流れとする。全アイテムの作成・設定を GHCP 内で完結させ、ポータルでの手動設定は不要にする。
