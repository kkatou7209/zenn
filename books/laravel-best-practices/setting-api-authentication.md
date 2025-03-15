---
title: "API認証の設定"
---

# 概要

本章ではAPIリクエストの認証を行います。

Webの認証はセッションを使いましたが、APIではトークンを使います。


## 本章で学ぶこと

- APIの認証


# APIトークン

今回使用するSanctumはAPIトークンで認証を行います。

APIトークンはリクエストの`Authorization`ヘッダーに格納し、Sanctumはヘッダーのトークンを見てリクエストの妥当性を検証します。

まずはAPIトークンが生成できるように変更を行いましょう。

`User`モデルを次のように変更してください。

```diff php:/laravel-app/app/Models/User.php
    ...
+   use Laravel\Sanctum\HasApiTokens;
    ...
    class User extends Authenticatable
    {
        /** @use HasFactory<\Database\Factories\UserFactory> */
-       use HasFactory, Notifiable;
+       use HasFactory, Notifiable, HasApiTokens;
    ...
```

`HasApiTokens`というトレイトを追加しました。

`HasApiTokens`はAPIトークンを扱う際に便利な機能を提供してくれるSanctum用のトレイトです。これで`User`モデル経由でAPIトークンの作成が行えます。


## CookieへのAPIトークン設定

APIトークンの作成が行えるようになったので、次はこのトークンをCookieに設定できるようにします。

ログイン時にトークンをCookieに格納し、ログアウト時はCookieから削除するようにしましょう。

```diff php:/laravel-app/app/Http/Controllers/User/CreateController.php
    ...
    public function __invoke(CreateRequest $request)
    {
        $data = $request->validated();

        DB::table('users')
            ->insert([
                'name' => $data['username'],
                'email' => $data['email'],
                'password' => Hash::make($data['password']),
            ]);

        $logined = Auth::attempt([
            'email' => $data['email'],
            'password' => $data['password'],
        ]);

        if ($logined === true) {

+           /**
+            * @var \App\Models\User $user
+            */
+           $user = Auth::user();
+   
+           $token = $user->createToken('api-token');
+   
+           $cookie = cookie('API_TOKEN', $token->plainTextToken, sameSite: 'Strict', httpOnly: true, );

-           return redirect()->route('todo.index');
+           return redirect()
+               ->route('todo.index')
+               ->withCookie($cookie);
        }

        return redirect()->route('auth.login',);
    }
```

```diff php:/laravel-app/app/Http/Controllers/Auth/LoginController.php
    ...
    public function __invoke(LoginRequest $request)
    {
        $data = $request->validated();

        $logined = Auth::attempt([
            'email' => $data['email'],
            'password' => $data['password'],
        ]);

        if ($logined === true) {

+           /**
+               * @var \App\Models\User $user
+               */
+           $user = Auth::user();
+   
+           $token = $user->createToken('api-token');
+   
+           $cookie = cookie('API_TOKEN', $token->plainTextToken, sameSite: 'Strict', httpOnly: true, );

            $request->session()->regenerate();

-           return redirect()->intended(route('todo.index'));
+           return redirect()
+               ->intended(route('todo.index'))
+               ->withCookie($cookie);
        }

        return back()->withErrors([
            'login' => 'メールアドレスまたはパスワードが間違っています',
        ]);
    }
```

```diff php:/laravel-app/app/Http/Controllers/Auth/LogoutController.php
+   use Illuminate\Support\Facades\Cookie;
    ...
    public function __invoke(Request $request)
    {
+       /**
+        * @var \App\Models\User $user
+        */
+       $user = Auth::user();
+
+       $user->tokens()->delete();

        Auth::logout();

        $request->session()->invalidate();

        $request->session()->regenerateToken();

+       $cookie = Cookie::forget('API_TOKEN');

-       return redirect()->route('login');
+       return redirect()
+           ->route('login')
+           ->withCookie($cookie);
    }
```

ログイン時には`HasApiTokens`のメソッドである`createToken`を使ってトークンを生成し、ハッシュ化前の文字列をCookieに設定しています。

ログアウト時は反対に`tokens()->delete()`でデータベース上に登録されたトークンを削除し、Cookie上のトークンも消しています。

ユーザーの変数に`/** @var \App\Models\User $user */`というコメントをつけているのは、Intelepheseのエラーを抑制するためです。


## AuthorizationヘッダーへのAPIトークンの設定

APIトークンの生成とCookieへの割り当ては以上で完了ですが、Cookieにトークンを入れるだけでSanctumが認証できるわけではありません。

トークンの認証を行うためには`Authorization`ヘッダーにトークンを割り当てる必要があります。

しかしCookieの属性に`HttpOnly: true`を設定したので、JavaScriptではCookieからAPIトークンを設定することができません。

ではどうするかというと、ミドルウェアを使ってリクエストの途中でヘッダーとトークンをくっつけます。


### ミドルウェアの作成

ここまで何度かミドルウェアの機能を利用してきましたが、実際に作るのは初めてですね。

ミドルウェアもコマンドから作成することができます。

```bash:/laravel-app
$ php artisan make:middleware AssignApiToken

   INFO  Middleware [app/Http/Middleware/AssignApiToken.php] created successfully.
```

`handle`というメソッドだけを持ったクラスが作成されます。

このメソッドの中に`Authorization`ヘッダーを設定する処理を追加します。

```php:/laravel-app/app/Http/Middleware/AssignApiToken.php
    ...
    public function handle(Request $request, Closure $next): Response
    {
        $header = $request->header('Authorization');

        $token = $request->cookie('API_TOKEN');

        if (empty($header) === true && $token !== null) {

            $request->headers->set('Authorization', "Bearer {$token}");
        }

        return $next($request);
    }
```

`Authorization`ヘッダーに値が設定されておらず、なおかつCookieに`API_TOKEN`が設定されている場合にのみヘッダーにトークンを設定しています。

リクエストのヘッダーは`header`メソッドで、Cookieは`cookie`メソッドで取得できます。

なおCookieはデフォルトで全てハッシュ化されてしまうので、APIトークンの値がハッシュ化されてしまわないように、`API_TOKEN`という名前のCookieはハッシュ化から除外します。

`AppServiceProvider`に追記してください。

```diff php:/laravel-app/app/Providers/AppServiceProvider.php
+   use Illuminate\Cookie\Middleware\EncryptCookies;
    ...
    public function boot(): void
    {
-       //
+       EncryptCookies::except('API_TOKEN');
    }
```


### ミドルウェアの使用

認証の実装はできたので、ミドルウェアを設定しましょう。

`bootstarap/app.php`で`AssinApiToke`ミドルウェアを登録します。

```diff php:/laravel-app/bootstrap/app.php
    ...
    ->withMiddleware(function (Middleware $middleware) {
-       //
+       $middleware->prepend([
+           AssignApiToken::class,
+       ]);
    })
    ...
```

そして`api.php`を次のように変更します。

```diff php:/laravel-app/routes/api.php
    ...
-   Route::as('api.')->group(function() {
+   Route::as('api.')
+       ->middleware('auth:sanctum')
+       ->group(function() {

        Route::prefix('/todo')
            ->as('todo.')
            ->middleware('throttle:50,1')
            ->group(function() {

                Route::put('/toggle', ToggleController::class)->name('toggle');
            });
    });
```

# まとめ

APIの認証はSanctumを使ってトークン認証を実装しました。

その際、生成したトークンをCookieに設定し、リクエストと一緒に渡してもらうようにしました。

今回はミドルウェアを使って`Authorization`ヘッダーを設定しましたが、APIを使ってトークンを受け取り、それをヘッダーに設定する方法もあります。

