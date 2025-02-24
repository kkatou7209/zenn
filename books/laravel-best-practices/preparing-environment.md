---
title: "開発環境の準備"
---

# 概要

## 本章で学ぶこと

- Docker環境の作成
- Laravelプロジェクトの作成
- データベース接続設定
- ViteとTypeScriptの設定

:::message
VSCodeとDockerはすでにインストール済みであることを前提として進めていきます。
:::

# Docker環境の作成

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

次にPHPとMariaDBのコンテナを準備します。

プロジェクトディレクトリに `.devcontainer`というディレクトリを作ってください。

VSCode のエクスプローラー（左のナビゲーションの一番上）を開き、フォルダアイコンをクリックすることで作成できます。

![](https://storage.googleapis.com/zenn-user-upload/2b0d2b5c8b80-20250202.png =400x)

`.devcontainer` ディレクトリの下に `app` ディレクトリと `db` ディレクトリを作成します。

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

# Laravelプロジェクトの作成

## テンプレートプロジェクトの作成

プロジェクトルート（ここでは `/laravel-app`）で `composer create laravel/laravel tmp` を実行します。

```bash:/laravel-app
$ composer create laravel/laravel tmp
```

`tmp` という名前のディレクトリが作られ、その下にLaravelのプロジェクトが初期化されます。

次にこの `tmp` 配下のプロジェクトをプロジェクトルートに引っ越しします。

:::message
現在のディレクトリで初期化することもできますが、すでにファイルやディレクトリがある場合は作成できません。
今回はすでに `.devcontainer` などのディレクトリが存在するため、一度別のディレクトリで初期化しています。
:::

プロジェクトができたらディレクトリを引っ越していきます。

プロジェクトルートで以下のコマンドを実行してください。

```bash:/laravel-app
$ rm -rf ./tmp/vendor/* # vendor以下は持ってこれないので消してしまう
$ mv ./tmp/{.,}* ./
```

もう `tmp` ディレクトリはいらないので消してしまいましょう。

```bash:/laravel-app
$ rm -r ./tmp
```

あとは以下のコマンドを実行してください。

```bash:/laravel-app
$ composer install # パッケージインストール
$ php artisan key:generate # 秘密鍵の生成
```

## 地域設定

Laravelの地域設定はデフォルトではアメリカになっています。

これを日本に変えましょう。

プロジェクト直下の`.env`を以下のように変えてください。

```diff ini:/laravel-app/.env
...
- APP_TIMEZONE=UTC
+ APP_TIMEZONE=Asia/Tokyo
...
- APP_LOCALE=en
- APP_FALLBACK_LOCALE=en
- APP_FAKER_LOCALE=en_US
+ APP_LOCALE=ja
+ APP_FALLBACK_LOCALE=ja
+ APP_FAKER_LOCALE=ja_JP
...
```

# データベース設定

次はLaravelとMariaDBを繋げましょう。

ドライバはコンテナ立ち上げ時にインストールしてあります。

`.env` ファイルを見てみてください。

データベース関連の項目が以下のようになっているかと思います。

```ini:/laravel-app/.env
DB_CONNECTION=sqlite
# DB_HOST=127.0.0.1
# DB_PORT=3306
# DB_DATABASE=laravel
# DB_USERNAME=root
# DB_PASSWORD=
```

次のように変更してください。

```diff ini:/laravel-app
- DB_CONNECTION=sqlite
- # DB_HOST=127.0.0.1
- # DB_PORT=3306
- # DB_DATABASE=laravel
- # DB_USERNAME=root
- # DB_PASSWORD=
+ DB_CONNECTION=mysql
+ DB_HOST=db
+ DB_PORT=3306
+ DB_DATABASE=db-name
+ DB_USERNAME=db-user
+ DB_PASSWORD=db-pass
```

コマンドでマイグレーションを行いましょう。

以下のような出力が出てくれば成功です。

```bash:/laravel-app
$ php artisan migrate

   INFO  Preparing database.  

  Creating migration table ............................................................................................................ 24.78ms DONE

   INFO  Running migrations.  

  0001_01_01_000000_create_users_table ................................................................................................ 71.08ms DONE
  0001_01_01_000001_create_cache_table ................................................................................................ 13.55ms DONE
  0001_01_01_000002_create_jobs_table ................................................................................................ 107.22ms DONE
```

:::message
**データベース自体の設定について**
「データベース自体の設定もしていないのになぜ接続できたのか」と不思議に思う方もいるかもしれません。
実はデータベースの上のデータベースやユーザーの作成は `compose.yml` の `environment` で行なっています。
特定の環境変数に値を設定することで、コンテナ作成時に自動で設定を行ってくれるのです。

ちなみに本番環境ではこのような設定で稼働させてはいけません。
パスワードも単調で予測しやすいですし、第一リポジトリ上に上がってしまっています。
実際の本番環境の構築ではちゃんとクエリを使って設定し、パスワードも強固なものにして別の場所に保管するようにしましょう。
:::


最後にサーバーを起動して正常に動いている確認しましょう。

```bash:/laravel-app
$ php artisan serve

   INFO  Server running on [http://127.0.0.1:8000].  

  Press Ctrl+C to stop the server
```

ブラウザで `http://127.0.0.1:8000` を開いてください。

Laravelの初期画面が表示されていればOKです。

![alt text](/images/image.png)


# ViteとTypeScriptの設定

デフォルトの設定でもJacaScriptとViteを利用できますが、TypeScriptbの方が潜在的なバグも減り、開発効率も上がります。

ここではTypeScriptの設定を行います。

TypeScript関連のパッケージをインストールする必要がありますが、まずはパッケージマネージャを変更しましょう。

今回はBunを使います。NPMよりもサイズが小さく、動作も高速なのでコンテナ向きです。

```bash:/laravel-app
$ npm i -g bun
```

Bunのインストールが完了したら次はTypeScriptの利用に必要なパッケージをインストールします。

```bash:/laravel-app
$ bun i -D typescript @types/jsdom @types/node
```

次に`tsconfig.json`をプロジェクトルートに作成してください。

以下の内容をコピペしてください。

```json:/laarvel-app/tsconfig.json
{
    "compilerOptions": {
        "baseUrl": "./",
        "paths": {
            "@/*": ["resources/ts/*"],
            "@css/*": ["resources/css/*"]
        },
        "target": "ESNext",
        "module": "ESNext",
        "types": [
            "vite/client"
        ],
        "strict": true,
        "noImplicitThis": true,
        "skipLibCheck": true,

    },
    "include": [
        "resources/ts/**/*.ts",
        "resources/ts/**/*.d.ts"
    ],
    "exclude": [
        "app",
        "bootstrap",
        "config",
        "database",
        "node_modules",
        "public",
        "routes",
        "storage",
        "tests",
        "vendor"
    ]
}
```

TypeScriptを利用するように変更したので、`vite.config.js`も変更していきます。

拡張子を`ts`に変えて以下のように変更してください。

```diff ts:/laravel-app/vite.config.ts
export default defineConfig({
    plugins: [
        laravel({
-            input: ['resources/css/app.css', 'resources/js/app.js'],
+            input: ['resources/ts/app.ts'],
            refresh: true,
        }),
    ],
});
```

`resources/js`ディレクトリを`resources/ts`に変更します。

`resiurces/ts`配下の`js`ファイルも`ts`ファイルに変更してください。

TypeScriptを利用している状態では`laravel-vite-plugin`がCSSをそのまま読み込めなくなるので、`app.ts`で読み込ませましょう。

```diff ts:/laarvel-app/resources/ts/app.ts
import './bootstrap';
+ import '@css/app.css';
```

最後にホットリロードの設定を行います。

`vite.config.ts`を以下のように変更してください。

```diff ts:/laravel-app/vite.config.ts
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import path from 'path';

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            refresh: true,
        }),
    ],
+    server: {
+        host: '127.0.0.1',
+        port: 5173,
+        strictPort: true,
+        hmr: {
+            protocol: 'ws'
+        },
+        watch: {
+            usePolling: true,
+            interval: 1000,
+        }
+    },
+    resolve: {
+        alias: {
+            '@': `${path.resolve(__dirname, 'resources/ts')}/`,
+            '@css': `${path.resolve(__dirname, 'resources/css')}/`,
+        }
+    }
});
```

`welcome.blade.php`で読み込んでいるファイル名を変更します。

```diff php:/laravel-app/resources/view/welcom.blade.php
        ...
        <!-- Styles / Scripts -->
        @if (file_exists(public_path('build/manifest.json')) || file_exists(public_path('hot')))
-            @vite(['resources/css/app.css', 'resources/js/app.js'])
+            @vite(['resources/ts/app.ts'])
        @else
        ...
```

Viteサーバーを立てて表示に問題がないか確認しましょう。

```bash:/laravel-app
$ bun run dev

  VITE v6.1.0  ready in 144 ms

  ➜  Local:   http://127.0.0.1:5173/
  ➜  press h + enter to show help

  LARAVEL v11.41.3  plugin v1.2.0

  ➜  APP_URL: http://localhost
```

`localhost:8000`をブラウザで開いてください。

コンソールにエラーなど出ていなければOKです。

<br>

---

開発環境の準備は以上です。