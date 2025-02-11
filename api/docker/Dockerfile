# ビルドステージ
FROM golang:1.23 AS builder
WORKDIR /app

# 依存ファイルのみ先にコピー
COPY ./cmd/go.mod ./cmd/go.sum ./
RUN go mod download

# ソースコード全体をコピーしてビルド
COPY ./cmd/ ./
RUN go build -o main .

# 本番用ステージ
FROM debian:stable-slim
RUN apt-get update && apt-get install -y ca-certificates && rm -rf /var/lib/apt/lists/*
WORKDIR /root/
COPY --from=builder /app/main .
EXPOSE 8080
CMD ["./main"]
