# Fabric GHCP デモテンプレート

GitHub Copilot (GHCP) + MCP サーバーで、Fabric の Data Agent デモ環境を構築するテンプレート。

> **🚀 GHCP チャット内で、Lakehouse 作成からデータ投入・テーブル変換まで自動実行。**
> Semantic Model ・ Data Agent のアイテム作成も GHCP 内で完了します。

---

## できること

- ワークスペース新規作成
- Lakehouse 作成・変換済みサンプルデータ投入
- CSV → Delta テーブル変換（REST API で自動）
- Semantic Model 作成（TMDL 定義付き: テーブル・リレーションシップ・メジャー）
- Data Agent 作成（自然言語でデータに質問）

**主要な構築作業は GHCP チャット内で完結。ポータルでの設定は最小限です。**

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

> **ターミナルツールについて:** Agent モードでのターミナル実行（`github.copilot.chat.agent.runInTerminal.enabled`）は `.vscode/settings.json` で有効化済みです。フェーズ 4 の REST API 呼び出しに必要です。

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
| **2. MCP 調査** | Fabric API 仕様・アイテム定義を取得 | GHCP が自動実行 |
| **3. 設計書 + データ生成** | スタースキーマ設計・変換済み CSV をローカルに出力 | GHCP が自動実行 |
| **4. Fabric デプロイ** | Lakehouse・Semantic Model・Data Agent を作成・データ投入 | GHCP が MCP + REST API で自動実行 |

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
   ├── ワークスペース選択 or 作成（+ 容量割り当て）
   ├── Lakehouse 作成（MCP）
   ├── CSV アップロード（MCP）
   ├── CSV → Delta 変換（REST API + LRO ポーリング）
   ├── Semantic Model 作成（REST API、定義付き）
   └── Data Agent 作成（MCP）
   ↓
⑥ ポータルで Data Agent のデータソース + インストラクションを設定
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

7. Fabric ポータルで残りの設定:
   - Semantic Model → テーブル追加 + リレーションシップ設定（TMDL 自動適用が成功した場合は不要）
   - Data Agent → データソースとして Semantic Model を選択

### 生成されるもの

```
ローカル:
  demo-output/
  ├── demo-spec.md       # 設計書
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
| Lakehouse | データ保存（ファクト＋ディメンション） | MCP で自動作成 |
| SQL Analytics Endpoint | Delta テーブルへの SQL アクセス | Lakehouse と同時に自動作成 |
| Semantic Model | スタースキーマの分析モデル（Direct Lake） | MCP で作成 → ポータルで設定補完 |
| Data Agent | 自然言語でデータに質問 | MCP で作成 → ポータルでデータソース選択 |

---

## 制約事項

| 制約 | 詳細 |
|---|---|
| Semantic Model の設定 | MCP でアイテム作成後、テーブル追加・リレーションシップ・メジャーはポータルで設定（TMDL 自動適用成功時は不要） |
| Data Agent のデータソース | MCP でアイテム作成後、ポータルで Semantic Model をデータソースとして選択 |
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
