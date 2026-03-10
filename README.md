# Fabric GHCP デモテンプレート

GitHub Copilot (GHCP) + MCP サーバーで、Fabric の Data Agent デモ環境を構築するテンプレート。

> **🚀 GHCP チャット内で、Lakehouse 作成からデータ投入・テーブル変換まで自動実行。**
> Semantic Model や Data Agent のリソース作成も GHCP 内で完了します。

---

## できること

- 新規ワークスペース作成
- Lakehouse 作成・変換済みサンプルデータ投入
- CSV → Delta テーブル変換（REST API で自動）
- Semantic Model 作成（TMSL 定義付き: テーブル・リレーションシップ・メジャー）
- Data Agent 作成（自然言語でデータに質問）

**主要な構築作業は GHCP チャット内で完結。ポータルでの手動設定は Approved for Copilot のみです。**

---

## 前提条件

### 環境

| 項目 | 要件 |
|---|---|
| VS Code | 最新版 |
| GitHub Copilot | 拡張機能インストール済み（Agent モード対応） |
| Azure CLI | `az` コマンドが利用可能なこと（Fabric REST API のトークン取得に使用） |
| Node.js | 20 LTS 以降（Fabric MCP サーバーの実行に必要） |

**推奨拡張機能（オプション）:**
- [Microsoft Fabric](https://marketplace.visualstudio.com/items?itemName=Microsoft.fabric) — Fabric アイテムの確認・管理に便利

> **ターミナルツールについて:** Agent モードでのターミナル実行は `chat.agent.enabled`（デフォルト `true`）で有効化されています。フェーズ 0 の事前チェックとフェーズ 4 の REST API 呼び出しに必要です。

**ターミナルが使えない場合は、以下を確認してください:**
1. VS Code で **このリポジトリのルート** を開いているか（サブフォルダとして開くと `.vscode/settings.json` が認識されない）
2. GitHub Copilot Chat 拡張機能が最新版か
3. Agent モードが有効か — `settings.json` に以下が入っていることを確認:

```json
{
   "chat.agent.enabled": true
}
```

### ターミナルコマンドの自動承認について

このテンプレートでは `chat.tools.terminal.autoApprove` で **安全な読み取り系コマンドのみ** を自動承認しています:

```jsonc
"chat.tools.terminal.autoApprove": {
    "az account get-access-token": true,  // Fabric REST API のトークン取得
    "Get-ChildItem": true,                // ファイル一覧
    "Test-Path": true,                    // ファイル存在確認
    "Get-Content": true,                  // ファイル読み取り
    "Get-Location": true                  // カレントディレクトリ確認
}
```

リソース作成・変更を伴うコマンド（`Invoke-WebRequest`, `Invoke-RestMethod` 等）は自動承認に **含めていません**。実行時に確認ダイアログが表示されるので、内容を確認してから承認してください。

> **⚠️ コマンド追加時の注意**
>
> 自動承認するコマンドを追加する場合は、ファイルを壊さない検索・参照系のコマンドに限定してください。`rm`, `Remove-Item`, `del` など **削除系コマンドは追加しないでください**。最悪の場合、重要なファイルが消えます。
>
> すべてのツール呼び出しを自動承認したい場合は `"chat.tools.global.autoApprove": true` を設定できますが、セキュリティ保護が無効化されるため推奨しません。

### Fabric 権限

| 項目 | 要件 |
|---|---|
| Fabric ワークスペース | **共同作成者** 以上のロール |
| Fabric 容量 | F2 以上（または Power BI Premium Per User） |
| Fabric 容量の状態 | **起動済み（アクティブ）**であること（一時停止中はデプロイ失敗する） |
| Microsoft Entra ID | Fabric MCP サーバーの認証に必要 |

### MCP サーバー

このテンプレートは 2 つの MCP サーバーを使用する（`.vscode/mcp.json` で設定済み）:

| サーバー名 | プロトコル | 用途 |
|---|---|---|
| `fabric-mcp-server` | stdio (npx) | Fabric API 仕様の参照 + OneLake 直接操作 |
| `microsoft-learn` | HTTP | Microsoft 公式ドキュメントの検索・取得 |

**セットアップ確認**:
1. VS Code でこのリポジトリを開く
2. `Ctrl+Shift+P` → `MCP: サーバーの一覧` を実行
3. `fabric-mcp-server` と `microsoft-learn` が一覧に表示されていることを確認
4. 各サーバーをクリック → **Start** で起動（初回はオンデマンド起動も可）
5. `fabric-mcp-server` の初回起動時は Microsoft Entra ID の認証が求められる

---

## クイックスタート

### フェーズ概要

`/fabric-demo-create` は **5 フェーズ** でデモ環境を構築します:

| フェーズ | 内容 | 実行者 |
|---|---|---|
| **0. 事前チェック** | Azure CLI・Fabric 容量・MCP 接続を確認 | GHCP が自動確認 |
| **1. ヒアリング** | 業種・目的・追加データを確認 | ユーザーが回答 |
| **2. MCP 調査** | Fabric API 仕様・利用可能な定義/サンプルを取得 | GHCP が自動実行 |
| **3. 設計書 + データ生成** | スタースキーマ設計・変換済み CSV をローカルに出力 | GHCP が自動実行 |
| **4. Fabric デプロイ** | Lakehouse・Semantic Model・Data Agent を作成・データ投入 | GHCP が MCP + REST API で自動実行 |

実行中は、各フェーズの開始時に現在位置を表示します。

- `現在の進捗: フェーズ 0/5 - 事前チェック`
- `現在の進捗: フェーズ 1/5 - ヒアリング`
- `現在の進捗: フェーズ 2/5 - MCP 調査`
- `現在の進捗: フェーズ 3/5 - 設計書＋データ生成`
- `現在の進捗: フェーズ 4/5 - Fabric デプロイ`

### 実行フロー

```
/fabric-demo-create
   ↓
① 事前チェック — Azure CLI・Fabric 容量・MCP 接続
   ↓
② ヒアリング — 業種・目的 + 追加データ（任意）
   ↓
③ MCP で Fabric API 仕様を調査
   ↓
④ 設計書 + 変換済み CSV を生成（ローカル）
   ↓
⑤ MCP ツール + Fabric REST API で自動デプロイ
   ├── 新規ワークスペース作成（+ 容量割り当て）
   ├── Lakehouse 作成（REST API）
   ├── CSV アップロード（MCP）
   ├── CSV → Delta 変換（REST API + LRO ポーリング）
   ├── Semantic Model 作成（REST API、定義付き）
   └── Data Agent 作成（REST API、定義付き: データソース・AI インストラクション設定済み）
   ↓
⑦ 動作検証（Fabric ポータルで代表質問を確認）
```

### 始め方

> ❗ **重要**: このリポジトリを **VS Code のワークスペースルート** として開いてください。
> サブフォルダとして開くと、`.github/prompts` や `.vscode/settings.json` が認識されず `/fabric-demo-create` が使えません。

1. このリポジトリをクローンして VS Code で **リポジトリルートを直接** 開く
2. MCP サーバーの接続を確認（上記「セットアップ確認」参照）
3. GitHub Copilot Chat を **Agent モード** で開く
4. 以下を入力:

```
/fabric-demo-create
```

5. **質問** に答える:
   - 🏭 業種・シナリオ（例: 製造業の品質管理）
   - 🎯 目的・対象者（例: 経営層向けデモ）
   - 📎 追加データ（任意: Excel や CSV を添付可能）

   **入力例:**
   ```
   1. 小売の売上分析
   2. 経営層向けデモ
   3. なし
   ```

6. GHCP が自動で:
   - 設計書 + 変換済みサンプルデータを生成
   - **MCP ツール + Fabric REST API で Fabric にデプロイ**
   - Lakehouse 作成、CSV アップロード、CSV → Delta 変換、Semantic Model・Data Agent 作成まで自動実行

7. 動作検証:
   - Fabric ポータルで Data Agent を開き、代表質問セットで動作を確認

### 生成されるもの

```
ローカル:
  demo-output/
  ├── demo-spec.md       # 設計書
  ├── usage-guide.md     # 活用ガイド（質問例・デモシナリオ・トラブルシューティング）
  └── data/              # CSV（スタースキーマ: fact_ + dim_、変換済み）

Fabric 環境:
  [ワークスペース]/
  ├── Lakehouse          # データ保存 + SQL Analytics Endpoint (自動)
  ├── Semantic Model     # スタースキーマ分析モデル (Direct Lake)
  └── Data Agent         # 自然言語クエリ
```

---

## デモの標準構成

| ワークロード | 用途 | 作成方法 |
|---|---|---|
| Lakehouse | データ保存（ファクト＋ディメンション） | REST API で自動作成 |
| SQL Analytics Endpoint | Delta テーブルへの SQL アクセス | Lakehouse と同時に自動作成 |
| Semantic Model | スタースキーマの分析モデル（Direct Lake） | REST API で定義付き作成（自動） |
| Data Agent | 自然言語でデータに質問 | REST API で定義付き作成（データソース・AIインストラクション自動設定） |

---

## 制約事項

| 制約 | 詳細 |
|---|---|
| Semantic Model の設定 | REST API で定義付き（definition.pbism + model.bim）で作成。テーブル・リレーションシップ・メジャーが自動設定済み |
| Data Agent のデータソース | REST API で定義付き（datasource.json + stage_config.json）で作成。データソース・AIインストラクションが自動設定済み |
| Data Agent の対応範囲 | Semantic Model または Lakehouse に対する自然言語クエリのみ（現時点ではリアルタイムデータ非対応） |
| ワークロード名・item-type | Fabric API の仕様変更に左右されるため、構築前に必ず `publicapis_list` で確認する |

---

## その他のコマンド

| コマンド | 用途 |
|---|---|
| `/fabric-explore` | Fabric の機能・API を調べる |
| `/fabric-best-practices` | ベストプラクティスを取得 |

---

## フォルダ構成

```
.github/
├── copilot-instructions.md        # 基本ルール（常に読み込まれる）
├── instructions/                  # MCP ツールの使い分けガイド
│   ├── fabric-mcp.instructions.md #   Fabric MCP ツール選択ルール
│   └── ms-learn-mcp.instructions.md # MS Learn 参照ルール
├── prompts/                       # /コマンド定義
│   ├── fabric-demo-create.prompt.md # デモ構築の実行手順
│   ├── fabric-explore.prompt.md     # Fabric 探索
│   └── fabric-best-practices.prompt.md # ベストプラクティス取得
└── skills/fabric-demo-builder/    # デモ構築の専門知識
    ├── SKILL.md                   #   再利用可能な実行パターン
    └── examples/                  #   テンプレート・シナリオ例
.vscode/
├── mcp.json                       # MCP サーバー接続設定
├── settings.json                  # ワークスペース設定
└── extensions.json                # 推奨拡張機能
```

### 各ファイルの責務

| ファイル | 責務 | 含めるべき情報 |
|---|---|---|
| `copilot-instructions.md` | 基本方針 | 言語、MCP サーバー、標準構成 |
| `fabric-mcp.instructions.md` | ツール選択ルールのみ | どのツールをいつ使うか |
| `ms-learn-mcp.instructions.md` | MS Learn 参照ルールのみ | 検索と取得の使い分け |
| `fabric-demo-create.prompt.md` | デモ構築手順 | フェーズ 1〜4 の実行フロー |
| `SKILL.md` | 再利用可能な実行パターン | スタースキーマ設計、入出力契約 |

---

## ライセンス

MIT
