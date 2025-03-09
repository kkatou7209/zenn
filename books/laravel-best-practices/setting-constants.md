---
title: "定数の設定"
---

# 概要

本章で学ぶこと

- 定数の管理方法
- 定数の利用


## BackedEnum

皆さんは普段、定数管理をどのように行っていますか？

よくあるやり方として、`Consts`クラスなるものを作って読み取り専用のプロパティを作ったり、JavaScriptであれば`Object.freeze`なんかをつかってオブジェクトのプロパティを定数にしたりするのではないでしょうか。

もちろんPHPでもそういうやり方ができます。しかしLaravelを使う際は列挙型（`Enum`）を使うことを強くお勧めします。

<br>

PHPには値つき列挙型（`BackedEnum`）があります。

これがとても便利で、Laravelのクエエリビルダーではそのままデータとして使えますし、バリデーションとしても利用できます。

今回はこの`BackedEnum`を中心的に使って定数管理を行います。


# 列挙型の作成

Laravelには列挙型を作るコマンドがあります（大したことはしませんが）。

名前空間の宣言忘れも防げますし、せっかくなので使っていきましょう。

<br>

今回の開発で定数として必要なのは**色**に関する定数です。

ToDoを色分けする際の色を表す定数を作っていきましょう。

```bash:/laravel-app
$ php artisan make:enum Enums/Color

   INFO  Enum [app/Enums/Color.php] created successfully.  
```

`app/Enums`の下に`Color.php`ができましたでしょうか。

このファイルを以下のように編集してください。

```diff php:/laravel-app/app/Enums/Color.php
    ...

-   enum Color
+   enum Color: string
    {
-       //
+       case Red = '#FA777B';
+   
+       case Orange = '#E69D5A';
+   
+       case Yellow = '#E6DE56';
+   
+       case Blue = '#61A1E5';
+   
+       case Green = '#6CF082';
+   
+       case Purple = '#C1A3E5';
    }

```

基本的な準備はこれで以上です、早速この列挙型を使ってみましょう。


# バリデーションでの列挙型の利用

列挙型を作ったときに値を設定しましたが、この値をそのままバリデーションにも使えます。

前章で`FormRequest`を作成し、バリデーションを実装しましたが、ここで列挙型を使ってみましょう。

`CreateRequest`と`UpdateRequest`を次のように変えてください。

```diff php:laravel-app/app/Http/Requests/Todo/CreateRequest.php
+   use Illuminate\Validation\Rules\Enum;
+   use App\Enums\Color;

    ...

        public function rules(): array
        {
            return [
                'title' => ['required', 'string'],
                'memo' => ['nullable', 'string'],
-               'color' => ['nullable', 'string'],
+               'color' => ['nullable', new Enum(Color::class)],
                'done' => ['required', 'boolean'],
            ];
        }
```

```diff php:laravel-app/app/Http/Requests/Todo/UpdateRequest.php
+   use Illuminate\Validation\Rules\Enum;
+   use App\Enums\Color;

    ...

        public function rules(): array
        {
            return [
                'id' => ['required', 'integer'],
                'title' => ['required', 'string'],
                'memo' => ['nullable', 'string'],
-               'color' => ['nullable', 'string'],
+               'color' => ['nullable', new Enum(Color::class)],
                'done' => ['required', 'boolean'],
            ];
        }
```

これで`Color`に定義されている色コード以外は入力できないようになりました。


# Bladeでの列挙型の利用

定数を作ったならフォームのコンボックスなどでも使いたくなります。

動的に変更できた方が後々の変更の際にも効率がいいですしね。

しかしPHPの列挙型には表示値を定義する機能などはありません。なので自分で実装する必要があります。

<br>

今回は`display`というメソッドで表示値を返すようにしましょう。

後々同じような列挙型を作るかもしれませんので、インターフェースを定義してそれを実装するようにします。

```bash:/laravel-app
$ php artisan make:interface Interfaces/Display    

   INFO  Interface [app/Interfaces/Display.php] created successfully.
```

```diff php:/laravel-app/app/Interfaces/Display.php
    namespace App\Interfaces;

    interface Display
    {
-       //
+       public function display(): string;
    }
```

```diff php:/laravel-app/app/Enums/Color.php
+   use App\Interfaces\Display;

-   enum Color: string
+   enum Color: string implements Display
    {
        case Red = '#FA777B';

        case Orange = '#E69D5A';

        case Yellow = '#E6DE56';

        case Blue = '#61A1E5';

        case Green = '#6CF082';

        case Purple = '#C1A3E5';

+       public function display(): string
+       {
+           return match($this) {
+               self::Red => '赤',
+               self::Orange => 'オレンジ',
+               self::Yellow => '黄',
+               self::Blue => '青',
+               self::Green => '緑',
+               self::Purple => '紫',
+           };
+       }
    }
```

表示値が取得できるか早速試してみましょう。

ToDo登録画面のBladeファイルを変更します。

```diff php:/laravel-app/resources/views/todo/new.blade.php
+   @use(App\Enums\Color)

    <div>
        <h1>ToDo登録</h1>
+       <ul>
+           @foreach (Color::cases() as $color)
+               <li>{{ $color->display() }}</li>
+           @endforeach
+       </ul>
    </div>
```

ちゃんと表示値が返されていますね。

![alt text](/images/setting-constants_1.png =250x)
*`http://127.0.0.1:8000/todo/new`*

:::message
`@`から始まる新しい構文が出てきましたが、これは**ディレクティブ**といいます。

ディレクティブは後の章で扱います。
:::