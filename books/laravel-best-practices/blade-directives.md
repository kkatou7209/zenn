---
title: "Bladeのディレクティブ"
---

# 概要

本章ではBladeディレクティブを使い、作成したページに機能を追加していきます。

**ディレクティブ（Directive）** とは日本語で「命令」という意味です。

ディレクテブを使うことでLaravelにページを生成する際の指示を与えることができます。

## 本章で学ぶこと

- Bladeディレクティブの使い方
- 頻繁に使うディレクティブ

# Bladeディレクティブとは

LaravelがBladeファイルからHTMLに変換する際に参照するマーカーのようなものです。

`@`を使って関数のように使用します。例えばよく使う`@foreach`などはPHPの`foreach`キーワードにあたり、`@foreach`と`@endforeach`に囲まれたHTMLタグを繰り返しHTMLに埋め込みます。

```php:@foreachの例

<ul>
    @foreach(range(1, 3) as $num)
        <li>{{ $num }}</li>
    @endforeach
</ul>
```

```html:上記の結果
<ul>
    <li>1</li>
    <li>2</li>
    <li>3</li>
</ul>
```

レイアウトを作成する際に使用した`@extend`や`@section`、`@yield`などのディレクティブの一種です。

# よく使うディレクティブ

ここからは先に作っておいたページを編集しながらディレクティブについて学んでいきましょう。

## `@csrf`ディレクティブ

ページを作成したときに`<form>`タグを使ってリクエストを送れるようにしましたが、デフォルトの設定ではリクエストを送ってもLaravelは受け付けてくれません。

CSRFトークンがないリクエストはセキュリティ上の観点から無効と見做されます。

なので、リクエストの際にトークンも一緒に送れるように、`<form>`タグ内にトークンを追加しましょう。

トークンを追加するには`@csrf`ディレクティブを使います。

```diff php:/laravel-app/resources/views/login.blade.php
    <div class="h-auto w-[70vh] py-10 px-14 border-solid border-[0.5px] border-gray-300 rounded-lg shadow-md shadow-gray-200 -translate-y-header">
        <form action="{{ route('user.login') }}" method="POST">
+           @csrf
            <h2 class="text-xl font-semibold tracking-wider">
                Login
            </h2>
```

同じように、他のページにも`@csrf`を追加してください。


## `@checked`ディレクティブ

`@checked`ディレクティブは`<input>`要素の`checkbox`タイプなどに用いるディレクティブです。

ディレクティブを使わなかった場合は次のような書き方になります。

```php:ディレクティブを使わない場合の書き方
<input type="checkbox" {{ $todo->done ? 'checked' : '' }}>
```

ディレクティブを使う場合は次のようになります。

```php:@checkedディレクティブを使う場合
<input type="checkbox" @checked($todo->done)>
```

ディレクティブを使った方はかなり簡潔になったと思います。

では作成したページに適用してみましょう。今回チェックボックスを使っているページは**ToDo一覧**ページです。

このページのToDoに`@checked`を適用してみましょう。

```diff php:/laravel-app/resources/views/todo/index.blade.php
    ...

    @foreach ($todos as $todo)

        <div class="h-auto w-[70vh] py-5 pl-13 pr-6 border-solid border-[0.5px] border-gray-300 rounded-lg shadow-md shadow-gray-200 relative overflow-hidden">
            <div class="w-[7%] h-full absolute top-0 left-0" style="background-color: {{ $todo->color }}"></div>
            <div class="flex items-center gap-7">
                <div class="mt-2">
-                   <input type="checkbox" class="w-[25px] h-[25px]">
+                   <input type="checkbox" @checked($todo->done) class="w-[25px] h-[25px]">
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
        </div>

    @endforeach

    ...
```

Seederでデータを作成してあると思うので、`todos`テーブルの`done`カラムが`true`になっている場合はチェックされている状態で表示されるはずです。

サーバーを起動して確認してみてください。


## `@selected`ディレクティブ

`@selected`ディレクティブも`@checked`ディレクティブと同じく、記述を簡潔にしてくれるディレクティブです。こちらは`<select>`タグ内の`<option>`タグなどで使います。

例で使い方を見てみましょう。

まずはディレクティブを使わない方法から。

```php:ディレクティブを使わない場合
<select>
    <option value="1" {{ $number === 1 ? 'selected' : '' }}>1</option>
    <option value="2" {{ $number === 2 ? 'selected' : '' }}>2</option>
    <option value="3" {{ $number === 3 ? 'selected' : '' }}>3</option>
</select>
```

次がディレクティブを使った場合です。

```php:ディレクティブ使う場合
<select>
    <option value="1" @selected($number === 1)>1</option>
    <option value="2" @selected($number === 2)>2</option>
    <option value="3" @selected($number === 3)>3</option>
</select>
```

こちらのディレクティブもページに適用してみましょう。

今回のアプリケーションでは**ToDo登録**ページと**ToDo編集**ページで`<select>`を使用しています。

ですが、今は編集ページだけに適応します。`@select`ディレクティブだけでは最初からデータが存在している場合でしか使う意味がないからです。

```diff php:
    <h2 class="text-xl font-semibold tracking-wider">
        Edit ToDo
    </h2>
    <div class="mt-5 flex gap-6 flex-col">
        <input type="text" placeholder="ToDo" autocomplete="off" class="p-3 text-lg border-b border-gray-400 focus:outline-none focus:border-blue-500">
        <input type="date" placeholder="期限" autocomplete="off" class="p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500">
        <input type="time" placeholder="期限" autocomplete="off" class="p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500">
        <select name="color" autocomplete="off" class="p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500">
            <option value="">色を選択してください</option>
            @foreach (Color::cases() as $color)
-               <option value="{{ $color }}">{{ $color->display() }}</option>
+               <option value="{{ $color }}" @selected($color->value === $todo->color)>{{ $color->display() }}</option>
            @endforeach
        </select>
        <textarea placeholder="メモ" class="w-full min-h-20 resize-none field-sizing-content p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500"></textarea>
    </div>
```

## `@old`ディレクティブ

`@old`ディレクティブは前回のリクエスト時の値を取得します。

例えば、ユーザーの入力が不正で同じページにリダイレクトしたときに、`@old`ディレクティブを使えば前回の入力値を取り出すことができます。

もっとも、リクエストとして送った値でなければ取り出せません。

こちらも使用例を見てみましょう。

```php:ディレクティブを使わない場合
<input name="title" value="{{ request()->old('title') }}">
```

```php:ディレクティブを使った場合
<input name="title" value="{{ @old('title') }}">
```

このディレクティブはToDo登録画面で使いましょう。

```diff php:/laravel-app/resources/views/todo/new.blade.php
    ....
    <h2 class="text-xl font-semibold tracking-wider">
        New ToDo
    </h2>
    <div class="mt-5 flex gap-6 flex-col">
-       <input type="text" name="title" placeholder="ToDo" autocomplete="off" class="p-3 text-lg border-b border-gray-400 focus:outline-none focus:border-blue-500">
+       <input type="text" name="title" value="{{ @old('title') }}" placeholder="ToDo" autocomplete="off" class="p-3 text-lg border-b border-gray-400 focus:outline-none focus:border-blue-500">
-       <input type="date" name="date" placeholder="期限" autocomplete="off" class="p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500">
+       <input type="date" name="date" value="{{ @old('date') }}" placeholder="期限" autocomplete="off" class="p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500">
-       <input type="time" name="time" placeholder="期限" autocomplete="off" class="p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500">
+       <input type="time" name="time" value="{{ @old('time') }}" placeholder="期限" autocomplete="off" class="p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500">
        <select name="color" autocomplete="off" class="p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500">
                <option value="">色を選択してください</option>
                @foreach (Color::cases() as $color)
                    <option value="{{ $color }}" style="background-color: {{ $color }};">{{ $color->display() }}</option>
                @endforeach
        </select>
-       <textarea name="memo" placeholder="メモ" class="w-full min-h-20 resize-none field-sizing-content p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500"></textarea>
+       <textarea name="memo" placeholder="メモ" class="w-full min-h-20 resize-none field-sizing-content p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500">{{ @old('memo') }}</textarea>
    </div>
    ...
```

### `@selected`との合体技

`@old`を使うことで前回のリクエストの値を取ることができるようになったので、色のコンボボックスで`@selected`を使って前回の値から選択されたオプションを判断することができるようになりました。

```diff php
    <select name="color" autocomplete="off" class="p-3 text-xs rounded-md border border-gray-400 focus:outline-blue-500">
        <option value="">色を選択してください</option>
        @foreach (Color::cases() as $color)
-           <option value="{{ $color }}" style="background-color: {{ $color }};">{{ $color->display() }}</option>
+           <option value="{{ $color }}" style="background-color: {{ $color }};" @selected(@old('color') === $color)>{{ $color->display() }}</option>
        @endforeach
    </select>
```

## `@error`ディレクティブ

`@error`ディレクティブはバリデーションエラーの有無によって表示を切り替えることができるディレクティブです。

`@if`と`@endif`によって条件付きの表示が実装できますが、`@error`は`@if`のエラー特化型といえます。

`@error`を使わない場合は、どの入力に対するどのようなエラーなのかを`@if`などを使って処理しなければなりませんが、`@error`を使えば`@error('title')`といったふうに記述を簡略化できます。

```diff php:/laravel-app/resources/views/todo/new.blade.php
    ...
    <input type="text" name="title" value="{{ @old('title') }}" placeholder="ToDo" autocomplete="off" class="p-3 text-lg border-b border-gray-400 focus:outline-none focus:border-blue-500">
+   @error('title')
+       <p class="text-red-400">{{ $message }}</p>
+   @enderror
    ...
```

<br>

入力のバリデーションエラーがあった際はページに`$message`という変数が自動で追加されます。`$message`は`@error`とよく一緒に使われるので覚えておくと良いでしょう。

ただ、今の状態では`validation.required`という文字列が表示されるだけで、ユーザーにとってはあまり役に立たない情報しかありません。

バリデーションを実装する際に`FormRequest`クラスを作ったことを覚えているでしょうか。実はこのクラスに`messages`メソッドを追加することでカスタムエラーメッセージを追加できます。

```diff php:/laravel-app/app/Http/Requests/Todo/CreateRequest.php
    ....
    public function rules(): array
    {
        return [
            'title' => ['required', 'string'],
            'memo' => ['nullable', 'string'],
            'date' => ['nullable', 'date_format:Y-m-d'],
            'time' => ['nullable', 'date_format:H:i'],
            'color' => ['nullable', new Enum(Color::class)],
        ];
    }

+   public function messages()
+   {
+       return [
+           'title.required' => 'ToDoのタイトルが入力されていません',
+       ];
+   }
    ...
```

これで日本語のエラーメッセージが表示されるようになりました。

同じように`UpdateRequest`にも追加してください。


## その他のディレクティブ

今回は使いませんでしたが、よく使うディレクティブをざっと紹介します。

知っておくといざという時に役に立つかもしれません。

### `@php`ディレクティブ

`@php`ディレクティブを使うとその中でPHPを実行できます。

基本は`@endphp`とでか今度使うやり方です。

```php
@php
    $title = 'Laravel App';
@endphp

<h1>
    {{ $title }}
<h1>
```

一行で十分なら`@php`単体でも使えます。

```php
@php($title = 'Laeavel App')

<h1>
    {{ $title }}
<h1>
```

### `@env`ディレクティブ

`.env`ファイルの`APP_ENV`の値によって実行するかどうかを決めるディレクティブです。

`APP_ENV`が`local`になっている場合、以下のコードは実行されます。

```php
@env('local')
    <h1>Hello!!</h1>
@endenv
```

開発と本番、テスト環境などで動きを変える際には便利なディレクティブです。


# まとめ

Bladeのディレクティブには記述を簡略化するものや、表示の柔軟性を高めるものがあり、うまく使いこなすことによって見通しの良いコードが書けるようになります。

書き方が少し冗長だなと思った時は是非とも便利なディレクティブがないか探してみてください。

ここで紹介した他にも様々なディレクティブがありますので、気になる方は[こちら](https://laravel.com/docs/11.x/blade#:~:text=%3E%3C/span%3E-,Additional%20Attributes,-For%20convenience%2C%20you)のページをご確認ください。