# Talk To World

このリポジトリは、**1 対 1 ランダムチャットアプリケーション**の全体プロジェクトです。

## 入室画面

<p align="center">
  <img width="500" alt="入室画面" src="https://github.com/user-attachments/assets/c4eefa85-9b0f-48a5-aec1-3625a55fc41c" />
</p>

## チャット画面

<p align="center">
  <img width="500" alt="チャット画面" src="https://github.com/user-attachments/assets/9c5100fe-c1a5-4c77-ba6e-c7b1d4b354dc" />
</p>

## このリポジトリの概要

API サーバー (Go / WebSocket) とアプリケーション (Laravel + Vite) の各コードは、個別の Git リポジトリとして管理され、サブモジュールとして組み込まれています。

- API サーバー (Go)： `api/cmd`
- アプリケーション (Laravel + Vite)： `app/src`

---

## 目次

- [ディレクトリ構成](#ディレクトリ構成)
- [Git サブモジュールについて](#git-サブモジュールについて)
- [docker-compose.yml の構成](#docker-composeyml-の構成)
- [動作環境](#動作環境)
- [初期セットアップ](#初期セットアップ)
- [起動方法](#起動方法)
- [開発時の注意](#開発時の注意)

---

## ディレクトリ構成

```
.
├── api
│   ├── cmd                // API サーバー (Go) のコード（サブモジュール）
│   ├── docker
│   │   └── Dockerfile     // 本番用 Dockerfile（例）
│   └── docker.dev
│       └── Dockerfile     // 開発用 Dockerfile（例）
├── app
│   ├── docker
│   │   ├── nginx
│   │   │   └── default.conf // Nginx の設定ファイル
│   │   └── Dockerfile       // Laravel 側の Dockerfile
│   ├── src                // Laravel プロジェクト & Vite 構成（サブモジュール）
├── .DS_Store              // macOS 環境で生成された隠しファイル（不要なら削除）
└── docker-compose.yml     // 複数コンテナを統合して起動する設定ファイル
```

---

## Git サブモジュールについて

このリポジトリでは、以下のサブモジュールが利用されています。

- **API サーバー**： `api/cmd`
- **アプリケーション**： `app/src`

**クローン時の注意点**

リポジトリをクローンする際は、サブモジュールも同時に取得するため、以下のコマンドを利用してください。

```bash
git clone --recurse-submodules https://github.com/Skosuke/talk-to-world.git
```

もし既にクローン済みの場合は、以下のコマンドでサブモジュールを初期化および更新できます。

```bash
git submodule update --init --recursive
```

---

## docker-compose.yml の構成

以下は、本プロジェクトで利用している `docker-compose.yml` の内容です。

```yaml
version: '3.8'

services:
  app:
    build:
      context: ./app
      dockerfile: docker/Dockerfile
    container_name: app
    volumes:
      - ./app/src:/var/www/html
    depends_on:
      - mysql
    ports:
      - '3000:3000' # Vite の開発サーバー用ホスト側ポート
    networks:
      - talk-to-world-network

  api:
    build:
      context: ./api
      dockerfile: docker.dev/Dockerfile
    container_name: api
    volumes:
      - ./api/cmd:/app
    ports:
      - '8080:8080'
    depends_on:
      - mysql
    networks:
      - talk-to-world-network

  mysql:
    image: mysql:8.0
    container_name: mysql
    environment:
      MYSQL_ROOT_PASSWORD: your_root_password
      MYSQL_DATABASE: your_database_name
      MYSQL_USER: your_user
      MYSQL_PASSWORD: your_password
    ports:
      - '3306:3306'
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - talk-to-world-network

  nginx:
    image: nginx:latest
    container_name: nginx
    ports:
      - '8090:80'
    volumes:
      - ./app/src:/var/www/html
      - ./app/docker/nginx/default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - app
    networks:
      - talk-to-world-network

networks:
  talk-to-world-network:
    driver: bridge

volumes:
  db_data:
```

### 各サービスの概要

- **app**
  - **用途**: Laravel アプリケーション (フロントエンド＆バックエンド)
  - **ポート**: ホスト側の `3000` 番ポートにマッピング。Vite の開発サーバー用。
  - **ボリューム**: `./app/src`（サブモジュール）を `/var/www/html` にマウント。
- **api**

  - **用途**: Go 言語による WebSocket チャットサーバー (API)
  - **ポート**: `8080` 番ポートにマッピング。
  - **ボリューム**: `./api/cmd`（サブモジュール）を `/app` にマウント。

- **mysql**

  - **用途**: MySQL 8.0 データベース
  - **ポート**: `3306` 番ポートにマッピング。
  - **ボリューム**: 永続化用の `db_data` を利用。

- **nginx**
  - **用途**: Nginx リバースプロキシ
  - **ポート**: ホスト側の `8090` 番ポートにマッピングし、コンテナ内の `80` 番ポートへ。
  - **ボリューム**: `./app/src`（サブモジュール）および `./app/docker/nginx/default.conf` をマウント。

---

## 動作環境

- **Docker / Docker Compose**
  - Docker version: **20.x** 以上
  - Docker Compose version: **1.29** 以上 (または Docker Compose v2)
- **app コンテナ (Laravel + Vite)**
  - Node.js: **v18.20.6**
  - npm: **10.8.2**
- **api コンテナ (Go)**
  - Go: **go1.23.6 linux/amd64**
- ※ Node.js は、Laravel 側で Vite を実行するために必要です。

---

## 初期セットアップ

1. **リポジトリのクローン**  
   サブモジュールも含めてクローンするため、以下のコマンドを実行してください。

   ```bash
   git clone --recurse-submodules https://github.com/Skosuke/talk-to-world.git
   cd talk-to-world
   ```

2. **環境変数の設定（必要に応じて）**

   - Laravel 側 (`app/src` ディレクトリ直下) の `.env` ファイル
   - Go 側 (`api/cmd` または `api/` 直下) の `.env` ファイル  
     ※ DB 接続設定や各種ポート番号など、ご自身の環境に合わせて編集してください。

3. **依存パッケージのインストール（ローカル実行の場合）**
   - **Laravel 側**:
     ```bash
     cd app/src
     composer install
     npm install
     ```
   - **Go 側**:
     ```bash
     cd ../../api/cmd
     go mod tidy
     ```
     ※ Docker Compose 経由で実行する場合、依存パッケージのインストールは Dockerfile 内で行われることが多いため、必ずしもローカルでの作業は不要です。

---

## 起動方法

1. **Docker Compose のビルド & 起動**

   ```bash
   docker-compose up --build -d
   ```

   - `--build` オプションにより、Dockerfile の変更を反映した新しいイメージを再ビルドします。
   - `-d` はバックグラウンド起動 (detached mode) を意味します。

2. **コンテナの状態確認**

   ```bash
   docker-compose ps
   ```

   - `app`、`api`、`mysql`、`nginx` 各コンテナが正常に起動していることを確認してください。

3. **アプリケーションへのアクセス**

   - **Laravel アプリケーション (フロントエンド)**  
     アクセス URL: [http://localhost:8090](http://localhost:8090)  
     ※ Nginx 経由で配信されます。
   - **Vite の開発サーバー**  
     アクセス URL: [http://localhost:3000](http://localhost:3000)  
     ※ 開発中のホットリロードなどに利用します。

     **※ Vite の開発サーバー起動について**  
     開発中は、`app` コンテナ内またはローカルの `app/src` ディレクトリで以下のコマンドを実行してください。

     ```bash
     npm run dev
     ```

   - **Go API (WebSocket チャットサーバー)**  
     接続 URL: `ws://localhost:8080/ws`

   > ※ ポートマッピングや Nginx の設定は、`docker-compose.yml` および `./app/docker/nginx/default.conf` の内容をご確認ください。

---

## 開発時の注意

1. **ホットリロード (Laravel + Vite)**

   - ソースコード変更時、Vite によるホットリロードが有効になり、ブラウザが自動更新されます。
   - `.env` の設定が開発用になっていることを確認してください。

2. **Go 側の再コンパイル**

   - Go のコード変更時は、API コンテナの再ビルド・再起動が必要です。
   - ボリュームマウントによりローカルのコード変更は自動で反映されますが、反映されない場合はコンテナの再起動を行ってください。

3. **ログの確認**  
   各コンテナのログは以下のコマンドで確認できます:

   ```bash
   docker-compose logs -f app
   docker-compose logs -f api
   docker-compose logs -f mysql
   docker-compose logs -f nginx
   ```
