---
title: "ページの作成"
---

# 概要

## 本章で学ぶこと

- レイアウトの作成
- コンポーネントの作成

本章では画面の実装を行っていきます。

今回はVueなどのSPAフレームワークは使用せず、**Blade**だけで実装していきます。

Bladeで画面を作る際は共通化を意識しましょう（プログラミング全般に言えることですが）。

Bladeにはコンポーネント化の機能が備わっています。何度も使い回すような要素はコンポーネント化しておくのが定石です。

コンポーネント以外でもページに別のBladeファイルを埋め込むようなこともできます。こちらは共通レイアウトを作る際に便利です。

スタイリングはデフォルトで利用できる**TailwindCSS**を使います。

# レイアウト

共通の表示をレイアウトとしてまとめておけば開発効率が上がります。

同じ内容を何度もコピペしなくても済みますし、変更箇所があったときは１箇所変更するだけで全体に適応できることもあります。

共通化は初めに全てやってしまうのではなく、適宜必要と判断されれば実行する、くらいの意識で構いません。個別のケースで対応しなければならないこともあるので、過剰に共通化してしまうと後々大変になってしまうこともあります。

## 共通レイアウトの作成

まずはアプリケーション全体の共通レイアウトを作っていきましょう。

```bash:/laravel-app/resources/views/layout.blade.php
$ php artisan make:view layout

   INFO  View [resources/views/layout.blade.php] created successfully.  
```

中身は以下のようにします。

```php:/laravel-app/resources/views/layout.blade.php
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    @vite(['resources/ts/app.ts'])
    <title>{{ env('APP_NAME') }}</title>
</head>
<body class="h-dvh w-screen bg-white [&>:box-border]">

    <main class="w-full h-full pt-header">
        @yield('main')
    </main>

</body>
</html>
```

`app.css`を以下のように変更します。

```css:/laravel-app/resources/css/app.css
@import "tailwindcss";
@source "../views";

@theme {
    --spacing-header: 10vh;
}
```

`@yield`とした部分に他のBladeを埋め込んで利用します。

`@vite`の部分ではViteがトランスパイルしたTypeScriptを読み込んでいます。

開発環境でTypeScriptの読み込みとホットリロードを有効化するにはViteサーバーを起動します。

```bash:/laravel-app
$ bun run dev
$ vite

  VITE v6.1.0  ready in 170 ms

  ➜  Local:   http://127.0.0.1:5173/
  ➜  press h + enter to show help

  LARAVEL v11.41.3  plugin v1.2.0
```

## `@extends`と`@section`：レイアウトの利用

試しにホーム画面にこのレイアウトを適用してみましょう。

`home.blade.php`を以下のように変更してください。

```php:/laravel-app/resources/views/home.blade.php
@extends('layout')

@section('main')
<div>
    <h1>ホーム画面</h1>
</div>
@endsection
```

`localhost:8000`を確認してみましょう。

タブの表記が`Laravel`になり、TailwindCSSが読み込まれて表示が変更されているはずです。

![](/images/making-page_1.png =250x)
*`localhost:8000`*

同じように他の画面にもレイアウトを適用してください。

```php:/laravel-app/resources/views/login.blade.php
@extends('layout')

@section('main')
<div>
    <h1>ログイン画面</h1>
</div>
@endsection
```

```php:/laravel-app/resources/views/signup.blade.php
@extends('layout')

@section('main')
<div>
    <h1>アカウント登録画面</h1>
</div>
@endsection
```

```php:/laravel-app/resources/views/todo/index.blade.php
@extends('layout')

@section('main')
<div>
    <h1>ToDo一覧</h1>
    <ul>
        @foreach ($todos as $todo)
            <li>{{ $todo->title }}</li>
        @endforeach
    </ul>
</div>
@endsection
```

```php:/laravel-app/resources/views/todo/new.blade.php
@php
    use App\Enums\Color;
@endphp

@extends('layout')

@section('main')
<div>
    <h1>ToDo登録</h1>
    <ul>
        @foreach (Color::cases() as $color)
            <li>{{ $color->display() }}</li>
        @endforeach
    </ul>
</div>
@endsection
```

```php:/laravel-app/resources/views/todo/edit.blade.php
@extends('layout')

@section('main')
<div>
    <h1>ToDo編集</h1>
</div>
@endsection
```

## `@include`：Bladeの読み込み

次にヘッダーを作ってみましょう。

```bash:/laravel-app
$ php artisan make:view common/header

   INFO  View [resources/views/common/header.blade.php] created successfully.
```

アイコンを使いたいので[Google Icons](https://developers.google.com/fonts/docs/material_icons?hl=ja)を導入しましょう。

ついでに[Google Fonts](https://fonts.google.com/)も使っちゃいます。

`layout.blade.php`にリンクを追加してください。

```diff php
    ...
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
+       <link href="https://fonts.googleapis.com/css2?family=Material+Icons+Round" rel="stylesheet">
+       <link rel="preconnect" href="https://fonts.googleapis.com">
+       <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
+       <link href="https://fonts.googleapis.com/css2?family=M+PLUS+1:wght@100..900&family=Montserrat:ital,wght@0,100..900;1,100..900&family=Murecho:wght@100..900&family=Noto+Sans+JP:wght@100..900&display=swap" rel="stylesheet">
        @vite(['resources/ts/app.ts'])
        <title>{{ env('APP_NAME') }}</title>
    </head>
    ...
```

フォントを適用するのに`app.css`を編集します。

```diff css:/laravel-app/resources/css/app.css
    @import "tailwindcss";
    @source "../views";

    @theme {
+       --font-sans: "M PLUS 1", "Noto Sans JP", serif;
        --spacing-header: 10vh;
    }
```

フォントが適用されているか一度確認してみてください。

改めてヘッダーの作成を続けましょう。

`header.blade.php`の中身を以下のように変更します。

```php:/laravel-app/resources/views/common/header.blade.php
<header class="fixed top-0 left-0 bg-inherit z-000">
    <div class="w-screen h-header pl-5 pr-5 flex justify-between items-center">
        <div>
            <a href="{{ route('home') }}">
                <h1 class="text-3xl font-bold">{{ env('APP_NAME') }}</h1>
            </a>
        </div>

        <div class="flex gap-2">
            <a
                class="transition duration-300 ease-in-out hover:scale-[1.2] hover:text-blue-500"
                href="{{ route('todo.new') }}"
            >
                <span class="material-icons-round text-2xl">add</span>
            </a>
            <a
                class="transition duration-300 ease-in-out hover:scale-[1.2] hover:text-blue-500"
                href="{{ route('todo.index') }}"
            >
                <span class="material-icons-round text-2xl">list</span>
            </a>
        </div>
    </div>
</header>
```

`a`タグで使っている`route`関数はリンクを生成してくれるLaravelの組み込み関数です。ルート名を指定することで`web.php`で指定したルートからURLを生成してくれます。

<br>

共通ヘッダーが出来上がったので共通レイアウトで読み込みたいと思います。

`layout.blade.php`を以下のように変更します。

```diff php:/laravel-app/resources/views/layout.blade.php
    ...
    <body class="h-dvh w-screen bg-white [&>:box-border]">

+       @include('common.header')

        <main class="w-full h-full pt-header">
            @yield('main')
        </main>

    </body>
    ...
```

`@include`を使って共通Bladeを読み込んでいます。引数にはview名を指定します。

変更が終わったらブラウザで表示を確認してみてください。

レイアウトを使っているすべてのページでヘッダーが適用されておりことが確認できるはずです。

![](/images/making-page_2.png)
*`http://localhost:8000`*

# コンポーネント

Bladeにはコンポーネント化機能があります。使い回す部品はコンポーネントとして使いまわしましょう。

先ほど作ったヘッダーにも繰り返し同じ処理をしている箇所があります。アイコンを指定している箇所です。

アイコンは何度も使うことになりそうなので、早めに共通化してしまいましょう。

コンポーネントを作るには以下のコマンドを実行します。

```bash:/laravel-app
$ php artisan make:component GoogleIcon

   INFO  Component [app/View/Components/GoogleIcon.php] created successfully.  

   INFO  View [resources/views/components/google-icon.blade.php] created successfully. 
```

ViewファイルとComponentファイルが作成されました。

まずはViewファイルから編集します。

```php:/laravel-app/resources/views/components/google-icon.blade.php
<span
    {{
        $attributes->merge([
            'class' => 'material-icons-round'
        ])
    }}
>
    {{ $name }}
</span>
```

Componentファイルは次のようにします。

```diff php:/laravel-app/app/View/Components/GoogleIcon.php
<?php

    namespace App\View\Components;

    use Closure;
    use Illuminate\Contracts\View\View;
    use Illuminate\View\Component;

    class GoogleIcon extends Component
    {
        /**
        * Create a new component instance.
        */
        public function __construct(
+           public string $name,
        )
        {
            //
        }

        /**
        * Get the view / contents that represent the component.
        */
        public function render(): View|Closure|string
        {
            return view('components.google-icon');
        }
    }
```

## コンポーネントのコンストラクタ

コンストラクターで指定した引数はコンポーネントの属性になります。

コンストラクタに渡された引数はView内で使用できます。ここでは`$name`を使ってアイコン名を受け取り、`span`タグ内で利用しています。

## `attributes`について

`$attributes`にはその他の指定された属性名と値が入っています。

通常ではコンポーネントに`class`を指定すると元々指定されていた`class`が上書きされてしまいますが、`merge`メソッドを使うことで、渡された`class`と元々の`class`を結合した状態でレンダリングするようにできます。

## コンポーネントの利用

コンポーネントを使用する際は接頭辞として`x-`をつけます。早速共通ヘッダーで使ってみましょう。

```diff php:/laravel-app/resources/views/common/header.blade.php
    <header class="fixed top-0 left-0 bg-inherit">
        <div class="w-screen h-[10vh] pl-5 pr-5 flex justify-between items-center">
            <div>
                <a href="{{ route('home') }}">
                    <h1 class="text-3xl font-bold">{{ env('APP_NAME') }}</h1>
                </a>
            </div>

            <div class="flex gap-2">
                <a
                    class="transition duration-300 ease-in-out hover:scale-[1.2] hover:text-blue-500"
                    href="{{ route('todo.create') }}"
                >
-                   <span class="material-icons-round text-2xl">add</span>
+                   <x-google-icon name="add" class="text-2xl"/>
                </a>
                <a
                    class="transition duration-300 ease-in-out hover:scale-[1.2] hover:text-blue-500"
                    href="{{ route('todo.index') }}"
                >
-                   <span class="material-icons-round text-2xl">list</span>
+                   <x-google-icon name="list" class="text-2xl"/>
                </a>
            </div>
        </div>
    </header>
```

## `$slot`：スロット

ヘッダーないのリンクも共通化できそうです。

リンクのコンポーネントも作りましょう。

```bash:/laravel-app
php artisan make:component Link      

   INFO  Component [app/View/Components/Link.php] created successfully.  

   INFO  View [resources/views/components/link.blade.php] created successfully.
```

```php:/laravel-app/resources/views/components/link.blade.php
<a
    {{
        $attributes->merge([
            'class' => 'transition duration-300 ease-in-out hover:scale-[1.2] hover:text-blue-500'
        ])
    }}
>
    {{ $slot }}
</a>
```

`$slot`という変数には子要素が渡されます。これでこのコンポーネントにネストされた子要素をどこで利用するかが指定できます。

ヘッダーを再度書き換えます。

```diff php:/laravel-app/resources/views/common/header.blade.php
    <header class="fixed top-0 left-0 bg-inherit">
        <div class="w-screen h-[10vh] pl-5 pr-5 flex justify-between items-center">
            <div>
                <a href="{{ route('home') }}">
                    <h1 class="text-3xl font-bold">{{ env('APP_NAME') }}</h1>
                </a>
            </div>

            <div class="flex gap-2">
-               <a
-                   class="transition duration-300 ease-in-out hover:scale-[1.2] hover:text-blue-500"
-                   href="{{ route('todo.create') }}"
-               >
+                   <x-link href="{{ route('todo.new') }}">
                        <x-google-icon name="add" class="text-2xl"/>
+                   </x-link>
-               </a>
-               <a
-                   class="transition duration-300 ease-in-out hover:scale-[1.2] hover:text-blue-500"
-                   href="{{ route('todo.index') }}"
-               >
+                   <x-link href="{{ route('todo.index') }}">
                        <x-google-icon name="list" class="text-2xl"/>
+                   </x-link>
-               </a>
            </div>
        </div>
    </header>
```

表示に変わりがないか確認してみてください。

# 次に進む前に

ここで各画面のデザインを先に作ってしまおうと思います。

新しい内容はないのでそのままコピペしてしまってください。

:::details 各画面のBladeの内容

### ホーム画面

```php:/laravel-app/resources/views/home.blade.php
@extends('layout')

@section('main')
<div class="w-full h-full flex justify-center items-center">
    <h1 class="text-5xl font-semibold -translate-y-header tracking-widest" style="text-shadow: 10px 10px 5px rgba(0, 0, 0, 0.2)">
        {{ env('APP_NAME') }}
    </h1>
</div>
@endsection
```

<br>

### ログイン画面

```php:/laravel-app/resources/views/login.blade.php
@extends('layout')

@section('main')
<div class="w-full h-full flex justify-center items-center">
    <div class="h-auto w-[70vh] py-10 px-14 border-solid border-[0.5px] border-gray-300 rounded-lg shadow-md shadow-gray-200 -translate-y-header">
        <form>
            <h2 class="text-xl font-semibold tracking-wider">
                Login
            </h2>
            <div class="w-full h-[1px] mt-2 bg-gray-300"></div>
            <div class="mt-9 flex gap-6 flex-col">
                <input type="text" placeholder="ユーザー名" autocomplete="off" class="p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500">
                <input type="password" placeholder="パスワード" autocomplete="off" class="p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500">
            </div>
            <div class="mt-10 flex justify-end">
                <button class="p-3 text-sm text-white bg-blue-500 rounded-sm shadow-md shadow-gray-300">
                    ログイン
                </button>
            </div>
        </form>
    </div>
</div>
@endsection
```

<br>

### アカウント登録画面

```php:/laravel-app/resources/views/signup.blade.php
@extends('layout')

@section('main')
<div class="w-full h-full flex justify-center items-center">
    <div class="h-auto w-[70vh] py-10 px-14 border-solid border-[0.5px] border-gray-300 rounded-lg shadow-md shadow-gray-200 -translate-y-[1rem]">
        <form>
            <h2 class="text-xl font-semibold tracking-wider">
                Sign Up
            </h2>
            <div class="w-full h-[1px] mt-2 bg-gray-300"></div>
            <div class="mt-9 flex gap-6 flex-col">
                <input type="text" placeholder="ユーザー名" autocomplete="off" class="p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500">
                <input type="email" placeholder="メールアドレス" autocomplete="off" class="p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500">
                <input type="password" placeholder="パスワード" autocomplete="off" class="p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500">
                <input type="password" placeholder="パスワード（確認用）" autocomplete="off" class="p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500">
            </div>
            <div class="mt-10 flex justify-end">
                <button class="p-3 text-sm text-white bg-blue-500 rounded-sm shadow-md shadow-gray-300">
                    登録
                </button>
            </div>
        </form>
    </div>
</div>
@endsection
```

<br>

### ToDo一覧画面

```php:/laravel-app/resources/views/todo/index.blade.php
@use(Illuminate\Support\Carbon)
@extends('layout')

@section('main')
<div class="w-full h-auto pt-14 pb-10 flex flex-col justify-start items-center gap-6">

    @foreach ($todos as $todo)

        <div class="h-auto w-[70vh] py-5 pl-13 pr-6 border-solid border-[0.5px] border-gray-300 rounded-lg shadow-md shadow-gray-200 relative overflow-hidden">
            <div class="w-[7%] h-full absolute top-0 left-0" style="background-color: {{ $todo->color }}"></div>
            <form>
                <div class="flex items-center gap-7">
                    <div class="mt-2">
                        <input type="checkbox" @checked($todo->done) class="w-[25px] h-[25px]">
                    </div>
                    <div class="flex flex-col gap-3">
                        <x-link href="" class="text-md font-semibold tracking-wider">
                            {{ $todo->title }}
                        </x-link>
                        @if ($todo->memo !== null)
                            <span class="text-xs">
                                {{ $todo->memo }}
                            </span>
                        @endif
                        <div class="flex gap-2">
                            <x-google-icon name="schedule" class="text-[20px]! text-gray-400"/>
                            <span class="text-xs text-gray-400">
                                {{ Carbon::create($todo->deadline)->isoFormat('YYYY/MM/DD（ddd）HH:mm') }}
                            </span>
                        </div>
                    </div>
                </div>
            </form>
        </div>

    @endforeach
    
</div>
@endsection
```

<br>

### ToDo登録画面

```php:/laravel-app/resources/views/todo/new.blade.php
@use(App\Enums\Color)

@extends('layout')

@section('main')
<div class="w-full h-auto pt-14 pb-10 flex justify-center items-start">
    <div class="h-auto w-[70vh] py-10 px-14 border-solid border-[0.5px] border-gray-300 rounded-lg shadow-md shadow-gray-200 -translate-y-[1rem]">
        <form>
            <h2 class="text-xl font-semibold tracking-wider">
                New ToDo
            </h2>
            <div class="mt-5 flex gap-6 flex-col">
                <input type="text" placeholder="ToDo" autocomplete="off" class="p-3 text-lg border-b border-gray-400 focus:outline-none focus:border-blue-500">
                <input type="date" placeholder="期限" autocomplete="off" class="p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500">
                <input type="time" placeholder="期限" autocomplete="off" class="p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500">
                <textarea placeholder="メモ" class="w-full min-h-20 resize-none field-sizing-content p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500"></textarea>
            </div>
            <div class="mt-10 flex justify-end">
                <button class="p-3 text-sm text-white bg-blue-500 rounded-sm shadow-md shadow-gray-300">
                    ToDoを追加
                </button>
            </div>
        </form>
    </div>
</div>
@endsection
```

<br>

### ToDo編集画面

```php:/laravel-app/resources/views/todo/edit.blade.php
@extends('layout')

@section('main')
<div class="w-full h-auto pt-14 pb-10 flex justify-center items-start">
    <div class="h-auto w-[70vh] py-10 px-14 border-solid border-[0.5px] border-gray-300 rounded-lg shadow-md shadow-gray-200 -translate-y-[1rem]">
        <form>
            <h2 class="text-xl font-semibold tracking-wider">
                Edit ToDo
            </h2>
            <div class="mt-5 flex gap-6 flex-col">
                <input type="text" placeholder="ToDo" autocomplete="off" class="p-3 text-lg border-b border-gray-400 focus:outline-none focus:border-blue-500">
                <input type="date" placeholder="期限" autocomplete="off" class="p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500">
                <input type="time" placeholder="期限" autocomplete="off" class="p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500">
                <textarea placeholder="メモ" class="w-full min-h-20 resize-none field-sizing-content p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500"></textarea>
            </div>
            <div class="mt-10 flex justify-end">
                <button class="p-3 text-sm text-white bg-blue-500 rounded-sm shadow-md shadow-gray-300">
                    ToDoを追加
                </button>
            </div>
        </form>
    </div>
</div>
@endsection
```
:::