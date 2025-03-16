---
title: "コントローラーの設定"
---

# 概要

コントローラーはルートごとの内部処理の起点となるクラスです。

ここではコントローラーの作成方法とルートへの割り当て方法を説明します。


## 本章で学ぶこと

- コントローラーの作成


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


# まとめ

コントローラの作成は`php artisan make:controller`で行います。

今回は`__invoke`メソッドを使ってシングルメソッドコントローラーを実装しました。