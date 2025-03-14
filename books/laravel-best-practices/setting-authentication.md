---
title: "認証の設定"
---

# 概要

いよいよ認証の実装です。

ちょっと難しくなりますが、ひとつずつ説明していくので頑張ってください。

はじめにユーザーの作成と登録を実装します。テーブルは出来合いのものをそのまま使うので、データベースの追加設定などは必要ありません。

今回はカスタムされた認証などは実装せず、用意されたものをそのまま使います。

とは言っても、実装部分だけ取り上げると何をしているのかわかりませんので、適宜追加の説明をしていきます。


## 本書で学ぶこと

- ログイン・ログアウト
- リクエストの認証


# ユーザーの作成とログインの実装

データの登録やコントローラーの設定などは以前の章で扱いましたので改めて説明しません。

## `FormRequest`の作成


ユーザー登録用とログイン用の２つを作ります。

```bash:/laravel-app
$ php artisan make:request User/CreateRequest

   INFO  Request [app/Http/Requests/User/CreateRequest.php] created successfully.  

$ php artisan make:request Auth/LoginRequest

   INFO  Request [app/Http/Requests/Auth/LoginRequest.php] created successfully. 
```

中身は次のようにします。

```php:/laravel-app/app/Http/Requests/User/LoginRequest.php
<?php

namespace App\Http\Requests\User;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rules\Password;

class CreateRequest extends FormRequest
{
    /**
     * Determine if the user is authorized to make this request.
     */
    public function authorize(): bool
    {
        return true;
    }

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array<string, \Illuminate\Contracts\Validation\ValidationRule|array<mixed>|string>
     */
    public function rules(): array
    {
        return [
            'username' => ['required', 'string'],
            'email' => ['required', 'email:refc,dns', 'unique:users,email'],
            'password' => ['required', Password::defaults()],
            'password_confirmed' => ['required', 'same:password'],
        ];
    }

    public function messages()
    {
        return [
            'username.required' => 'ユーザー名は必ず入力してください',
            'email.required' => 'メールアドレスは必ず入力してください',
            'email.email' => '不正なフォーマットです',
            'password.required' => 'パスワードは必ず入力してください',
            'password.password' => '8文字以上使って作成してください',
            'password_confirmed' => 'パスワードが一致しません',
        ];
    }
}
```

新しいバリデーションが登場しましたので軽く説明しておきます。

### `email`

`email:rfc,dns`のうち、`email`と`rfc`は同じ意味です。どちらも`RFCValidation`が実行され、**RFC 5322**に準拠したメールアドレスかどうかを確認します。

`dns`は有効な**MXレコード**を持っているかどうかを確認します。

つまり`email:rfc,dns`は「**RFC 5322に準拠しており、有効なMXレコードを持っているメールアドレス**」という意味になります。


### `unique`

データベース上で一意の値かどうかを確認します。

`unique:users,email`のうち、`users`はテーブル名を、`email`はカラム名を表します。

わざわざ自分でデータベース上の値を確認せずに済むので便利ですね。


### `Password`

`Password::defaults`を使うことでアプリケーションに登録されたデフォルトのパスワードバリデーションを使用することができます。

サービスプロバイダで登録すればカスタマイズできますが、今回はデフォルトのまま使用しています。

デフォルトでは「**文字が最低で８文字以上含まれるか**」をチェックします。


### `same`

リクエストで送られたデータの内のどれかと同じ値であることを確認します。

ここまでシチュエーションの限られた処理を用意してくれていることに驚きです。

`same:password`とすることで、`password`というキーの値と同一であるかを確認しています。


<br>

次はログイン時のリクエストです。

```php:/laravel-app/app/Http/Requests/User/CreateRequest.php
<?php

namespace App\Http\Requests\User;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rules\Password;

class LoginRequest extends FormRequest
{
    /**
     * Determine if the user is authorized to make this request.
     */
    public function authorize(): bool
    {
        return true;
    }

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array<string, \Illuminate\Contracts\Validation\ValidationRule|array<mixed>|string>
     */
    public function rules(): array
    {
        return [
            'email' => ['required', 'email:rfc,dns'],
            'password' => ['required', 'string'],
        ];
    }

    public function messages()
    {
        return [
            'email.required' => 'メールアドレスは必ず入力してください',
            'email.email' => '不正なフォーマットです',
            'password.required' => 'パスワードは必ず入力してください',
        ];
    }
}
```

攻撃者にパスワードのルールを悟らせないよう、あえて`string`のみにしています。


## アカウント登録・ログインページの編集

アカウント登録、ログインページともに現状のままでは`FormRequest`のバリデーションに引っかかってしまいます。

ユーザー登録ページを次のように編集します。

```diff php:/laravel-app/resources/views/signup.blade.php
    <div class="mt-9 flex gap-6 flex-col">
-       <input type="text" placeholder="ユーザー名" autocomplete="off" class="p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500">
-       <input type="email" placeholder="メールアドレス" autocomplete="off" class="p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500">
-       <input type="password" placeholder="パスワード" autocomplete="off" class="p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500">
-       <input type="password" placeholder="パスワード（確認用）" autocomplete="off" class="p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500">
+       <input type="text" name="username" value="{{ @old('username') }}" placeholder="ユーザー名" autocomplete="off" class="p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500">
+       @error('username')
+           <p class="text-sm text-red-400">{{ $message }}</p>
+       @enderror
+       <input type="email" name="email" value="{{ @old('email') }}" placeholder="メールアドレス" autocomplete="off" class="p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500">
+       @error('email')
+           <p class="text-sm text-red-400">{{ $message }}</p>
+       @enderror
+       <input type="password" name="password" value="{{ @old('password') }}" placeholder="パスワード" autocomplete="off" class="p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500">
+       @error('password')
+           <p class="text-sm text-red-400">{{ $message }}</p>
+       @enderror
+       <input type="password" name="password_confirmed" value="{{ @old('password_confirmed') }}" placeholder="パスワード（確認用）" autocomplete="off" class="p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500">
+       @error('password_confirmed')
+           <p class="text-sm text-red-400">{{ $message }}</p>
+       @enderror
    </div>
```

`@old`でエラー時にリクエスト直前の入力を保持するようにしました。

さらにエラーメッセージの表示を加えています。

<br>

ログインページも似たような感じで編集します。

```diff php
    <form action="{{ route('auth.login') }}" method="POST">
        @csrf
        <h2 class="text-xl font-semibold tracking-wider">
            Login
        </h2>
        <div class="w-full h-[1px] mt-2 bg-gray-300"></div>
        <div class="mt-9 flex gap-6 flex-col">
-           <input type="email" placeholder="メールアドレス" autocomplete="off" class="p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500">
-           <input type="password" placeholder="パスワード" autocomplete="off" class="p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500">
+          @error('login')
+              <p class="text-sm text-red-400">{{ $message }}</p>
+          @enderror
+          <input type="email" name="email" value="{{ @old('email') }}" placeholder="メールアドレス" autocomplete="off" class="p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500">
+          @error('email')
+              <p class="text-sm text-red-400">{{ $message }}</p>
+          @enderror
+          <input type="password" name="password" placeholder="パスワード" autocomplete="off" class="p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500">
+          @error('password')
+              <p class="text-sm text-red-400">{{ $message }}</p>
+          @enderror
        </div>
        <div class="mt-10 flex justify-end">
            <button type="submit" class="p-3 text-sm text-white bg-blue-500 rounded-sm shadow-md shadow-gray-300">
                ログイン
            </button>
        </div>
    </form>
```


## コントローラーの実装

新しくコントローラーを作成します。

```bash
$ php artisan make:controller User/CreateController

   INFO  Controller [app/Http/Controllers/User/CreateController.php] created successfully.  

$ php artisan make:controller Auth/LoginController

   INFO  Controller [app/Http/Controllers/Auth/LoginController.php] created successfully.  
```

最初にユーザーを作成するコントローラーの処理を実装します。

```diff php:/laravel-app/app/Http/Controllers/User/CreateController.php
+   use App\Http\Requests\User\CreateRequest;
+   use Illuminate\Support\Facades\Auth;
+   use Illuminate\Support\Facades\DB;
+   use Illuminate\Support\Facades\Hash;
    ...
-      public function __invoke()
+      public function __invoke(CreateRequest $request)
       {
+          $data = $request->validated();
+      
+          DB::table('users')
+              ->insert([
+                  'name' => $data['username'],
+                  'email' => $data['email'],
+                  'password' => Hash::make($data['password']),
+              ]);
+      
+          $logined = Auth::attempt([
+              'email' => $data['email'],
+              'password' => $data['password'],
+          ]);
+      
+          if ($logined === true) {
+      
+              return redirect()->route('todo.index');
+          }
+      
+          return redirect()->route('signup');
    }
```

ユーザーを新規で追加する時に`Hash::make`でパスワードをハッシュ化して登録しています。

登録後、`Auth::attempt`でログインを行い、成功時にはToDo一覧ページにリダイレクトします。

<br>

このままログイン処理も実装しましょう。

```diff php:/laravel-app/app/Http/Controllers/User/LoginController.php
-   public function __invoke()
+   public function __invoke(LoginRequest $request)
    {
+       $data = $request->validated();
+   
+       $logined = Auth::attempt([
+           'email' => $data['email'],
+           'password' => $data['password'],
+       ]);
+   
+       if ($logined === true) {
+   
+           $request->session()->regenerate();
+   
+           return redirect()->intended(route('todo.index'));
+       }
+   
+       return back()->withErrors([
+           'login' => 'メールアドレスまたはパスワードが間違っています',
+       ]);
    }
```

ユーザー登録時と同じように`Auth::attempt`でログインを試みています。

ログイン成功時にはユーザーが元々訪れようとしていたページに遷移させます。初めからログインページにやってきたユーザーはデフォルトでToDo一覧ページに遷移させています。

ついでにセキュリティのためにセッショントークンを再生成します。

ログイン失敗時はエラーメッセージと共にログインページに戻します。`wthErrors`に渡した配列のキーを`@error`ディレクティブに渡すことでエラーメッセージを取得することが可能です。

<br>

最後は新しくルートを定義してコントローラーを適用します。

```diff php
+   Route::prefix('/auth')
+       ->as('auth.')
+       ->group(function () {
+   
+           Route::post('/login', Auth\LoginController::class)->name('login');
+       });

+   Route::prefix('/user')
+       ->as('user.')
+       ->group(function() {
+   
+           Route::post('/create', User\CreateController::class)->name('create');
+       });

    Route::prefix('/todo')
        ->as('todo.')
        ->group(function () {
    ...
```


# 認証用ミドルウェアを適用する

認証関係の処理ができたので、次は各ルートに認証を適用していきます。

とは言っても加えるのは一行です。

`web.php`を開いて次のように変更してください。

```diff php:
    Route::prefix('/todo')
        ->as('todo.')
+       ->middleware('auth')
        ->group(function () {

            Route::get('/', Todo\IndexController::class)->name('index');

            Route::get('/new', Todo\NewController::class)->name('new');

            Route::get('/edit/{id}', Todo\EditController::class)->name('edit');

            Route::post('/create', Todo\CreateController::class)->name('create');

            Route::put('/update', Todo\UpdateController::class)->name('update');

            Route::delete('/delete', Todo\DeleteController::class)->name('delete');
        });
```

これで`/todo`以降のルートはログイン後でないとアクセスできないようになりました。

`middleware`メソッドを使うことでリクエストからコントローラーまでの間に処理を挟むことができます。

`auth`で指定しているのはLaravelがデフォルトで用意している認証関係のミドルウェエアです。

もちろんミドルウェアは自身で実装して適用することも可能です。

今回はルートのグループ全体に適用しましたが、必要であればルートごとにも設定可能です。


# 認証状況に応じたページの変更

ここまでくれば認証処理はほぼほぼ終わったようなものです。ちゃんと動くかどうかご自身で確かめてみてください。

ここからはまだたりてない機能を追加していきます。

お気づきかもしれませんが、まだログアウトする処理が実装できていません。加えて、ヘッダーのナビゲーションに認証後にしか訪問できないページへのリンクが表示されてしまっています。

どうせなら認証状況に応じてヘッダーの見た目を変えられるようにしたいところです。

ログインしていない、またはアカウントを作っていないユーザーにはログインとアカウント登録のリンクを見せるようにして、ログインしているユーザーにはToDo関連のリンクを見せるようにしましょう。


## ログアウト処理の実装

ヘッダーをいじる前にログアウト用の処理を実装しましょう。

また`FormRequest`か、と思うかもしれませんが、今回は必要ありません。特定のURLにGETメソッドでアクセスするだけでログアウトできるようにします。

<br>

ログアウト用のコントローラーを作成します。お馴染みの`php artisan`です。

```bash:/laravel-app
$ php artisan make:controller LogoutController

   INFO  Controller [app/Http/Controllers/LogoutController.php] created successfully. 
```

作成できたら中身を次のように編集してください。

```php:/laravel-app/app/Http/Controllers/Auth/LogoutController.php
<?php

namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\Auth;
use Illuminate\Http\Request;

class LogoutController extends Controller
{
    public function __invoke(Request $request)
    {
        Auth::logout();

        $request->session()->invalidate();

        $request->session()->regenerateToken();

        return redirect()->route('login');
    }
}
```

`invalidate`でセッションIDを更新し、`regenerateToken`でCSRFトークンを再発行しています。

ログアウトは`Auth::logout`を実行するだけです。


## ヘッダーの編集

次にヘッダーのナビゲーションを認証状況に応じて変更するようにしましょう。

```diff php:/laravel-app/resources/views/common/header.blade.php
<header class="fixed top-0 left-0 bg-inherit z-999">
    <div class="w-screen h-header pl-5 pr-5 flex justify-between items-center bg-test">
        <div>
            <a href="{{ route('home') }}">
                <h1 class="text-3xl font-bold tracking-widest">{{ env('APP_NAME') }}</h1>
            </a>
        </div>

        <div class="flex gap-2">
-           <x-link href="{{ route('todo.new') }}">
-               <x-google-icon name="add" class="text-2xl"/>
-           </x-link>
-           <x-link href="{{ route('todo.index') }}">
-               <x-google-icon name="list" class="text-2xl"/>
-           </x-link>

+           @if (Route::is('home'))
+               <x-link href="{{ route('login') }}">
+                   <p class="text-xs">ログイン</p>
+               </x-link>
+               <x-link href="{{ route('signup') }}">
+                   <p class="text-xs">新規登録</p>
+               </x-link>
+           @else
+               @auth
+                   <x-link href="{{ route('todo.new') }}">
+                       <x-google-icon name="add" class="text-2xl"/>
+                   </x-link>
+                   <x-link href="{{ route('todo.index') }}">
+                       <x-google-icon name="list" class="text-2xl"/>
+                   </x-link>
+                   <x-link href="{{ route('auth.logout') }}">
+                       <p class="text-xs">ログアウト</p>
+                   </x-link>
+               @endauth
+           @endif

        </div>
    </div>
</header>
```

`@auth`という新しいディレクティブが登場しました。このディレクティブを使うと認証済みの場合だけ表示されるようになります。

これでToDO関連のナビゲーションは認証済みの場合だけ表示されるようになります。

ログインやアカウント登録画面へのリンクは`Route::is`でホーム画面の場合のみ描画するようにしています。


# Authファサードのその他の利用方法

## リクエスト時の認証

`FormRequest`を作成した際、`authorize`メソッドで`true`を返していたのを覚えているでしょうか。

認証が実装できたので、この部分も変えていこうと思います。

認証が必要になるのはToDO関連のリクエストです。

`CreateRequest`、`UpdateRequest`、`DeleteRequest`を次のように変更します。

```diff php://laravel-app/app/Http/Requests/Todo/~Request.php
+   use Illuminate\Support\Facades\Auth;
    ...
    public function authorize(): bool
    {
-       return true;
+       return Auth::check();
    }
```

`Auth::check`は認証が完了している場合に`true`を返します。

認証がされていないユーザーからのリクエストはこれで弾くことができます。


## ユーザーIDの取得

ここまではずっとユーザーIDが`1`であることを前提に実装してきました。

ですが認証を実装すると`Auth::id`で簡単にログイン中のユーザーのIDが取得できます。

これをToDoの処理で使おうと思います。

```diff php:/laravel-app/app/Http/Controllers/Todo/IndexController.php
+   use Illuminate\Support\Facades\Auth;
    ...
    public function __invoke()
    {
        $todos = $this->repository
-           ->list(1)
+           ->list(Auth::id())
            ->sortBy('deadline');

        return view('todo.index', ['todos' => $todos]);
    }
```

```diff php:/laravel-app/app/Http/Controllers/Todo/CreateController.php
+   use Illuminate\Support\Facades\Auth;
    ...
    public function __invoke(CreateRequest $request)
    {
        $data = $request->validated();

        $date = $data['date'];
        $time = $data['time'];

        $datetime = null;

        if ($date !== null && $time !== null) {

            $datetime = "{$date} {$time}";
        }

        $this->repository->add([
            'title' => $data['title'],
            'memo' => $data['memo'],
            'deadline' => $datetime,
            'color' => $data['color'],
-           'user_id' => 1,
+           'user_id' => Auth::id(),
        ]);

        return redirect()->route('todo.index');
    }
```

```diff php:/laravel-app/app/Repositories/TodoRepository.php
+   use Illuminate\Support\Facades\Auth;
    ...
    public function find(int $id): \stdClass|null
    {
        return DB::table($this->table)
            ->where('id', $id)
+           ->where('user_id', Auth::id())
            ->first();
    }

    public function list(int $userId): Collection
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
+           ->where('user_id', Auth::id())
            ->update((array) $data);
    }

    public function delete(int $id): void
    {
-       DB::table($this->table)->delete($id);
+       DB::table($this->table)
+           ->where('user_id', Auth::id())
+           ->delete($id);
    }
```

これでユーザーIDが分からなければデータの追加・更新・削除ができなくなりました。


# Laravelの認証の仕組み

