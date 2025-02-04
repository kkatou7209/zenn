---
title: "開発環境の準備"
---

# 概要

本章ではLaravelの開発環境の構築を行います。

使用するツールをざっと紹介します。

## Docker

Dockerは仮想環境を実行するためのツールです。

普通言われる仮想環境とは少し違いのですが、詳しい説明はここでは行いません。

開発環境の準備や破棄が簡単に行えるので、今回はDocker上に開発環境を構築します。

## Visual Studio Code (VSCode)

Microsoftが開発しているエディターです。

Dockerとスムーズに連携できる拡張機能があるので利用します。

# Docker Desktop のインストール

1. まず[こちらのリンク](https://www.docker.com/get-started/)から **Docker Desktop** をインストールしてください。

2. *Download Docker Desktop* から、自分のPCにあったインストーラを選択してください。

![](https://storage.googleapis.com/zenn-user-upload/c2a255077be0-20250202.png)

3. インストールが終わったらアプリケーションメニューから Docker Desktop を起動します。

4. Docker Hub へのログインを求められるかもしれませんが、スキップして構いません。

:::message
**Windows で Docker を使用する場合**
Windowsでコンテナ（仮想環境）を動かすためには **Windows Subsystem for Linux（WSL）** を有効化する必要があります。
WSL を有効化する方法についてはインターネット上に情報が転がっているのでここでは説明しません。
:::

:::message
**Mac OS で Docker を使う場合**
Docker Desktop よりも軽量で高速な **OrbStack** というツールがあります。
Mac で Docker を使用する際はこちらをお勧めします。
Docker Desktop で開発をしていた場合も簡単にデータを移行できます。
:::

# VSCode のインストール

[こちらのリンク](https://code.visualstudio.com/download)へいき、自身の環境にあったインストーラーをダウンロードしてください。

# プロジェクトの作成

コマンドプロンプト（またはターミナル）を開き、任意のディレクトリに移動してください。（ここではホームディレクトリを使用することにします）

```bash
$ cd ~
```

好きな名前でディレクトリを作りましょう。ここでは `laravel-app` にします。

作成したそのディレクトリに移動してください。

```bash
$ mkdir laravel-app
$ cd laravel-app
```

現在のディレクトリで VSCode を開きます。

```bash:laravel-app
$ code .
```

## Dev Containers のインストール

**Dev Containers** は Docker での開発を可能にするための VSCode の拡張機能です。

VSCode が立ち上がったら、左のナビゲーションにある下記のアイコンをクリックしてください。

![](https://storage.googleapis.com/zenn-user-upload/0283e1e3cd67-20250202.png)

次に検索バーに　`Dev Containers` と入力します。

![](https://storage.googleapis.com/zenn-user-upload/c7bb4def8d64-20250202.png =400x)

一番上に現れた拡張機能をクリックし、インストールしてください。

## コンテナの作成

今回は PHP と MariaDB のコンテナを立てます。

それぞれ PHP コンテナは Laravel 用、MariaDB はデータベースです。

プロジェクトディレクトリに `.devcontainer`というディレクトリを作ってください。

VSCode のエクスプローラー（左のナビゲーションの一番上）を開き、フォルダアイコンをクリックすることで作成できます。

![](https://storage.googleapis.com/zenn-user-upload/2b0d2b5c8b80-20250202.png =400x)

次に `.devcontainer` ディレクトリの下に `app` ディレクトリと `db` ディレクトリを作成します。

![](https://storage.googleapis.com/zenn-user-upload/d45a0a81b8e5-20250202.png =400x)

`.devcontainer` ディレクトリの直下に `compose.yml` というファイルを作成します。

`compose.yml` はコンテナを立ち上げる際の起点となるファイルです。

![](https://storage.googleapis.com/zenn-user-upload/f360922f8d4f-20250202.png =400x)

最後に、`app` ディレクトリの直下に `devcontainer.json` というファイルを作ります。

`devcontainer.json` は個々のコンテナの設定ファイルだと思ってくれれば大丈夫です。

Dev Containers 拡張機能は `devcontainer.json` の設定からコンテナを立ち上げる際の情報を取得します。

![](https://storage.googleapis.com/zenn-user-upload/122f23c80237-20250202.png =400x)

<br>

ここまでのディレクトリ構造は以下のようになります。

```
laravel-app/
├── .devcontainer/
│   ├── app/
│   │   └── devcontainer.json
│   └── db
└── compose.yml
```

---

次に `compose.yml` でコンテナの設定を行います。`compose.yml` ファイルをクリックしてください。

`compose.yml` は **Docker Compose** という機能で使用するファイルです。Docker Compose 自体は Docker Desktop インストール時に一緒にインストールされています。

Docker Compose を使うと、複数のコンテナを同時に立ち上げることができます。

`compose.yml` の内容については本筋から外れるため詳しくは説明しません。

以下の内容をコピーして `compose.yml` に貼り付けてください。

```yml:laravel-app/.devcontainer/compose.yml
name: laravel-app

services:
  app:
    container_name: laravel-app-app
    build:
      context: ./app
    volumes:
      - ../:/laravel-app:cached
      - laravel-app-vendor:/laravel-app/vendor
      - laravel-app-node_modules:/laravel-app/node_modules
    tty: true
    networks:
      - laravel-app-net
    depends_on:
      - db

  db:
    container_name: laravel-app-db
    image: mariadb:11.4.4
    environment:
      MARIADB_ROOT_PASSWORD: root
      MARIADB_DATABASE: db-name
      MARIADB_USER: db-user
      MARIADB_PASSWORD: db-pass
      TZ: Asia/Tokyo
    ports:
      - 3306:3306
    networks:
      - laravel-app-net

volumes:
  laravel-app-vendor:
  laravel-app-node_modules:

networks:
  - laravel-app-net
```

次に Laravel のコンテナを作る際の `Dockerfile` を作成します。

`.devcontainer/app` の直下に `Dockerfile` を作成し、以下を貼り付けてください。

```dockerfile:laravel-app/.devcontainer/app/Dockerfile
FROM php:8.4

SHELL ["/bin/bash", "-c"]

RUN apt update && apt install zip unzip curl -y && \
    docker-php-ext-install pdo pdo_mysql mysqli && \
    docker-php-ext-enable pdo_mysql

RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" && \
    php -r "if (hash_file('sha384', 'composer-setup.php') === 'dac665fdc30fdd8ec78b38b9800061b4150413ff2e3b6f88543c636f7cd84f6db9189d43a81e5503cda447da73c7e5b6') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;" && \
    php composer-setup.php --filename=composer --install-dir=/usr/local/bin && \
    php -r "unlink('composer-setup.php');"

RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash && \
    source ~/.bashrc && nvm install node --lts
```

ここでやっていることを簡単に説明すると: 

1. `FROM` で PHP コンテナのテンプレートイメージを使用することを宣言しています。
2. `SHELL` では後の `RUN` で使用するシェルの種類に **Bash** を指定しています。
4. 次の `RUN` でコンテナないにアプリケーションに必要な Linux ライブラリを読み込んでいます。
5. その次の `RUN` では PHP のパッケージマネージャーである [**Composer**](https://getcomposer.org/) をインストールしています。
6. 最後に JavaScript の実行環境である **Node.js** をインストールしています。

最後に `devcontainer.json` を編集しましょう。

以下の内容をそのまま貼り付けてください。

```json:laravel-app/.devcontainer/app/devcontainer.json
{
    "name": "Laravel App",
    "dockerComposeFile": "../compose.yml",
    "service": "app",
    "workspaceFolder": "/laravel-app",
    "customizations": {
        "vscode": {
            "settings": {
                "php.validate.executablePath": "/usr/bin/php",
                "intelephense.environment.phpVersion": "8.4",
                "intelephense.environment.includePaths": [ "/laravel-app" ]
            },
            "extensions": [
                "onecentlin.laravel-extension-pack",
                "formulahendry.auto-rename-tag",
                "bmewburn.vscode-intelephense-client",
                "edwinhuish.better-comments-next",
                "ecmel.vscode-html-css",
                "neilbrayfield.php-docblocker",
                "jacobcassidy.css-nesting-syntax-highlighting"
            ]
        }
    }
}
```

ここでは Dev Containers に対して、`compose.yml` 上のサービス名、コンテナに入った後に VSCodeで開くディレクトリ、コンテナ内の VSCode にインストールする拡張機能などの必要な指示を出しています。

## コンテナの起動

ここまで完了したら、VSCode の左下のアイコンをクリックしてください。

![](https://storage.googleapis.com/zenn-user-upload/18b159554751-20250202.png)

エディター上部に選択肢が表示されるので、その中の `Reopen in Container` を選択してください。コンテナの起動が始まります。

コンテナの起動が完了し、VSCode のエディタが再度表示されたら、上のナビゲーションから `Terminal` を選択し、`New Terminal` をクリックしてください。

ターミナルが開いたら、PHP などのインストールがうまくいっているか確認してみましょう。

```bash:laravel-app
$ php --version
PHP 8.4.3 (cli) (built: Jan 17 2025 17:30:17) (NTS)
Copyright (c) The PHP Group
Built by https://github.com/docker-library/php
Zend Engine v4.4.3, Copyright (c) Zend Technologies
$ composer --version
Composer version 2.8.5 2025-01-21 15:23:40
PHP version 8.4.3 (/usr/local/bin/php)
Run the "diagnose" command to get more detailed diagnostics output.
$ node -v
v23.7.0
```

ディレクトリ構造は以下のようになっているはずです。

![](https://storage.googleapis.com/zenn-user-upload/42ae57c4ee36-20250202.png =400x)