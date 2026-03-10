# Fabric GHCP デモテンプレート

GitHub Copilot + MCP サーバーで、Microsoft Fabric のデモ環境指示書を自動生成するテンプレートです。

---

## クイックスタート

### 1. 前提条件を確認

| 項目 | 要件 |
|---|---|
| VS Code | 最新版 |
| GitHub Copilot 拡張機能 | インストール済み（Copilot Chat 含む） |
| GitHub Copilot ライセンス | Individual / Business / Enterprise いずれか |
| Node.js | 20 LTS 以降 |

```bash
node --version   # v20.x.x 以上であること
npx --version    # 正常に返ること
```

### 2. ワークスペースを開く

1. このリポジトリをクローン（またはダウンロード）する
2. **VS Code でこのフォルダを開く**

### 3. MCP サーバーの接続を確認

1. `Ctrl + Shift + P` → **「MCP: サーバーの一覧」** を実行
2. 以下の 2 つが表示されていることを確認:
   - ✅ `fabric-mcp-server`
   - ✅ `microsoft-learn`
3. 停止中の場合は **「MCP: Start Server」** で起動

### 4. デモ指示書を生成する

1. **Copilot Chat を開く** (`Ctrl + Shift + I`)
2. **Agent モード** に切り替える（チャット左上のドロップダウン）
3. チャットに以下を入力して Enter:

```
/fabric-demo-create
```

4. **3 つの質問** に答える:
   - 🏭 **業種・シナリオ** — 例: `製造業の品質管理`, `教育機関の学習分析`
   - 🎯 **デモの目的・対象者** — 例: `経営層向け, Fabricの統合性をアピール`
   - 📐 **デモの規模** — 例: `標準 (30分)`
5. Copilot が Fabric MCP でワークロードや API 仕様を調査し、**デモ環境指示書** を `demo-output/` フォルダに自動生成します

### 5. 生成される成果物

```
demo-output/
├── demo-spec.md          # デモ環境指示書（アーキテクチャ・データ設計・ストーリー）
├── sample-data/          # サンプルデータファイル (CSV 等)
└── notebooks/            # Notebook サンプルコード
```

---

## その他のコマンド

| コマンド | 用途 | 例 |
|---|---|---|
| `/fabric-explore` | Fabric の機能・API を調べる | `/fabric-explore Lakehouse の API` |
| `/fabric-best-practices` | ベストプラクティスを取得 | `/fabric-best-practices 認証とリトライ` |

---

## フォルダ構成

```
fabric-ghcp-demo-template/
├── .github/
│   ├── copilot-instructions.md          # 常時適用: 基本ルール・前提条件
│   ├── instructions/
│   │   ├── fabric-mcp.instructions.md   # Fabric MCP ツールの使い分けガイド
│   │   └── ms-learn-mcp.instructions.md # MS Learn MCP ツールの使い分けガイド
│   ├── prompts/
│   │   ├── fabric-demo-create.prompt.md # /fabric-demo-create コマンド
│   │   ├── fabric-explore.prompt.md     # /fabric-explore コマンド
│   │   └── fabric-best-practices.prompt.md # /fabric-best-practices コマンド
│   └── skills/
│       └── fabric-demo-builder/
│           ├── SKILL.md                 # デモ構築の専門スキル
│           └── examples/
│               └── scenario-template.md # シナリオ記述テンプレート
├── .vscode/
│   ├── mcp.json                         # MCP サーバー接続設定
│   ├── settings.json                    # ワークスペース設定
│   └── extensions.json                  # 推奨拡張機能
└── README.md                            # このファイル
```

### 各フォルダの役割

| フォルダ / ファイル | 種類 | 説明 |
|---|---|---|
| `copilot-instructions.md` | 常時適用の指示 | すべてのチャットに自動適用される基本ルール |
| `instructions/*.instructions.md` | ファイルベースの指示 | MCP ツールの使い方を定義（`applyTo: '**'` で常時適用） |
| `prompts/*.prompt.md` | プロンプトファイル | `/` コマンドで手動呼び出しする再利用可能なタスク定義 |
| `skills/*/SKILL.md` | エージェントスキル | 必要なときだけ読み込まれるドメイン知識と手順 |
| `.vscode/mcp.json` | MCP 設定 | Fabric MCP と MS Learn MCP の接続先定義 |

## MCP サーバーについて

### Fabric MCP サーバー

- **種類**: ローカル (npx で起動)
- **パッケージ**: `@microsoft/fabric-mcp`
- **機能**: Fabric の OpenAPI 仕様、アイテム定義、ベストプラクティスの提供
- **認証**: 不要（API 仕様の参照のみ。OneLake 操作時は Azure 認証が必要）

### Microsoft Learn MCP サーバー

- **種類**: リモート (Streamable HTTP)
- **エンドポイント**: `https://learn.microsoft.com/api/mcp`
- **機能**: Microsoft 公式ドキュメントの検索・取得、コードサンプル検索
- **認証**: 不要
- **料金**: 無料

## カスタマイズ

### シナリオテンプレートの追加

`.github/skills/fabric-demo-builder/examples/` に独自のシナリオテンプレートを追加できます。

### 新しいプロンプトの追加

`.github/prompts/` に `.prompt.md` ファイルを追加すると、チャットの `/` コマンドとして利用できます。

### 指示の追加

`.github/instructions/` に `.instructions.md` ファイルを追加すると、条件に応じた指示を自動適用できます。

## ライセンス

MIT
