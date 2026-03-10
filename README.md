# Fabric GHCP デモテンプレート

GitHub Copilot + MCP サーバーで、Fabric のデモ環境を **GHCP 内からすべて構築** するテンプレート。

---

## できること

- Lakehouse 作成・サンプルデータ投入
- Notebook 作成（CSV → Delta テーブル変換）
- Semantic Model 作成（スタースキーマ）
- Data Agent 作成（自然言語でデータに質問）

**すべて Copilot チャット内で完結。Fabric ポータルを開く必要なし。**

---

## クイックスタート

### 1. 前提条件

| 項目 | 要件 |
|---|---|
| VS Code | 最新版 |
| GitHub Copilot | 拡張機能インストール済み |
| Node.js | 20 LTS 以降 |

### 2. 始め方

1. このリポジトリをクローンして VS Code で開く
2. MCP サーバーの接続を確認（`Ctrl+Shift+P` → `MCP: サーバーの一覧`）
   - ✅ `fabric-mcp-server`
   - ✅ `microsoft-learn`
3. Copilot Chat を Agent モードで開く
4. 以下を入力:

```
/fabric-demo-create
```

5. **3つの質問** に答える:
   - 🏭 業種・シナリオ（例: 製造業の品質管理）
   - 🎯 目的・対象者（例: 経営層向けデモ）
   - 📐 規模（ミニマル / 標準 / フル）

6. Copilot が自動で:
   - 設計書 + サンプルデータを生成
   - **MCP ツールで Fabric に直接デプロイ**

### 3. 生成されるもの

```
ローカル:
  demo-output/
  ├── demo-spec.md       # 設計書
  ├── data/              # CSV（スタースキーマ: fact_ + dim_）
  └── notebooks/         # ETL コード

Fabric 環境:
  [ワークスペース]/
  ├── Lakehouse          # データ保存
  ├── Notebook           # ETL 処理
  ├── Semantic Model     # スタースキーマ分析モデル
  └── Data Agent         # 自然言語クエリ
```

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
├── copilot-instructions.md        # 基本ルール
├── instructions/                  # MCP ツールの使い分けガイド
├── prompts/                       # /コマンド定義
└── skills/fabric-demo-builder/    # デモ構築の専門知識
.vscode/
├── mcp.json                       # MCP サーバー接続設定
├── settings.json                  # ワークスペース設定
└── extensions.json                # 推奨拡張機能
```

## ライセンス

MIT
