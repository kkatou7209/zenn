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

APIでも`FormRequest`が使えます（`Form`という名前がついていますが）。

`ToggleRequest`という名前の`FormRequest`を作りましょう。

```bash
$ php artisan make:request Todo/ToggleRequest

   INFO  Request [app/Http/Requests/Todo/ToggleRequest.php] created successfully.  
```

中身は以下のようにします。

```php:/laravel-app/app/Http/Requests/Todo/ToggleRequest.php
...
class ToggleRequest extends FormRequest
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
            'id' => ['required', 'integer'],
            'done' => ['required', 'boolean'],
        ];
    }
}
```


## コントローラーの作成

APIを実装する場合でも`web.php`の場合に習ってコントローラーを作りましょう。

今回も`__invoke`メソッドに処理を記述します（一度始めたことは最後まで一貫しましょう）。

それではいつものようにコントローラーを作成するコマンドを実行しましょう。

```php:/laravel-app
$ php artisan make:controller Api/Todo/ToggleController

   INFO  Controller [app/Http/Controllers/Api/Todo/ToggleController.php] created successfully.
```

作成したコントローラーを以下のように変更します。

```php:/laravel-app/app/Http/Controllers/Api/Todo/ToggleController.php
<?php

namespace App\Http\Controllers\Api\Todo;

use App\Http\Controllers\Controller;
use App\Http\Requests\Todo\ToggleRequest;
use App\Repositories\Repository;
use Illuminate\Http\Request;
use Illuminate\Http\Response;

class ToggleController extends Controller
{
    public function __construct(
        protected Repository $repository,
    )
    {}

    public function __invoke(ToggleRequest $request)
    {
        $data = $request->validated();

        $this->repository->update($data['id'], [
            'done' => $data['done'],
        ]);

        return response(status: Response::HTTP_OK);
    }
}
```

リポジトリも使っているので`RepsitoryServiceProvider`にこのコントローラーを追加しましょう。

```diff php:/laravel-app/app/Providers/RepositoryServiceProvider.php
    public function register(): void
    {
        $this->app->when([
                Todo\IndexController::class,
                Todo\NewController::class,
                Todo\EditController::class,
                Todo\IndexController::class,
                Todo\CreateController::class,
                Todo\UpdateController::class,
                Todo\DeleteController::class,
+               Api\Todo\ToggleController::class,
            ])
            ->needs(Repository::class)
            ->give(fn() => new TodoRepository());
    }

```

## ルートへのコントローラーの設定

`api.web`で先ほど作った`ToggleController`を設定します。

メソッドは`PUT`にします。

```diff php:/laravel-app/routes/api.php
   Route::prefix('/todo')
        ->as('todo.')
        ->group(function() {

-           Route::get('/toggle', function() {
-  
-                 return ['test' => 'API TEST'];
-           })->name('toggle');
+           Route::put('/toggle', ToggleController::class)->name('toggle');
        });
```

## ミドルウェアによるレート制限

過剰なリクエストに対する対策としてレート制限を設けます。

今回は1分間あたり`50`リクエストまで抑えます。

`routes/api.php`を次のように編集してください。

```diff php:/laravel-app/routes/api.php
    ...
        Route::prefix('/todo')
            ->as('todo.')
+           ->middleware('throttle:50,1')
            ->group(function() {

                Route::put('/toggle', ToggleController::class)->name('toggle');
            });
    ...
```

`middlware`メソッドはリクエストとレスポンスの間に処理を挟ませることができるメソッドです。

コールバック関数を渡すこともできますが、今回は`ThrottleRequests`という元々備わっているミドルウェアを利用しました。


# Axiosを使ったリクエスト

ここからはToDoの完了・未完了の切り替えをJavaScriptを使って実装します。

APIへのリクエストは**Axios**を使います。

Axiosはデフォルトで`package.json`に含まれているので`npm i`を実行するだけでインストールできます。


## JavaScriptファイルの読み込み

`/api/todo/toggle`へのリクエストは一覧ページで行います。

タイミングとしてはチェックボックスのチェック状態が変わった時にリクエストをかけます。

`resources/ts/todo`配下に`toggle.ts`を追加して読み込みます。

:::message
実際の開発ではチェックボックスが変わるたびにリクエストをかけるのは得策ではありません。

サーバーに負荷がかかりますし、その負荷を利用した攻撃の危険もあります。

代わりに、画面が遷移する前に差分だけまとめてリクエストする、変更箇所が一定数に達したらリクエストする、などの方法があります。
:::

### `@stack`ディレクティブと`@push`ディレクティブ

JavaScriptを読み込ませるために新しいディレクティブを導入します。

ますはレイアウトの編集からです。`<head>`タグに一行追加します。

```diff php:/laravel-app/resources/views/layout.blade.php
   <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <meta http-equiv="X-UA-Compatible" content="ie=edge">
      <link href="https://fonts.googleapis.com/css2?family=Material+Icons+Round" rel="stylesheet">
      <link rel="preconnect" href="https://fonts.googleapis.com">
      <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
      <link href="https://fonts.googleapis.com/css2?family=M+PLUS+1:wght@100..900&family=Montserrat:ital,wght@0,100..900;1,100..900&family=Murecho:wght@100..900&family=Noto+Sans+JP:wght@100..900&display=swap" rel="stylesheet">
      @vite(['resources/ts/app.ts'])
+     @stack('scripts')
      <title>{{ env('APP_NAME') }}</title>
   </head>
```

`@stack`の場所に`@push`あるいは`@pushOnce`を使うことで要素を追加することができます。

`@pushOnce`はその名の通り、繰り返し処理の中で使用しても一度だけ実行することを保証してくれます。

一覧ページのBladeで`@push`を使ってJavaScriptを読み込ませましょう。

```diff php:/laravel-app/resources/views/todo/index.blade.php
   @use(Illuminate\Support\Carbon)

+  @push('scripts')
+     @vite(['resources/ts/todo/toggle.ts'])
+  @endpush
```


### JavaScriptの実装

現状のままではチェックボックスの状態を取得できないので、`<input>`タグにクラス名を追加し、追加の属性を設定しましょう。

```diff php
   <div class="mt-2">
-     <input type="checkbox" @checked($todo->done) class="w-[25px] h-[25px]">
+     <input type="checkbox" @checked($todo->done) class="todo-done w-[25px] h-[25px]" todo-id="{{ $todo->id }}">
   </div>
```

`class`に`todo-done`を追加し、`todo-id`という属性を設けてそこにToDoのIDを渡すように変更しました。

自分で定義した属性は`HTMLElement`の`getAttribute`メソッドで取得できます。

`toggle.ts`を編集しましょう。

```ts:/laravel-app/resources/ts/todo/toggle.ts
import axios from 'axios';

const toggle = async (id: number, done: boolean): Promise<number> => {

    const res = await axios.put('/api/todo/toggle', { id, done, }, {
        headers: {
            "Content-Type": 'application/json; charset=utf-8',
        },
        withCredentials: true,
        withXSRFToken: true,
    });

    return res.status;
}

const onCheckChange = async (event: Event): Promise<void> => {

    if (event.type !== 'change') {

        throw Error('changeイベントに設定してください');
    }

    if (event.target instanceof HTMLInputElement === false) {

        throw Error('input要素ではありません');
    }

    if (
        event.target.hasAttribute('todo-id') === false ||
        event.target.getAttribute('todo-id') === null
    ) {
        throw Error('todo-id属性が設定されていません');
    }

    const attrId = event.target.getAttribute('todo-id')!;

    if (/^[1-9]+\d*$/.test(attrId) === false) {

        throw Error(`todo-id属性の値が不正です todo-id: ${attrId}`);
    }

    const id = Number.parseInt(attrId);

    const done = event.target.checked;

    const status = await toggle(id, done);

    if (status < 200 ||  300 <= status) {

        alert('更新に失敗しました。通信状況を見てもう一度お試しください');

        event.target.checked = !done;

        if (500 <= status) {

            console.error(`サーバーエラーが発生しました\nステータス: ${status}`);
        }

        return;
    }
}

window.addEventListener('DOMContentLoaded', () => {

    const todoCheckBoxes = document.querySelectorAll<HTMLInputElement>('.todo-done');

    for (const checkbox of todoCheckBoxes) {

        checkbox.addEventListener('change', onCheckChange);
    }
});
```

# まとめ

APIを実装するためにパッケージを追加しましたが、実装の仕方はページを返す場合とほとんど変わりません。

返却するデータがない場合は明示的にステータスコードを返すようにしましょう。そうするとフロントエンド側で状況に応じた処理を実装することができます。

ここで使った`@stack`ディレクティブは、`@push`などを使って柔軟にJavaScriptファイルを読み込ませることができます。