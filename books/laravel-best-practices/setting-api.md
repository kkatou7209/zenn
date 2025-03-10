---
title: "APIの設定"
---

# 概要

ToDo関連の処理の中で、完了・未完了の切り替えの実装がまだでした。

今回はこれをAPIで実装しようと思います。

## 本章で学ぶこと

- APIルートの設定
- Axiosの導入

# APIを実装する準備

APIを使うのに追加のパッケージなどは必要ありませんが、便利なコマンドがあるので今回は以下の方法で準備を進めます。

`php artisan install:api`というコマンドを実行してください。

```bash
$ php artisan install:api # <--- 実行
./composer.json has been updated
Running composer update laravel/sanctum
Loading composer repositories with package information
Updating dependencies
Lock file operations: 1 install, 0 updates, 0 removals
  - Locking laravel/sanctum (v4.0.8)
Writing lock file
Installing dependencies from lock file (including require-dev)
Package operations: 1 install, 0 updates, 0 removals
  - Downloading laravel/sanctum (v4.0.8)
  - Installing laravel/sanctum (v4.0.8): Extracting archive
Generating optimized autoload files
> Illuminate\Foundation\ComposerScripts::postAutoloadDump
> @php artisan package:discover --ansi

   INFO  Discovering packages.  

  laravel/pail .......................................................... DONE
  laravel/sail .......................................................... DONE
  laravel/sanctum ....................................................... DONE
  laravel/tinker ........................................................ DONE
  nesbot/carbon ......................................................... DONE
  nunomaduro/collision .................................................. DONE
  nunomaduro/termwind ................................................... DONE

80 packages you are using are looking for funding.
Use the `composer fund` command to find out more!
> @php artisan vendor:publish --tag=laravel-assets --ansi --force

   INFO  No publishable resources for tag [laravel-assets].  

Found 1 security vulnerability advisory affecting 1 package.
Run "composer audit" for a full list of advisories.

   INFO  Published API routes file.  

 One new database migration has been published. Would you like to run all pending database migrations? (yes/no) [yes]:
$ yes # <--- yesで大丈夫です

   INFO  Running migrations.  

  xxxx_xx_xx_xxxxxx_create_personal_access_tokens_table ............................................................... 57.60ms DONE


   INFO  API scaffolding installed. Please add the [Laravel\Sanctum\HasApiTokens] trait to your User model.
```

上記のコマンドで**Sanctum**というパッケージがインストールされ、ついでに`api.php`というファイルと`create_personal_access_tokens_table`というマイグレーションファイルが作られました。

先ほどのインタラクションで間違って`no`を選択してしまった場合は`php artisan migrate`を実行してください。

## Sanctumとは

[**Sanctum**](https://laravel.com/docs/12.x/sanctum#main-content)は認証用のパッケージです。APIに関する煩雑な処理を肩代わりしてくれます。

[**Passport**](https://laravel.com/docs/12.x/passport#main-content)という別の認証パッケージもありますが、こちらとの使い分けはOAuthを実装するかどうかです。

今回は単純なAPI認証だけ実装するのでSanctumを使います。


# APIルートの設定

APIのルート設定は`api.php`で行います。

こちらのファイルは`web.php`のAPI版です。使い方は`web.php`と大差ありません。

ただ、デフォルトの設定だと指定したルートの前に`/api`が付け加えられるので注意です。こちらの設定は`bootstrap/app.php`で変更できますが、今回はこのままで実装します。

詳しくは[こちら](https://laravel.com/docs/12.x/routing)を参照してください。

## ルートの指定

今回必要になるAPIはToDoの完了・未完了状態を変更するためのAPIだけを実装しますので、指定するルートも一つだけです。

`api.php`を以下のように変更しましょう。

```php:/laravel-app/routes/api.php
<?php

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

Route::as('api.')->group(function() {

    Route::prefix('/todo')
        ->as('todo.')
        ->group(function() {

            Route::get('/toggle', function() {

                return ['test' => 'API TEST'];
            })->name('toggle');
        });
});
```

まず最初にAPIおルートすべての名前に`api.`がつくようにグループ化しています。

次に`todo.`というグループを設け、ルートの接頭辞に`/todo`がつくようにしています。

最後に`/api/todo/toggle`というルートにGETメソッドでリクエストすると、`{"test": "API TEST"}`というJSONが返ってくるようにしています。

一つのルートに大袈裟じゃないかと思うかもしれませんが、単純な実装でも意図がわかりやすいようにしておくことは肝心です。

上のようにしておけば、APIのルート名にはすべて`api.`がつくこと、扱う多少ごとに`prefix`でグループ化すること、メソッドチェーンで記述する際は1段文落として記述すること、などが後から触る人にもわかります。（無視して実装されたら頑として抗議しましょう。）

もし`Route::get('/todo/toggle', fn() => ['test' => 'API TEST']);`などと書こうものなら、後からどんな書かれ方をされても文句は言えません。

:::message
Laravelでレスポンスとして`array`または`stdClass`を返すと自動的にJSONにエンコードされます。

他にも`Collection`やEloquentのモデルなどもJSONとして返せます。
:::

## FormRequestの作成

## コントローラーの作成

APIを実装する場合でも`web.php`の場合に習ってコントローラーを作りましょう。

今回も`__invoke`メソッドに処理を記述します。（一度始めたことは最後まで一貫しましょう）

それではいつものようにコントローラーを作成するコマンドを実行しましょう。

```php
$ php artisan make:controller Api/Todo/ToggleController

   INFO  Controller [app/Http/Controllers/Api/Todo/ToggleController.php] created successfully.
```

