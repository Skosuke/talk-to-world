# ベースイメージとして PHP 8.3 を使用
FROM php:8.3-fpm

# システム依存関係と PHP 拡張モジュールのインストール
RUN apt-get update && apt-get install -y \
    libpng-dev \
    libjpeg-dev \
    libfreetype6-dev \
    zip \
    unzip \
    git \
    curl \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install gd pdo pdo_mysql

# Composer のインストール
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# 作業ディレクトリの設定
WORKDIR /var/www/html

# アプリケーションファイルのコピー
# COPY ./src .

# Laravel の依存関係のインストール
# RUN composer install --no-dev --optimize-autoloader

# Node.js と npm のインストール
RUN curl -fsSL https://deb.nodesource.com/setup_18.x | bash - \
    && apt-get install -y nodejs

# フロントエンド依存関係のインストールとビルド
# RUN npm install \
#     && npm run build

# ポートの公開
EXPOSE 9000

# サービスの起動
CMD ["php-fpm"]
