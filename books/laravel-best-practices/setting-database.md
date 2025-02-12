---
title: "データベースの設定"
---

# 概要

データベースの設計は仕様定義の次に重要です。

後に控えている実装はデータベースの設計をもとに行われるからです。

本章ではLaravelのマイグレーション機能を使ったテーブルの構築を紹介します。

:::message
**本章で学ぶこと**

- マイグレーションファイルの利用
- Seeder（とFaker）の利用
:::

# マイグレーション

LaravelのマイグレーションはPHPファイルからテーブルの定義・作成を行う機能です。

開発でデータベースを扱う際、共通利用のサーバーにデータベースを立てたり、ダンンプファイルを利用したりしますが、これらの方法ではテーブル定義を更新することになった時にいささか柔軟性に欠けます。

Laravelのマイグレーション機能をつくことの利点は変更と適用の素早さと柔軟性にあります。

今回のようにDockerを使用する場面では特に有用です。

## マイグレーションファイルの作成

マイグレーションファイルは手動でも作成できますが、`artisan`コマンドを使えばテンプレートを自動でつくってくれますし、ミスも減るので便利です。

今回はToDoを登録するテーブルを作ります。

以下のコマンドでマイグレーションファイルを作成してください。

```bash:/laravel-app
php artisan make:migration "create todos table"

   INFO  Migration [database/migrations/xxxx_xx_xx_xxxxxx_create_todos_table.php] created successfully.  
```

作成した時点ではこのようになっているはずです。

```php:/laravel-app/database/migrations/xxxx_xx_xx_xxxxxx_create_todos_table.php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('todos', function (Blueprint $table) {
            $table->id();
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('todos');
    }
};
```

今回のアプリケーションでは以下の項目が必要になります。

- ID
- ToDoのタイトル
- 詳細メモ
- 締め切り日時
- 登録日
- 色
- 完了状態

これらの項目をマイグレーションファルに定義しましょう。

```diff php:/laravel-app/database/migrations/xxxx_xx_xx_xxxxxx_create_todos_table.php
public function up(): void
{
    Schema::create('todos', function (Blueprint $table) {
        $table->id();
-       $table->timestamps();
+       $table->string('title')
+           ->comment('ToDoタイトル');
+       $table->text('memo')
+           ->nullable()
+           ->comment('詳細メモ');
+       $table->dateTime('deadline')
+           ->comment('締め切り日時');
+       $table->dateTime('created_at')
+           ->useCurrent()
+           ->comment('登録日');
+       $table->string('color')
+           ->nullable()
+           ->comment('色');
+       $table->boolean('done')
+           ->default(false)
+           ->comment('完了状態');
    });
}
```

`詳細メモ`と`色`については任意にするので`nullable`にしています。

`登録日`はデータベースに登録した日を自動的に割り当てるため、`useCurrent`を使用しています。

`完了状態`にはデフォルト値として`false`を設定しました。

テーブルから無のデータ型には明示的にサイズも指定できますが、こちらはデフォルト（`255`）のままで設定します。

カラム定義にはコメントも付け加えることができます（`comment`関数）。ダンプした際や、データベースサーバーで閲覧する際にも反映されるので、コメントは積極的に記述しましょう。

<br>

マイグレーションファイルが作成できたので、早速マイグレーションを実行しましょう。

```bash:/laravel-app
$ php artisan migrate

   INFO  Running migrations.  

  xxxx_xx_xx_xxxxxx_create_todos_table ....................................................... 25.42ms DONE

```

Tinkerを使ってちゃんとテーブルが作られているか確認します。

```bash:/laravel-app
$ php artisan tinker 
Psy Shell v0.12.7 (PHP 8.4.3 — cli) by Justin Hileman
> DB::select("SHOW TABLES") # 入力してEnter
= [
    {#5210
      +"Tables_in_db-name": "cache",
    },
    {#5212
      +"Tables_in_db-name": "cache_locks",
    },
    {#5213
      +"Tables_in_db-name": "failed_jobs",
    },
    {#5214
      +"Tables_in_db-name": "job_batches",
    },
    {#5209
      +"Tables_in_db-name": "jobs",
    },
    {#5198
      +"Tables_in_db-name": "migrations",
    },
    {#5208
      +"Tables_in_db-name": "password_reset_tokens",
    },
    {#5207
      +"Tables_in_db-name": "sessions",
    },
    {#5206
      +"Tables_in_db-name": "todos",
    },
    {#5205
      +"Tables_in_db-name": "users",
    },
  ]
```

ちゃんと`todos`テーブルが登録されています。

カラム定義もついでに確認しておきましょう。

```bash:/laravel-app
> DB::select("DESCRIBE todos");
= [
    {#5227
      +"Field": "id",
      +"Type": "bigint(20) unsigned",
      +"Null": "NO",
      +"Key": "PRI",
      +"Default": null,
      +"Extra": "auto_increment",
    },
    {#5224
      +"Field": "title",
      +"Type": "varchar(255)",
      +"Null": "NO",
      +"Key": "",
      +"Default": null,
      +"Extra": "",
    },
    {#5225
      +"Field": "memo",
      +"Type": "text",
      +"Null": "YES",
      +"Key": "",
      +"Default": null,
      +"Extra": "",
    },
    {#5218
      +"Field": "deadline",
      +"Type": "datetime",
      +"Null": "NO",
      +"Key": "",
      +"Default": null,
      +"Extra": "",
    },
    {#5229
      +"Field": "created_at",
      +"Type": "datetime",
      +"Null": "NO",
      +"Key": "",
      +"Default": "current_timestamp()",
      +"Extra": "",
    },
    {#5230
      +"Field": "color",
      +"Type": "varchar(255)",
      +"Null": "YES",
      +"Key": "",
      +"Default": null,
      +"Extra": "",
    },
    {#5231
      +"Field": "done",
      +"Type": "tinyint(1)",
      +"Null": "NO",
      +"Key": "",
      +"Default": "0",
      +"Extra": "",
    },
  ]
```

定義通りのカラムが確認できました。


# Seederの利用