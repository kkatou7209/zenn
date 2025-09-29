---
title: "Vite + TypeScript プロジェクトの構築方法"
emoji: "⚡️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vite", "typescript", "node"]
published: true
---

この記事では Vite プロジェクトの構築方法について解説します。

Node.js と npm はすでにインストール済みと前提して話を進めていきますので、もし記事を読みながら自分で Vite 構築を進めていきたいのであればあらかじめ Node.js の環境を用意することをお勧めします。

本稿は JavaScript や TypeScript での開発経験はあるけど、Vite プロジェクトを自分で構築したことがないという方を対象にしています。

また、Vite とは何か、何ができるのか、といった概要的な解説は含みません。あらかじめご了承ください。


## 最も簡単な方法

ダラダラと前置きもなんなのでさっそく Vite の初め方について話していきます。

Vite プロジェクトの最も簡単な初期化の方法は `create-vite` を用いたやり方です。

単純にターミナルに `npm create vite@latest` と打ち込むと Vite の雛形プロジェクトを作成できます。

`latest` の部分には任意のバージョンを指定できます。

```sh
npm create vite@latest
```

:::message
以下のコマンドはいずれも同じ意味です。

```sh
npm create vite@latest

npm init vite@latest

npm exec create-vite@latest
```
:::

コマンドを実行するとインタラクティブ形式でプロジェクトの設定を行なってくれます。

```sh> npx
> create-vite

│
◆  Project name:         <-- プロジェクト名を選択します
│  vite-project
│
◆  Select a framework: # <-- 使用するライブラリを選択します
│  ● Vanilla
│  ○ Vue
│  ○ React
│  ○ Preact
│  ○ Lit
│  ○ Svelte
│  ○ Solid
│  ○ Qwik
│  ○ Angular
│  ...
│
◆  Select a variant:    <-- ライブラリの形式を選択します
│  ● TypeScript
│  ○ JavaScript
│  ○ Official Vue Starter ↗
│  ○ Nuxt ↗
│
◆  Use rolldown-vite (Experimental)?: <-- 後述
│  ○ Yes
│  ● No
│
◆  Install with npm and start now?    <-- このままパッケージをインストールしてサーバーを立ち上げるかどうか聞かれます
│  ● Yes / ○ No
│
◇  Scaffolding project in {PWD}/vite-project...
│
└  Done. Now run:

  cd vite-project
  npm install
  npm run dev
```

プロジェクトの要件に沿う設定を選んでください。

なお、`Select a variant` で `TypeScript` を選択した場合は TypeScript のトランスパイラも `package.json` に含まれて出力され、`tsconfig.json` も作ってくれます。

:::message
#### rolldown-vite

**rolldown** は Rust で開発され、今後 Vite との統合が計画されているJSバンドラーです。

現状 Vite は内部的に **esbuild** と **rollup** を使用していますが、それらの後継となるのがこの **rolldown** です。

現在（2025年9月時点）でベータ版が利用できるので、興味のある方は試してみてください。
:::

このように、基本的には初期化スクリプトを利用する形になるので、一から Vite をインストールして構築するようなことはほとんどないでしょう。

今回は npm を使いましたが、他の Node パッケージマネージャーでも同様に初期化できます。

```sh
yarn create vite@latest

pnpm create vite@latest

bun create vite@latest
```

## Vite でよく使う設定

ここからは Vite でよく使うことになる設定について紹介していきます。

Vite プロジェクトを `create-vite` を使って初期化するとルートディレクトリに `vite.config.ts`（使用言語に JavaScript を選択した場合は `vite.config.js`）というファイルが作られます。

Vite 関連の設定は基本的にこのファイルで行います。

:::message
`vite.config.ts` の詳細については[公式ページ](https://vitest.dev/config/)を確認してください
:::

### エイリアス

Vite で開発するときはモジュールのインポートに相対パスを使わず、エイリアスを使った絶対パスを用いることがよくあります。

エイリアスとはパスの別名のことです。

絶対パスを使うとルートから辿った長いパスを使わなければなりませんし、人によってプロジェクトのディレクトリを置く場所は違いますので共同で作業ができなくなります。

プルしてくるたびにパスを書き換えなきゃいけないなんて御免被りますよね。

なのでルートパスから現在のプロジェクトパスまでを別名に置き換えます。

よく使われるのは `@` です。今いる（at）場所という意味なんですかね。知りませんけど。

エイリアスの設定には `vite.config.ts` の `resolve` というオプションを使います。

```ts:vite.config.ts
export default defineConfig({
    plugins: [vue()],
    resolve: { // <= 
        ...
    }
})
```

このオプションの `alias` というプロパティにエイリアスを設定します。

```ts:vite.config.ts
export default defineConfig({
    plugins: [vue()],
    resolve: {
        alias: {
            '@': path.resolve(__dirname, 'src'),
        }
    }
})
```

`src` はプロジェクト直下の `src` ディレクトリを指しています。解決先のパスを変えたい場合はこの部分を変更してください。

相対パスを絶対パスに変換するために Node.js の機能（`path`）を使っています。詳しくは触れませんが、TypeScriptで利用するためには `@types/node` をインストールする必要があります。

また、TypeScript を使っている場合は　`tsconfig.json` にもエイリアスを反映させる必要があります。

`compilarOptions` に次のような設定を追加してください。

```json:tsconfig.json
...
    "compilerOptions": {
        "baseUrl": ".",
        "paths": {
            "@/*" : ["./src/*"]
        }
    },
...
```

`vite.condig.ts` とパスの書き方がちょっと違いますが、そういうものだと思ってください。

Vite の方のパスを変える時は `tsconfig.json` の方にも反映させる必要がありますので注意してください。

ここまでの設定で以下のようにモジュールをインポートできるようになります。

```ts
import { calc } from '@/utils';
```

これで　`src` 直下の `utils.ts` ファイルをインポートしたことになります。

### 開発サーバー

`vite.config.ts` では開発サーバーの設定もできます。`vite` コマンドを実行した時に立ち上がるサーバーです。

開発サーバーはデフォルトで `http://localhost:5173/` をリッスンします。

ネットワークに詳しくない人のために一応説明しておくと、`localhost` とは自分自身を指すホスト名のことで、このホストに要求をかけると自身（今使ってるマシン）にリクエストを送ります。このホストのことを**ループバックアドレス**といいます。

`:` 以降はポート番号で、マシン上のアプリケーションの場所を指します。複数の入江（ポート）がある港（）を想像するとわかりやすいかもしれません（港も port なんですが）。

船を出航させてすぐに元いた港に戻ってくるようなことをしているわけです。

余談はこれくらいにして、まずはこのデフォルトの設定をいじってみましょうか。

#### アドレスの設定

まずは `localhost` をリッスンする設定を変えてみましょう。

Vite の開発サーバーの設定には　`vite.config.ts` の `server` オプションを使用します。

ホストの変更の際には `server` オプションの `host` プロパティの値を変更します。

```ts:vite.config.ts
export default defineConfig({
...
    server: {
        host: '0.0.0.0',
    }
})
```

アドレスを `localhost` から `0.0.0.0` に変更しました。

サーバーを起動するとリッスン先のアドレスが変わっていることが確認できます。

```sh
$ npm run dev

  VITE v7.1.7  ready in 252 ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: http://192.168.215.2:5173/
  ➜  press h + enter to show help
```

「いや別のアドレスになってるやないかい」と言われるかもしれませんが、ちゃんと `0.0.0.0` をリッスンしています。

`0.0.0.0` は「すべてのネットワークインターフェース」などを指します。詳しい説明は省きますが、ここに表示されているのは自分のIPアドレスで、Vite が勝手に解決してくれています。

筆者は環境に Docker を使っているのでここではコンテナのIPアドレスが表示されています。Docker での開発の際はよく使う設定なので覚えておくとお得ですよ。

なお、`host: true` としても同じ意味になります。

### ポートを変更する

次はリッスンするポートを変更してみましょう。

同じく `server` オプションをいじります。ポートの変更には　`port` プロパティを使います。

```ts:vite.config.ts
export default defineConfig({
    server: {
        host: '0.0.0.0',
        port: 9000, // <=
    }
})
```

これでデフォルトの `5137` から `9000` に変更されます。

さっそく確認してみましょう。

```sh
$ npm run dev

  VITE v7.1.7  ready in 274 ms

  ➜  Local:   http://localhost:9000/
  ➜  Network: http://192.168.215.2:9000/
  ➜  press h + enter to show help
```

はい、ちゃんと `9000` に変わってますね。

使い所としてはあんましないんですが、同一プロジェクトで複数の Vite プロジェクトを動かす場合は衝突を避けるためにそれぞれ別のポートを設定するといった使い方があります。

## おわり

基本的なことをさらっと解説しただけですが、Vite を始める際のなんらかの助けになれば嬉しいです。

[日本語の Vite ドキュメント](https://ja.vite.dev/)もあるのでさらに学習を進めるのであれば参考にしてみてください。

---

### 参考

- [Rolldown](https://rolldown.rs/)
- [Vite](https://vitest.dev)
- [Vite : 日本語ドキュメント](https://ja.vite.dev)
