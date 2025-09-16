# ベースとなる軽量なPythonの公式イメージを選択
FROM python:3.11-slim

# gitコマンドをインストール (pipでのGitHubリポジトリからのインストールに必要)
RUN apt-get update && apt-get install -y git && apt-get clean && rm -rf /var/lib/apt/lists/*

# zotero-mcpをpipコマンドでGitHubから直接インストール
RUN pip install --no-cache-dir "git+https://github.com/54yyyu/zotero-mcp.git"

# コンテナ内の作業ディレクトリを指定
WORKDIR /app

# 設定ディレクトリとデータディレクトリを作成
RUN mkdir -p /root/.config/zotero-mcp /root/.local/share/zotero-mcp

# curlとsocat（プロキシ用）をインストール
RUN apt-get update && apt-get install -y curl socat && apt-get clean && rm -rf /var/lib/apt/lists/*

# ヘルスチェック用のスクリプトを追加（プロセス存在確認）
RUN echo '#!/bin/bash\npgrep -f "zotero-mcp serve" > /dev/null || exit 1' > /usr/local/bin/healthcheck.sh && \
    chmod +x /usr/local/bin/healthcheck.sh

# localhost:23119をCaddyプロキシにリダイレクトするスタートアップスクリプト
RUN echo '#!/bin/bash\n# Start socat proxy in background to redirect localhost:23119 to caddy:9998\nsocat TCP-LISTEN:23119,fork,reuseaddr TCP:caddy:9998 &\n# Execute the original command\nexec "$@"' > /usr/local/bin/entrypoint.sh && \
    chmod +x /usr/local/bin/entrypoint.sh

# ヘルスチェックを設定
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD /usr/local/bin/healthcheck.sh

# エントリーポイントを設定
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]

# コンテナ起動時に実行されるメインコマンド
# 修正: streamable-httpトランスポートでホストとポートを指定
CMD ["zotero-mcp", "serve", "--transport", "streamable-http", "--host", "0.0.0.0", "--port", "9999"]
