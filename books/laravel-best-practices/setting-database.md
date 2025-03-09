---
title: "データベースの設定"
---

# 概要

## 本章で学ぶこと

- マイグレーションファイル
- Seederの利用
- Fakerの利用


## 前提

本書では**Eloguent**は最低限しか用いません。（認証で間接的に使うぐらい。）

直感的に使いやすいEloquentですが、大量のレコードを扱う際に速度の問題があり、また柔軟性に難点があります。

内部で発行されているクエリも不透明なので、デバッグの際に手間が生じます。

本書では一貫して**Query Builder**を利用します。

Query BuilderはSQLと近い文法で記述でき、実際の処理も直にSQLを発行した時と大きな差がありませんが、生のクエリを発行するよりも直感的に記述できます。

複雑なクエリになると流石に普通のSQLで書いた方が可読性が高いこともありますが、多くの場合はQuery Builderで事足りますし、ある程度なら生のクエリと共存させることができます。

:::message
以上は筆者の個人的見解です。

ORMがいいか、それともSQLがいいかについての議論はネット上にたくさんありますので、よかったらご自身で調べてみてください。
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
- ToDoを登録したユーザーのID

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
+       $table->foreignId('user_id')
+           ->references('id')
+           ->on('users');
+   });
}
```

`詳細メモ`と`色`については任意にするので`nullable`にしています。

`登録日`はデータベースに登録した日を自動的に割り当てるため、`useCurrent`を使用しています。

`完了状態`にはデフォルト値として`false`を設定しました。

テーブルから無のデータ型には明示的にサイズも指定できますが、こちらはデフォルト（`255`）のままで設定します。

ユーザーIDには外部参照のカラムを設定します。`forignId`にカラム名、`references`に参照先のカラム名、`on`に参照先テーブル名を渡します。

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
    {#5215
      +"Field": "id",
      +"Type": "bigint(20) unsigned",
      +"Null": "NO",
      +"Key": "PRI",
      +"Default": null,
      +"Extra": "auto_increment",
    },
    {#5213
      +"Field": "title",
      +"Type": "varchar(255)",
      +"Null": "NO",
      +"Key": "",
      +"Default": null,
      +"Extra": "",
    },
    {#5212
      +"Field": "memo",
      +"Type": "text",
      +"Null": "YES",
      +"Key": "",
      +"Default": null,
      +"Extra": "",
    },
    {#5211
      +"Field": "deadline",
      +"Type": "datetime",
      +"Null": "NO",
      +"Key": "",
      +"Default": null,
      +"Extra": "",
    },
    {#5216
      +"Field": "created_at",
      +"Type": "datetime",
      +"Null": "NO",
      +"Key": "",
      +"Default": "current_timestamp()",
      +"Extra": "",
    },
    {#5217
      +"Field": "color",
      +"Type": "varchar(255)",
      +"Null": "YES",
      +"Key": "",
      +"Default": null,
      +"Extra": "",
    },
    {#5218
      +"Field": "done",
      +"Type": "tinyint(1)",
      +"Null": "NO",
      +"Key": "",
      +"Default": "0",
      +"Extra": "",
    },
    {#5219
      +"Field": "user_id",
      +"Type": "bigint(20) unsigned",
      +"Null": "NO",
      +"Key": "MUL",
      +"Default": null,
      +"Extra": "",
    },
  ]
```

定義通りのカラムが確認できました。


# Seeder

開発中はよくダミーのデータを入れて処理や表示を確かめます。

しかし毎回データを手動で入れるのは手間です。

<br>

**Seeder**はデータベースに開発用のデータの登録を自動化してくれる機能です。

今回はこのSeederを使ってモックデータを作成しましょう。

:::message

**Seederを使うメリット**

Seederを使うことでデータ作成の手間を減らすことができますが、メリットはそれだけではありません。

実際にデータを入れることでテーブル定義の妥当性を即座に検証できます。

:::

## Seederの作成

今回はToDoのモックを作成するので`TodoSeeder`というSeederを作成します。

その前に、`todos`テーブルには外部参照カラムの`user_id`があるので、まずは`users`テーブルにデータを追加する必要があります。

以下のコマンドで`UserSeeder`と`TodoSeeder`を作成しましょう。

```bash:/laravel-app
$ php artisan make:seeder UserSeeder

   INFO  Seeder [database/seeders/UserSeeder.php] created successfully. 

$ php artisan make:seeder TodoSeeder

   INFO  Seeder [database/seeders/TodoSeeder.php] created successfully. 
```

`database/seeders`の下に新しくファイルが追加されていることを確認してください。

## Seederの定義

まずは`UserSeeder`から作成していきます。

中身を以下のように変更してください。

```diff php:/laravel-app/database/seeders/UserSeeder.php
  <?php

  namespace Database\Seeders;

  use Illuminate\Database\Console\Seeds\WithoutModelEvents;
  use Illuminate\Database\Seeder;
+ use Illuminate\Support\Facades\DB;

  class UserSeeder extends Seeder
  {
      /**
       * Run the database seeds.
       */
      public function run(): void
      {
-         //
+         $user = [
+           'name' => 'test_user',
+           'email' => env('TEST_USER_MAIL'),
+           'password' => env('TEST_USER_PASSWORD'),
+         ];
+ 
+         DB::table('users')->insert($user);
      }
  }
```

ユーザーのメールアドレスとパスワードは`.env`ファイルに定義しています。

`UserSeeder`ができましたので、次は`TodoSeeder`を作っていきます。


```diff php:/laravel-app/database/seeders/TodoSeeder.php
  <?php

  namespace Database\Seeders;

  use Illuminate\Database\Console\Seeds\WithoutModelEvents;
  use Illuminate\Database\Seeder;
  use Illuminate\Support\Facades\DB;

  class TodoSeeder extends Seeder
  {
      /**
      * Run the database seeds.
      */
      public function run(): void
      {
-        //
+        $todos = [];
+
+        $user = DB::table('users')->first();
+
+        foreach(range(0, 4) as $_) {
+
+            $todos[] = [
+                'title' => fake()->realText(10),
+                'memo' => fake()->boolean() ? fake()->realText(30) : null,
+                'deadline' => fake()->dateTimeBetween('now', '+10 days'),
+                'color' => null,
+                'done' => fake()->boolean(),
+                'user_id' => $user->id,
+            ];
+        }
+
+        DB::table('todos')->insert($todos);
      }
  }

```

データを自動生成するために**Faker**を利用しています。

`.env`ファイルを編集した際に`APP_FAKER_LOCALE`を`ja_JP`としましたが、そうすることで`realText`などのメソッドを使った際に日本語で出力するようになります。

Fakerライブラリについてはここでは詳しく扱いませんが、[リポジトリ](https://github.com/fzaninotto/Faker)に詳しい説明があります。

## Seedの設定

Seederファイルを作っただけではSeedは実行できません。（個別にファイルを指定すればできますが。）

一度にすべてのSeederを実行するには`DatabaseSeeder.php`にSeederを登録する必要があります。

```diff php:/laravel-app/database/seeders/DatabaseSeeder.php
  <?php

  namespace Database\Seeders;

  use App\Models\User;
  // use Illuminate\Database\Console\Seeds\WithoutModelEvents;
  use Illuminate\Database\Seeder;
+ use Illuminate\Support\Facades\DB;
+ use Illuminate\Support\Facades\Schema;

  class DatabaseSeeder extends Seeder
  {
      /**
      * Seed the application's database.
      */
      public function run(): void
      {
-        // User::factory(10)->create();
-
-        User::factory()->create([
-            'name' => 'Test User',
-            'email' => 'test@example.com',
-        ]);
+        Schema::disableForeignKeyConstraints();
+  
+        DB::table('users')->truncate();
+        DB::table('todos')->truncate();
+  
+        Schema::enableForeignKeyConstraints();
+  
+        $this->call([
+            UserSeeder::class,
+            TodoSeeder::class,
+        ]);
      }
  }

```

Seederを実行した際にはこの`DatabaseSeeder`が起点になりますので、ここで他のSeederを呼び出します（`call`メソッド）。

`Schema`ファサードのところで外部参照制約を無効化し、順不同にテーブルを初期化するようにしています。


## Seederの実行

Seederの準備ができたので実行してみましょう。

```bash:/laravel-app
$ php artisan db:seed

   INFO  Seeding database.  

  Database\Seeders\UserSeeder ..................................................................................................... RUNNING  
  Database\Seeders\UserSeeder ................................................................................................... 2 ms DONE  

  Database\Seeders\TodoSeeder ..................................................................................................... RUNNING  
  Database\Seeders\TodoSeeder .................................................................................................. 26 ms DONE  

```

これでダミーデータの作成が完了しました。

実際に登録されているかTinkerで確認してみましょう。

```bash:/laravel-app
$ php artisan tinker
Psy Shell v0.12.7 (PHP 8.4.3 — cli) by Justin Hileman
> DB::table('users')->get();
= Illuminate\Support\Collection {#5211
    all: [
      {#5215
        +"id": 1,
        +"name": "test_user",
        +"email": "******@********",
        +"email_verified_at": null,
        +"password": "*************",
        +"remember_token": null,
        +"created_at": null,
        +"updated_at": null,
      },
    ],
  }

> DB::table('todos')->get();
= Illuminate\Support\Collection {#5224
    all: [
      {#5220
        +"id": 1,
        +"title": "けびました。ルビー。",
        +"memo": "たがねえ、その霧きり六十度どこから、早く鳥がおもしれなくな。",
        +"deadline": "2025-02-21 00:03:32",
        +"created_at": "2025-02-16 11:38:30",
        +"color": null,
        +"done": 0,
        +"user_id": 1,
      },
      {#5219
        +"id": 2,
        +"title": "さまざまれ、黒い丘。",
        +"memo": "の眼めがしてきゅうだ」カムパネルラがすと、野原を指ゆびでき。",
        +"deadline": "2025-02-24 08:07:44",
        +"created_at": "2025-02-16 11:38:30",
        +"color": null,
        +"done": 0,
        +"user_id": 1,
      },
      {#5218
        +"id": 3,
        +"title": "ぐに落おちてしから。",
        +"memo": null,
        +"deadline": "2025-02-24 21:30:43",
        +"created_at": "2025-02-16 11:38:30",
        +"color": null,
        +"done": 0,
        +"user_id": 1,
      },
      {#5225
        +"id": 4,
        +"title": "そっちでいちばんの。",
        +"memo": null,
        +"deadline": "2025-02-23 05:34:33",
        +"created_at": "2025-02-16 11:38:30",
        +"color": null,
        +"done": 1,
        +"user_id": 1,
      },
      {#5226
        +"id": 5,
        +"title": "がわの雲で鋳いた小。",
        +"memo": "士はかせいのりんどは思われませんです」「それはいつをとった。",
        +"deadline": "2025-02-19 08:55:36",
        +"created_at": "2025-02-16 11:38:30",
        +"color": null,
        +"done": 0,
        +"user_id": 1,
      },
    ],
  }
```

Seederについての説明は以上ですが、開発を進めるにあたって必要であれば適宜説明を行います。