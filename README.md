# zotero-mcp-docker

[zotero-mcp](https://github.com/54yyyu/zotero-mcp)をDocker Compose環境で簡単に運用するためのプロジェクトです。

ドキュメント，コードはAIアシスタントにより生成されているため，内容の正確性については保証されません。

## 🎯 概要

このプロジェクトは、zotero-mcpをDockerコンテナ化し、Docker Composeで簡単にセットアップ・運用できるようにしたものです。zotero-mcpの機能詳細については、[元のリポジトリ](https://github.com/54yyyu/zotero-mcp)をご確認ください。

## ✨ このリポジトリの特徴

- 🐳 Docker Composeによる簡単なセットアップ
- 🔄 リバースプロキシ（Caddy）による安定した通信
- 💾 設定・データの永続化
- 🔧 コンテナ経由での簡単な管理・メンテナンス

## 🏗️ アーキテクチャ

```text
┌─────────────────────┐    ┌──────────────────┐    ┌─────────────────────┐
│   AI Assistant      │    │  zotero-mcp      │    │     Caddy           │
│ (Claude/Copilot)    │◄──►│   Container      │◄──►│  (Reverse Proxy)    │
│                     │    │   Port: 9999     │    │   Port: 9998        │
└─────────────────────┘    └──────────────────┘    └─────────────────────┘
                                                              │
                                                              ▼
                                                    ┌─────────────────────┐
                                                    │   Host OS (Windows) │
                                                    │  Zotero App         │
                                                    │  Port: 23119        │
                                                    └─────────────────────┘
```

## 🛠️ 必要環境

- Windows 10/11
- Docker Desktop for Windows
- Zotero (ローカルインストール済み、HTTPサーバー有効化済み)

## 🚀 クイックスタート

### 1. プロジェクトの起動

```powershell
# リポジトリをクローン
cd mcp

# コンテナをビルド・起動
docker-compose up --build -d

# コンテナの状態確認
docker-compose ps
```

### 2. 初期設定

```powershell
# zotero-mcpの初期設定を実行
docker-compose exec zotero_mcp_server zotero-mcp setup
```

### 3. 接続確認

```powershell
# Zotero APIが動作していることを確認
curl http://localhost:23119/api/
```

### 4. データベース構築

```powershell
# セマンティック検索用データベースを構築
docker-compose exec zotero_mcp_server zotero-mcp update-db
```

## 🔧 管理・メンテナンス

### データベース操作

```powershell
# データベース状態確認
docker-compose exec zotero_mcp_server zotero-mcp db-status

# データベース手動更新
docker-compose exec zotero_mcp_server zotero-mcp update-db

# データベース詳細情報
docker-compose exec zotero_mcp_server zotero-mcp db-inspect
```

### 設定管理

```powershell
# 設定を再実行
docker-compose exec zotero_mcp_server zotero-mcp setup

# 設定ファイル確認
docker-compose exec zotero_mcp_server cat /root/.config/zotero-mcp/config.json

# 設定情報表示
docker-compose exec zotero_mcp_server zotero-mcp setup-info
```

## 📁 プロジェクト構造

```text
mcp/
├── docker-compose.yml      # Docker構成ファイル
├── .vscode/
│   └── mcp.json           # VS Code GitHub Copilot Agent用MCP設定
├── caddy/
│   └── Caddyfile          # リバースプロキシ設定
├── zotero/
│   ├── Dockerfile         # zotero-mcpコンテナ定義
│   ├── config/           # 設定ファイル（永続化）
│   └── data/             # データベース（永続化）
└── README.md             # このファイル
```

## 🧪 動作確認

### 基本テスト

```powershell
# 1. コンテナ状態確認
docker-compose ps

# 2. MCPサーバーログ確認（エラーがないことを確認）
docker-compose logs --tail=20 zotero_mcp_server

# 3. Zotero接続確認（コンテナ内から）
docker-compose exec zotero_mcp_server curl -s http://localhost:23119/api/
```

### MCPサーバー接続確認

```powershell
# MCPサーバーの基本応答確認
curl -s http://localhost:9999/ | findstr "Not Found"

# コンテナ内からZotero API接続確認
docker-compose exec zotero_mcp_server curl -s http://localhost:23119/api/
```

## 🔧 トラブルシューティング

### よくある問題

#### 1. Zotero APIに接続できない場合

```powershell
# Zotero HTTPサーバーの確認
curl -s http://localhost:23119/api/

# Zoteroアプリケーションの設定確認
# 設定 > 一般 > その他 > HTTPサーバーを有効化 をチェック
```

#### 2. MCPサーバーが応答しない場合

```powershell
# コンテナ状態確認
docker-compose ps

# ヘルシーでない場合の詳細ログ確認
docker-compose logs --tail=50 zotero_mcp_server

# サービス再起動
docker-compose restart zotero_mcp_server
```

#### 3. データベースの問題

```powershell
# データベースの再構築
docker-compose exec zotero_mcp_server zotero-mcp update-db

# 設定ファイルの内容確認
docker-compose exec zotero_mcp_server cat /root/.config/zotero-mcp/config.json

# データベース状態確認
docker-compose exec zotero_mcp_server zotero-mcp db-status
```

### ログとデバッグ

```powershell
# リアルタイムログ監視
docker-compose logs -f

# 特定サービスのログ
docker-compose logs zotero_mcp_server
docker-compose logs caddy

# コンテナ内でのコマンド実行
docker-compose exec zotero_mcp_server bash
```

## 🔄 Docker環境のメンテナンス

### コンテナ管理

```powershell
# コンテナの再起動
docker-compose restart

# コンテナの停止・削除
docker-compose down

# イメージの再ビルド
docker-compose up --build -d
```

### データのバックアップ

```powershell
# 設定とデータのバックアップ
Copy-Item -Recurse ".\zotero\config" ".\backup\config_$(Get-Date -Format 'yyyyMMdd')"
Copy-Item -Recurse ".\zotero\data" ".\backup\data_$(Get-Date -Format 'yyyyMMdd')"
```

## 📚 関連リンク

- [zotero-mcp（元プロジェクト）](https://github.com/54yyyu/zotero-mcp) - 機能詳細・使用方法
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Caddy Web Server](https://caddyserver.com/)

## 📄 ライセンス

このプロジェクトはMITライセンスで提供されています。

---

**このプロジェクトについて**: zotero-mcpのDocker Compose環境です。zotero-mcpの機能や使用方法については[元のリポジトリ](https://github.com/54yyyu/zotero-mcp)をご確認ください。
