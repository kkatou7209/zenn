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


# ユーザーの作成

データの登録やコントローラーの設定などは以前の章で扱いましたので改めて説明しません。

コントローラーはすでに作成済みですので、まずは`FormRequest`を作成しましょう。

ユーザー登録用とログイン用の２つを作ります。

```bash:/laravel-app
$ php artisan make:request User/CreateRequest

   INFO  Request [app/Http/Requests/User/CreateRequest.php] created successfully.  

$ php artisan make:request User/LoginRequest

   INFO  Request [app/Http/Requests/User/LoginRequest.php] created successfully. 
```

中身は次のようにします。

```php
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

コントローラーも編集してユーザーをデータベースに登録するようにしましょう。

```php:/laravel-app/app/Http/Controllers/User/CreateController.php
+   use App\Http\Requests\User\CreateRequest;
    ...
    public function __invoke()
    {

    }
```