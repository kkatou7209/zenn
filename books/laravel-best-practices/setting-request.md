---
title: "リクエスト/バリデーションの設定"
---

# 概要

## 本章で学ぶこと

- リクエストのフォーマットの設定
- バリデーションの設定

## 前提

Laravelではコントローラーの引数の型に`Illuminate\Http\Request`を指定することでリクエストに関する処理を扱うことができます。

リクエストに対するバリデーションは`validate`メソッドで行うことができます。

各コントローラーでバリデーションを行っても良いのですが、`Illuminate\Foundation\Http\FormRequest`クラスを利用することで、コントローラーの処理からバリデーションの処理を引き剥がすことができます。

そうすると何がメリットになるかというと、データの取得やページの返却に関する処理の中にバリデーションを紛れ込ませる必要がなくなり、コードの見通しが良くなります。

# ルートの追加

リクエストの作成を行う前に、まだページを返すためのルートしか用意していなかったので、ここで新しいルートを追加します。

|処理|メソッド|URL|
|:-|:-|:-|
|ログイン|`POST`|`/user/login`|
|アカウント新規作成|`POST`|`/user/create`|
|ToDo新規作成|`POST`|`/todo/create`|
|ToDo更新|`PUT`|`/todo/update`|
|ToDo削除|`DELETE`|`/todo/delete`|

今回は先にコントローラーを作りましょう。

```bash:/laravel-app
$ php artisan make:controller User/LoginController

   INFO  Controller [app/Http/Controllers/User/LoginController.php] created successfully.  

$ php artisan make:controller User/CreateController

   INFO  Controller [app/Http/Controllers/User/CreateController.php] created successfully.  

$ php artisan make:controller Todo/CreateController

   INFO  Controller [app/Http/Controllers/Todo/CreateController.php] created successfully.  

$ php artisan make:controller Todo/UpdateController

   INFO  Controller [app/Http/Controllers/Todo/UpdateController.php] created successfully.  

$ php artisan make:controller Todo/DeleteController

   INFO  Controller [app/Http/Controllers/Todo/DeleteController.php] created successfully.  
```

:::message
「なぜリポジトリと同じように処理をクラスにまとめないのか」と思う方もいるでしょう。

今回も「**１ページあたり１コントローラー**」の原則に則っているからです。

例えばToDoを登録するリクエストに対しても、登録後に一覧ページに遷移するといったページを返却する処理が行われることを考えると、GET以外のPOSTなどのリクエストもページを要求する処理の一つだと考えられます。
:::

現状はまだ空の`__invoke`メソッドを用意しておきます。

```diff php:/laravel-app/app/Http/Controllers/User/LoginController.php
    <?php

    namespace App\Http\Controllers\User;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;

    class LoginController extends Controller
    {
-       //
+       public function __invoke()
+       {
+       
+       }
    }
```

```diff php:/laravel-app/app/Http/Controllers/User/CreateController.php
    <?php

    namespace App\Http\Controllers\User;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;

    class CreateController extends Controller
    {
-       //
+       public function __invoke()
+       {
+       
+       }
    }
```

```diff php:/laravel-app/app/Http/Controllers/Tood/CreateController.php
    <?php

    namespace App\Http\Controllers\Todo;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;

    class CreateController extends Controller
    {
-       //
+       public function __invoke()
+       {
+       
+       }
    }
```

```diff php:/laravel-app/app/Http/Controllers/Todo/UpdateController.php
    <?php

    namespace App\Http\Controllers\Todo;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;

    class UpdateController extends Controller
    {
-       //
+       public function __invoke()
+       {
+       
+       }
    }
```

```diff php:/laravel-app/app/Http/Controllers/Todo/DeleteController.php
    <?php

    namespace App\Http\Controllers\Todo;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;

    class DeleteController extends Controller
    {
-       //
+       public function __invoke()
+       {
+       
+       }
    }
```

`web.php`を編集してルートを設定しましょう。

```diff php:/laravel-app/routes/web.php
    <?php

    use Illuminate\Support\Facades\Route;
    use App\Http\Controllers\HomeController;
    use App\Http\Controllers\LoginController;
    use App\Http\Controllers\SignupController;
+   use App\Http\Controllers\User;
    use App\Http\Controllers\Todo;

    Route::get('/', HomeController::class)->name('home');

    Route::get('/login', LoginController::class)->name('login');

    Route::get('/signup', SignupController::class)->name('signup');

+   Route::prefix('/user')
+       ->as('user.')
+       ->group(function() {
+   
+           Route::post('/login', User\LoginController::class)->name('login');
+   
+           Route::post('/create', User\CreateController::class)->name('create');
+       });

    Route::prefix('/todo')
        ->as('todo.')
        ->group(function () {

            Route::get('/', Todo\IndexController::class)->name('index');

            Route::get('/new', Todo\NewController::class)->name('new');

            Route::get('/edit', Todo\EditController::class)->name('edit');
+
+           Route::post('/create', Todo\CreateController::class)->name('create');
+
+           Route::put('/update', Todo\UpdateController::class)->name('update');
+
+           Route::delete('/delete', Todo\DeleteController::class)->name('delete');
        });
```

# FormRequestクラスの作成

ここからっそく`FormRequest`を作っていきます。

ToDoの登録・更新・削除を行うリクエストを作成します。

下記のコマンドを実行してください。

```
$ php artisan make:request Todo/UpdateRequest

   INFO  Request [app/Http/Requests/Todo/UpdateRequest.php] created successfully.  

$ php artisan make:request Todo/CreateRequest   

   INFO  Request [app/Http/Requests/Todo/CreateRequest.php] created successfully.

$ php artisan make:request Todo/DeleteRequest

   INFO  Request [app/Http/Requests/Todo/DeleteRequest.php] created successfully. 
```

作成が完了したら、それぞれのリクエストを以下のように変更してください。

```diff php:/laravel-app/app/Http/Request/Todo/AddRequest.php
    <?php

    namespace App\Http\Requests\Todo;

    use Illuminate\Foundation\Http\FormRequest;

    class CreateRequest extends FormRequest
    {
        /**
        * Determine if the user is authorized to make this request.
        */
        public function authorize(): bool
        {
-           return false;
+           return true;
        }

        /**
        * Get the validation rules that apply to the request.
        *
        * @return array<string, \Illuminate\Contracts\Validation\ValidationRule|array<mixed>|string>
        */
        public function rules(): array
        {
            return [
-               //
+               'title' => ['required', 'string'],
+               'memo' => ['nullable', 'string'],
+               'date' => ['nullable', 'date_format:Y-m-d'],
+               'time' => ['nullable', 'date_format:H:i'],
+               'color' => ['nullable', 'string'],
            ];
        }
    }
```

```diff php:/laravel-app/app/Http/Request/Todo/UpdateRequest.php
    <?php

    namespace App\Http\Requests\Todo;

    use Illuminate\Foundation\Http\FormRequest;

    class UpdateRequest extends FormRequest
    {
        /**
        * Determine if the user is authorized to make this request.
        */
        public function authorize(): bool
        {
-           return false;
+           return true;
        }

        /**
        * Get the validation rules that apply to the request.
        *
        * @return array<string, \Illuminate\Contracts\Validation\ValidationRule|array<mixed>|string>
        */
        public function rules(): array
        {
            return [
-               //
+               'id' => ['required', 'integer'],
+               'title' => ['required', 'string'],
+               'memo' => ['nullable', 'string'],
+               'date' => ['nullable', 'date_format:Y-m-d'],
+               'time' => ['nullable', 'date_format:H:i'],
+               'color' => ['nullable', 'string'],
            ];
        }
    }

```

```diff php:/laravel-app/app/Http/Request/Todo/DeleteRequest.php
    <?php

    namespace App\Http\Requests\Todo;

    use Illuminate\Foundation\Http\FormRequest;

    class DeleteRequest extends FormRequest
    {
        /**
        * Determine if the user is authorized to make this request.
        */
        public function authorize(): bool
        {
-           return false;
+           return true;
        }

        /**
        * Get the validation rules that apply to the request.
        *
        * @return array<string, \Illuminate\Contracts\Validation\ValidationRule|array<mixed>|string>
        */
        public function rules(): array
        {
            return [
-               //
+               'id' => ['required', 'integer'],
            ];
        }
    }
```

`authorize`関数は認証・認可で利用するメソッドです。返り値として真偽値を返すことで、このリクエストがユーザーに’許可されているかを確認します。

今はただ`true`を返しているだけですが、後々ここにも処理を追加します。

`rules`メソッドはリクエストの入力値に対するバリデーションルールを返します。書き方は`Illuminate\Http\Request`の`validate`メソッドの引数と同じです。


# FormRequestの利用

作った`FormRequest`クラスをコントローラーに導入しましょう。

```diff php:/laravel-app/app/Http/Controllers/Todo/CreateController.php
    ...
+   use App\Http\Requests\Todo\CreateRequest;

    class CreateController extends Controller
    {
-       public function __invoke()
+       public function __invoke(CreateRequest $request)
        {
        }
    }
```

```diff php:/laravel-app/app/Http/Controllers/Todo/UpdateController.php
    ...
+   use App\Http\Requests\Todo\UpdateRequest;

    class UpdateController extends Controller
    {
-       public function __invoke()
+       public function __invoke(UpdateRequest $request)
        {
        }
}
```

```diff php:/laravel-app/app/Http/Controllers/Todo/DeleteController.php
    ...
+   use App\Http\Requests\Todo\DeleteRequest;

    class DeleteController extends Controller
    {
-       public function __invoke()
+       public function __invoke(DeleteRequest $request)
        {
        }
    }
```

`FormRequest`のバリデーションに引っかかったリクエストには自動的に元のページにリダイレクトされます。

まだ認証機能を実装していないのでこれで完成ではありませんが、これでバリデーションの処理をコントローラーから切り離すことができました。

これらのコントローラーは他のTodo関連のコントローラと同じく`TodoRepository`を使うので、Repositoryの設定ものままやってしまいましょう。


```diff php:/laravel-app/app/Http/Controllers/Todo/CreateController.php
    ...
+   use App\Repositories\Repository;

    class CreateController extends Controller
    {
+       public function __construct(
+           protected Repository $repository,
+       )
+       {}

        public function __invoke(CreateRequest $request)
        {

        }
    }
```

```diff php:/laravel-app/app/Http/Controllers/Todo/UpdateController.php
    ...
+   use App\Repositories\Repository;

    class UpdateController extends Controller
    {
+       public function __construct(
+           protected Repository $repository,
+       )
+       {}

        public function __invoke(UpdateRequest $request)
        {

        }
}
```

```diff php:/laravel-app/app/Http/Controllers/Todo/DeleteController.php
    ...
+   use App\Repositories\Repository;

    class DeleteController extends Controller
    {
+       public function __construct(
+           protected Repository $repository,
+       )
+       {}

        public function __invoke(DeleteRequest $request)
        {

        }
    }
```

```diff php:/laravel-app/app/Providers/RepositoryProvider.php
    ...
        $this->app->when([
                Todo\IndexController::class,
                Todo\NewController::class,
                Todo\EditController::class,
+               Todo\CreateController::class,
+               Todo\UpdateController::class,
+               Todo\DeleteController::class,
            ])
            ->needs(Repository::class)
            ->give(fn() => new TodoRepository());
    ...
```