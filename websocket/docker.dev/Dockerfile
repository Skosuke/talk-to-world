# 開発用 Dockerfile (ホットリロード対応)
FROM golang:1.23

# 作業ディレクトリを /app に設定
WORKDIR /app

# go.mod と go.sum を先にコピーして依存関係をキャッシュ（※ビルド効率向上のため）
COPY ./cmd/go.mod ./cmd/go.sum ./
RUN go mod download

# ソースコード全体をコピー
COPY ./cmd/ ./

# Air をインストール
RUN go install github.com/air-verse/air@latest

# コンテナ外（ホスト）からソースコードの変更を反映させるために volume マウント予定のディレクトリと同じ場所
# （docker-compose.yml では、./websocket/cmd を /app にマウントすることを想定）

EXPOSE 8080

# CMD は Air を起動
CMD ["air"]
