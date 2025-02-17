---
title: "コントローラーの設定"
---

# 概要

本章ではコントローラーの設定を行います。

MVCフレームワークを使ったことがある方には不要な説明かもしれませんが、コントローラーとはクライアントからのリクエストを受け取り、それに応じてデータを取得し、動的にページを生成・返却する機能のことです。

コントローラーは`php artisan make:controller`で作成できます。

## 前提

Laravelには大きく２通りのコントローラーがあります。

複数のメソッドを持つコントローラーと単一の処理だけを行うコントローラーです。後者は`__invoke`メソッドを定義することで実装します。

本書の方針として、ページの関連のコントローラーには後者のコントローラーを採用します。

「ToDo関連のページなら`TodoController`にまとめれば良いのでは？」と考えるかと思いますが、本書の方針として、**テーブルとページは別の関心事**と割り切って開発を進めます。

確かに`TodoController`などに処理をまとめれば、共通の処理があった場合などに`TodoController`にまとめることができて便利でしょう。

しかしToDoの登録・編集・削除などの処理は本来コントローラーの仕事ではありません。コントローラーの役割は**インターフェース**としユーザーからのリクエストの処理を適切に他のモジュールに振り分けることです。データについての処理をコントローラーに定義するということは、インターフェース以上の役割をコントローラーに与えてしまうことになります。

そう考えると、表示するページの一つひとつは個々のインターフェースと考えることができます。

登録ページならば登録に関するインターフェースだけを提供し、編集ページなら編集に関する処理だけを提供するのが適切です。

# コントローラーの作成

前述の通り、今回は１ページにつき１つのコントローラーを定義していきます。

そうなるとルート名から推察しやすいコントローラ名やディレクトリ構成にすることが望ましいです。

```bash:/laravel-app
$ php artisan make:controller HomeController

   INFO  Controller [app/Http/Controllers/HomeController.php] created successfully.  

$ php artisan make:controller LoginController

   INFO  Controller [app/Http/Controllers/LoginController.php] created successfully.  

$ php artisan make:controller SignupController

   INFO  Controller [app/Http/Controllers/SignupController.php] created successfully.  

$ php artisan make:controller Todo/IndexController

   INFO  Controller [app/Http/Controllers/Todo/IndexController.php] created successfully.  

$ php artisan make:controller Todo/NewController

   INFO  Controller [app/Http/Controllers/Todo/NewController.php] created successfully.  

$ php artisan make:controller Todo/EditController

   INFO  Controller [app/Http/Controllers/Todo/EditController.php] created successfully.  

```

コントローラーが作成できましたら処理の方を記述していきます。

まずは`web.php`でやっていたページ返却処理をコントローラーに移していきます。

```diff php:/laravel-app/app/Http/Controllers/HomeController.php
    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class HomeController extends Controller
    {
-       //
+       public function __invoke()
+       {
+           return view('home');
+       }
    }
```

```diff php:/laravel-app/app/Http/Controllers/LoginController.php
    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class LoginController extends Controller
    {
-       //
+       public function __invoke()
+       {
+           return view('login');
+       }
    }
```

```diff php:/laravel-app/app/Http/Controllers/SignupController.php
    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class SignupController extends Controller
    {
-       //
+       public function __invoke()
+       {
+           return view('signup');
+       }
    }
```

```diff php:/laravel-app/app/Http/Controllers/Todo/IndexController.php
    <?php

    namespace App\Http\Controllers\Todo;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;

    class IndexController extends Controller
    {
-       //
+       public function __invoke()
+       {
+           return view('todo.index');
+       }
    }
```

```diff php:/laravel-app/app/Http/Controllers/Todo/NewController.php
    <?php

    namespace App\Http\Controllers\Todo;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;

    class NewController extends Controller
    {
-       //
+       public function __invoke()
+       {
+           return view('todo.new');
+       }
    }
```

```diff php:/laravel-app/app/Http/Controllers/Todo/EditController.php
    <?php

    namespace App\Http\Controllers\Todo;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;

    class EditController extends Controller
    {
-       //
+       public function __invoke()
+       {
+           return view('todo.edit');
+       }
    }
```

次にそれぞれのルートに対してコントローラーを設定していきます。

`web.php`を以下のように変更してください。

```diff php:/laravel-app/routes/web.php
    <?php

    use Illuminate\Support\Facades\Route;
+   use App\Http\Controllers\HomeController;
+   use App\Http\Controllers\LoginController;
+   use App\Http\Controllers\SignupController;
+   use App\Http\Controllers\Todo;

-   Route::get('/', function () {
-       return view('home');
-   })->name('home');
+   Route::get('/', HomeController::class)->name('home');

-   Route::get('/login', function() {
-       return view(('login'));
-   })->name('login');
+   Route::get('/login', LoginController::class)->name('login');

-   Route::get('/signup', function() {
-       return view(('signup'));
-   })->name('signup');
+   Route::get('/signup', SignupController::class)->name('signup');

    Route::prefix('/todo')
        ->as('todo.')
        ->group(function () {

-           Route::get('/', function() {
-               return view('todo.index');
-           })->name('index');
+           Route::get('/', Todo\IndexController::class)->name('index');

-           Route::get('/new', function () {
-               return view('todo.new');
-           })->name('new');
+           Route::get('/add', Todo\NewController::class)->name('new');

-           Route::get('/edit', function () {
-               return view('todo.edit');
-           })->name('edit');
+           Route::get('/edit', Todo\EditController::class)->name('edit');
        });
```

編集が終わりましたら、一度サーバーを立てて（`php artisan serve`）表示が変わっていないか確認してみてください。


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

### Repositoryクラスの作成

残念ながらLaravelにはRepositoryを扱う機能は標準で備わっていません。手動でファイル作成から実装まで行う必要があります。

`app`ディレクトリに`Repositories`というディレクトリを作って、そこにRepositoryを配置していくことにします。

Repositoryクラスを作るにあたって、まずはインタフェースを定義しましょう。インターフェースは`Repository.php`に定義します。

```php:/larave-app/app/Repositories/Repository.php
<?php

namespace App\Repositories;

use Illuminate\Support\Collection;

interface Repository
{
    public function find(int $id);

    public function list(): Collection;

    public function add(\stdClass|array $data): void;

    public function update(\stdClass|array $data): void;

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

    public function find(int $id)
    {
        return DB::table($this->table)
            ->where('id', $id)
            ->first();
    }

    public function list(): Collection
    {
        return DB::table($this->table)->get();
    }

    public function add(\stdClass|array $data): void
    {
        DB::table($this->table)->insert((array) $data);
    }

    public function update(\stdClass|array $data): void
    {
        DB::table($this->table)
            ->where($data->id)
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
+                   Todo\AddController::class,
+                   Todo\IndexController::class,
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
+           return view('todo.index', ['todos' => $this->repository->list()]);
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

![alt text](/images/setting-controller_1.png)