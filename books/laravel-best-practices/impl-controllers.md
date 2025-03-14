---
title: 'コントローラーの実装'
---

# 概要

ここにきてやっとコントローラーの内部の実装ができるようになりました。

本章ではこれまで実装してきた機能を使ってコントローラーの内部処理を実装していきます。


## 本書で学ぶこと

- コントローラー内の処理の実装方法


# リクエストから値を取得する

リクエストクラスを作ったときに、コントローラーの`__invoke`メソッドの引数にリクエストクラスを渡していたと思います。

しかし、クラスでなくてもリクエストから値を取り出すことが可能です。


## パスパラメータの値を受け取る

**ToDo編集**ページではパスパラメータから要求されているToDoのデータを判断します。

関心があるのはパスパラメータだけなので、リクエスト全体を受け取るのは冗長です。

実はコントローラーのメソッドの引数にパラメータと同じ引数名を指定することでパスパラメータの値を受け取ることができます。

その前にまず`web.php`を編集しましょう。

```diff php:/laravel-app/routes/web.php


    Route::prefix('/todo')
        ->as('todo.')
        ->group(function () {

            Route::get('/', Todo\IndexController::class)->name('index');

            Route::get('/new', Todo\NewController::class)->name('new');

-           Route::get('/edit', Todo\EditController::class)->name('edit');
+           Route::get('/edit/{id}', Todo\EditController::class)->name('edit');

            Route::post('/create', Todo\CreateController::class)->name('create');

            Route::put('/update', Todo\UpdateController::class)->name('update');

            Route::delete('/delete', Todo\DeleteController::class)->name('delete');
        });
```

これでメソッドの引数でパラメータを受け取ることができます。

次にコントローラーを編集します。

```diff php:/laravel-app/app/Http/Controllers/Todo/EditController.php

-   public function __invoke()
+   public function __invoke(int $id)
    {
+       $todo = $this->repository->find($id);
+   
+       if ($todo === null) {
+   
+           return redirect()->route('todo.index');
+       }
+
+       $date = Carbon::create($todo->deadline);
+
-       return view('todo.edit');
+       return view('todo.edit', ['todo' => (object) [
+           'id' => $todo->id,
+           'title' => $todo->title,
+           'color' => $todo->color,
+           'done' => $todo->done,
+           'date' => $date->isoFormat('YYYY-MM-DD'),
+           'time' => $date->isoFormat('HH:mm'),
+           'memo' => $todo->memo,
+       ]]);
    }
```

引数に`int $id`とすることで`id`パラメータを取得しています。

そして取得した`id`をもとにリポジトリクラスを使ってToDoを取得しています。

`id`に対応したToDoが見つからない場合もあり得るので、今回は一覧ページにリダイレクトすることにします。

最後に`view`関数の第２引数に取得したToDoデータを渡しています。

これで例えば`/todo/edit/1`というURLにアクセスすれば、`id`カラムの値が`1`のToDoに対応する編集ページが返されるようになりました。

:::message
取得したデータを表示する処理はのちの章で作成します。

今回は実装しませんでしたが、パスパラメータだけでなく、クエリパラメータ（`/tood/edit?id=1`など）もメソッドの引数で受け取ることができます。
:::

## リクエストの値を受け取る

URLから値を受け取ることはできましたが、POSTメソッドなどで送られたデータはどのように受け取るのでしょうか。

ここで`Request`の出番です。

`Request`の`input`メソッドを使うことでリクエストのデータを受け取ることができます。

```php:inputメソッドの例
public function __invoke(Request $request)
{
    $title = $request->input('title');
}
```

`input`メソッドは`FormRequest`にも備わっているので、あらかじめ作っておいた`FormRequest`クラスでも使えます。

しかしバリデーションを実装している場合は、検証済みの値の配列をはじめから受け取ることができます。

検証済みの値を受け取るには`validated`メソッドを使います。

コントローラーを編集してこれを実装しましょう。

```php:/laravel-app/app/Http/Controllers/Todo/CreateController.php
    ...
    public function __invoke(CreateRequest $request)
    {
        $data = $request->validated();
    }
```

```php:/laravel-app/app/Http/Controllers/Todo/UpdateController.php
    ...
    public function __invoke(UpdateRequest $request)
    {
        $data = $request->validated();
    }
```

```php:/laravel-app/app/Http/Controllers/Todo/DeleteController.php
    ...
    public function __invoke(DeleteRequest $request)
    {
        $data = $request->validated();
    }
```

返り値は連想配列になっているので、キーを指定して値を取り出します。

```php
$title = $data['title'];
```

# データ操作

今のままではただデータを受け取るだけで何もしていません。

送られてきたデータを扱う処理を追加しましょう。

## データの取得

まずはToDoの一覧を取得する処理を実装しましょう。`IndexController`を次のようにします。

```diff php:
    public function __invoke()
    {
+       $todos = $this->repository
+           ->list(1)
+           ->sortBy('deadline');

-       return view('todo.index');
+       return view('todo.index', ['todos' => $todos]);
    }
```


## データの登録

次はToDoを登録する処理です。`CreateController`を編集しましょう。

処理内容は以下のようにしました。

```diff php:/laravel-app/app/Http/Controllers/Todo/CreateController.php
    public function __invoke(CreateRequest $request)
    {
        $data = $request->validated();

+       $date = $data['date'];
+       $time = $data['time'];
+   
+       $datetime = null;
+   
+       if ($date !== null && $time !== null) {
+   
+           $datetime = "{$date} {$time}";
+       }
+   
+       $this->repository->add([
+           'title' => $data['title'],
+           'memo' => $data['memo'],
+           'deadline' => $datetime,
+           'color' => $data['color'],
+           'user_id' => 1,
+       ]);
+   
+       return redirect()->route('todo.index');
    }
```

`deadline`の値は日時形式ですが、入力の時点では日付と時間に分かれているので、日付と時間のどちらもが入力されている場合にだけ登録するようにしました。

次に`Repository`クラスを使ってToDoの登録を行い、一覧ページにリダイレクトをしています。

`user_id`に`1`を指定していますが、これは認証を実装するまでの仮の措置です。認証実装後はログイン中のユーザーに基づいて値を入れます。

## データの更新

今度はToDoの更新を行います。

`UpdateController`を以下のように編集してください。

```diff php:/laravel-app/app/Http/Controllers/Todo/UpdateController.php
    ...
    public function __invoke(UpdateRequest $request)
    {
        $data = $request->validated();

+       $date = $data['date'];
+       $time = $data['time'];
+   
+       $datetime = null;
+   
+       if ($date !== null && $time !== null) {
+   
+           $datetime = "{$date} {$time}";
+       }
+   
+       $this->repository->update($data['id'], [
+           'title' => $data['title'],
+           'memo' => $data['memo'],
+           'deadline' => $datetime,
+           'color' => $data['color'],
+       ]);
+   
+       return redirect()->route('todo.index');
    }
```

大まかな処理は、`id`を指定している以外は登録時と同じです。

## データの削除

最後に削除する処理を追加します。

`DeleteController`を編集します。

```diff php:/laravel-app/app/Http/Controllers/Todo/DeleteController.php
    ...
    public function __invoke(DeleteRequest $request)
    {
        $data = $request->validated();

+       $id = $data['id'];

+       $this->repository->delete($id);

+       return redirect()->route('todo.index');
    }
```

`id`の入力値だけを取り出して削除するToDoを指定しています。

:::message
ユーザーの登録やログイン、ToDoの完了・未完了の切り替えは？
:::


## まとめ

ここではコントローラーの内部処理を実装しました。

コントローラーで行った処理は大きく３つです。

- リクエストを受け取る
- データを処理する
- 適切なページを返す

これに加えて、後々認証関係の処理を追加していきます。

ですが忘れてはならないのが、コントローラーの役割はあくまで仲介であるということです。

個々の処理の詳細は個別のクラスなどに任せることが基本です。

今回は簡単な処理内容だったのでコントローラーのなかで`Repository`を扱いましたが、もっと複雑な処理が挟まる場合は`Service`クラスなどを作って任せてしまうのが良いでしょう。