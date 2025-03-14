---
title: "サービスプロバイダーの設定"
---

# 概要

この章では前章で作ったコントローラーで使用するデータ取得クラスの作成を通してLaravelの**サービスプロバイダー**の使い方について学びます。

サービスプロバイダーはLaravelの**DI**機能の一部です。

## 本章で学ぶこと

- サービスプロバイダーの使い方


## データの返却

ページを返却する処理はできたので、次はSeederで作成したデータを表示させてみましょう。

コントローラー内で`DB`ファサードを使って取得もできるのですが、前述した通り、コントローラーは処理間の調整役なので、自分がデータの取得や更新を行うのは望ましくありません。

Webフレームワークにおいてデータベース関連の処理を担う機能としてよく**Reository**が登場します。ここでもRepositoryクラスを作りましょう。

:::message
Laravelにはコントローラーでデータを返す以外に、**View Composer**を使う方法があります。

View Composerを使うと特定のページに対して自動でデータ取得してページに埋め込んでくれます。

しかしデータを取得するだけで、登録や削除といった処理は実装できません。それにデータのでどころがコントローラー以外のモジュールに散らばってしまって、可読性の低下にもつながります。

本書ではデータの最終的な出口はコントローラーに統一します。
:::

:::message
**RepositroyとService**

本書ではRepositoryをデータベース処理に関わるもの、Serviceをビジネスロジックに関わるものと簡単に区別しておきます。

ECサイトを例に挙げると、ユーザーがセット商品を購入したときに、セット割引価格を適用した購入履歴データを生成するのがService、それをデータベースに登録するのがRepositoryとっいたところでしょうか。
:::

## Repositoryクラスの作成

残念ながらLaravelにはRepositoryを扱う機能は標準で備わっていません。手動でファイル作成から実装まで行う必要があります。

`app`ディレクトリに`Repositories`というディレクトリを作って、そこにRepositoryを配置していくことにします。

Repositoryクラスを作るにあたって、まずはインタフェースを定義しましょう。インターフェースは`Repository.php`に定義します。

```php:/larave-app/app/Repositories/Repository.php
<?php

namespace App\Repositories;

use Illuminate\Support\Collection;

interface Repository
{
    public function find(int $id): \stdClass|null;

    public function list(int $userId): Collection;

    public function add(\stdClass|array $data): void;

    public function update(int $id, \stdClass|array $data): void;

    public function delete(int $id): void;
}
```

次にこのインターフェースを実装した`TodoRepository`クラスを作成します。

```php:/larave-app/app/Repositories/TodoRepository.php
<?php

namespace App\Repositories;

use Illuminate\Support\Collection;
use Illuminate\Support\Facades\DB;

class TodoRepository implements Repository
{
    protected $table = 'todos';

    public function find(int $id): \stdClass|null
    {
        return DB::table($this->table)
            ->where('id', $id)
            ->first();
    }

    public function list(): Collection
    {
        return DB::table($this->table)
            ->where('user_id', $userId)
            ->get();
    }

    public function add(\stdClass|array $data): void
    {
        DB::table($this->table)->insert((array) $data);
    }

    public function update(int $id, \stdClass|array $data): void
    {
        DB::table($this->table)
            ->where('id', $id)
            ->update((array) $data);
    }

    public function delete(int $id): void
    {
        DB::table($this->table)->delete($id);
    }
}
```

返り値や引数に`stdClass`と指定しているのは、この型が`DB`ファサードでデータを取得した際の返り値だからです。

<br>

Repositoryのインターフェースに定義するメソッドはできるだけシンプルで用途が限定されたものにすべきです。

機能を盛り込んでしまうと、個々のリポジトリでは必ずしも必要でないのに実装しないとエラーが出てしまう、といったことになりかねません。

ポイントは基本の**CRUD**操作に対応したメソッドだけ定義し、個別のケースに対しては個々のRepositoryを継承して新しいリポジトリを作るか、Serviceクラスを利用することです。


### サービスプロバイダーの利用

前節で作ったリポジトリを早速使ってみたいところですが、それぞれのコントローラーでいちいち`TodoRepository`をインスタンス化するのは冗長です。

そこで**サービスプロバイダー**の出番です。

サービスプロバイダーはLaravelの**DI**機能で、抽象クラスやインターフェースをもとに具象クラスをインスタンス化して提供してくれます。

例えば、コントローラーのコンストラクター引数にインターフェースの型（`Repository`など）を指定すると、そのインターフェースを実装した具象クラス（`TodoRepository`）をインスタンス化して引数に渡してくれます。

サービスプロバイダーは`Illuminate\Support\ServiceProvider`を継承することで実装できます。

元々ある`AppServiceProvider`を利用してもいいのですが、今回は新しいプロバイダーを定義して利用します。

サービスプロバイダーを作成するには以下のコマンドを実行します。

```bash:/laravel-app
$ php artisan make:provider RepositoryServiceProvider
```

`app/Providers`に`RepositoryServiceProvider.php`が作成され、`bootstrap/providers.php`に`RepositoryServiceProvider`が追加されていることを確認してください。

```diff php:/laravel-app/bootstrap/providers.php
    <?php

    return [
        App\Providers\AppServiceProvider::class,
+       App\Providers\RepositoryServiceProvider::class,
    ];
```

次に`RepositoryServiceProvider.php`を編集していきます。

```diff php:/laravel-app/app/Providers/RepositoryServiceProvider.php
    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
+   use App\Http\Controllers\Todo;
+   use App\Repositories\Repository;
+   use App\Repositories\TodoRepository;

    class RepositoryServiceProvider extends ServiceProvider
    {
        /**
        * Register services.
        */
        public function register(): void
        {
-           //
+           $this->app->when([
+                   Todo\IndexController::class,
+                   Todo\NewController::class,
+                   Todo\EditController::class,
+               ])
+               ->needs(Repository::class)
+               ->give(fn() => new TodoRepository());
        }

        /**
        * Bootstrap services.
        */
        public function boot(): void
        {
            //
        }
    }
```

ここでは`when`メソッドを使ってDIを行う対象のコントローラーを指定し、`needs`でDIする抽象クラス・インターフェースの指定、`give`で具象クラスの提供を行なっています。

`when`に指定できるクラスはコントローラーに限りません。

次にコントローラー側でRepositoryを利用する処理を追記していきましょう。

```diff php:/laravel-app/app/Http/Controllers/Todo/IndexController.php
    <?php

    namespace App\Http\Controllers\Todo;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;
+   use App\Repositories\Repository;

    class IndexController extends Controller
    {
+       public function __construct(
+           protected Repository $repository,
+       )
+       {}

        public function __invoke()
        {
-           return view('todo.index');

+           $todos = $this->repository->list()
+               ->sortBy('deadline');
+   
+           return view('todo.index', ['todos' => $todos]);
        }
    }
```

```diff php:/laravel-app/app/Http/Controllers/Todo/NewController.php
    <?php

    namespace App\Http\Controllers\Todo;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;
+   use App\Repositories\Repository;

    class NewController extends Controller
    {
+       public function __construct(
+           protected Repository $repository,
+       )
+       {}

        public function __invoke()
        {
            return view('todo.new');
        }
    }
```

```diff php:/laravel-app/app/Http/Controllers/Todo/EditController.php
    <?php

    namespace App\Http\Controllers\Todo;

    use App\Http\Controllers\Controller;
+   use Illuminate\Http\Request;
    use App\Repositories\Repository;

    class EditController extends Controller
    {
+       public function __construct(
+           protected Repository $repository,
+       )
+       {}

        public function __invoke()
        {
            return view('todo.edit');
        }
    }
```

表示を確認するために`resources/views/tood/index.blade.php`を編集します。

```diff php:/laravel-app/resources/views/index.blade.php
    <div>
        <h1>ToDo一覧</h1>
+       <ul>
+           @foreach ($todos as $todo)
+               <li>{{ $todo->title }}</li>
+           @endforeach
+       </ul>
    </div>
```

この状態でサーバーを起動（`php artisan serve`）し、`/todo`にアクセスしてみましょう。

Seederで登録した内容が表示されてるでしょうか。

![alt text](/images/setting-controller_1.png =250x)
