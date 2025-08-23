---
title: "オープンソースのドキュメントツールを比較する"
emoji: "📄"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ドキュメントツール", "オープンソース"]
published: true
---

# 本稿の目的

OSSのドキュメントツールで社内運用可能なものに絞って比較します。

対象とするツールは以下の条件で絞ります。

- 社内（オンプレ）で運用可能
- オープンソースで無料
- PDF 出力ができる
- UI で編集が可能
- Markdown とリッチテキスト形式で編集ができる
- 現在進行形で開発が続けられている

以上の条件で調べると下記のツールが見つかりました。

- [**Outline**](https://www.getoutline.com/)
  - [Webサイト]((https://www.getoutline.com/))
  - [GitHub](https://github.com/outline/outline)
  - 最新バージョン：0.86.1
  - ライセンス：[BSL](https://en.wikipedia.org/wiki/Business_Source_License)
  
- [**Wiki.js**](https://js.wiki/)
  - [Webサイト](https://js.wiki/)
  - [GitHub](https://github.com/requarks/wiki)
  - 最新バージョン：2.5.308
  - ライセンス：[AGPL](https://ja.wikipedia.org/wiki/GNU_Affero_General_Public_License)

- [**Book Stack**](https://www.bookstackapp.com/)
  - [Webサイト](https://www.bookstackapp.com/)
  - [GitHub](https://github.com/BookStackApp/BookStack)
  - 最新バージョン：25.07.1
  - ライセンス：[MIT](https://ja.wikipedia.org/wiki/MIT%E3%83%A9%E3%82%A4%E3%82%BB%E3%83%B3%E3%82%B9)

:::message
Wiki.js に関しては PDF 出力がバージョン3で対応予定です。
:::

# 依存パッケージ・ツール

:::message
いずれも Docker で導入する際は必要ありません。
:::

## Outline

|          |     |
|:---------|:----|
|Node.js   |`20~`|
|PostgreSQL|`12~`|
|Redis     |`4~` |

## Wiki.js

|          |                   |
|:---------|:------------------|
|Node.js   |`22~`, `20~`, `18~`|
|PostgreSQL|`9.5~`             |
|MySQL     |`8~`               |
|MariaDB   |`10.2.7~`          |
|MS SQL    |`2012~`            |
|SQLite    |`3.9~`             |

## Book Stack

|        |       |
|:-------|:------|
|PHP     |`8.2~` |
|Composer|`2.2~` |
|MySQL   |`5.7~` |
|MariaDB |`10.2~`|

# 導入要件

## Outline

- CPU
  - 記述なし

- メモリ
  - 最低：512MB
  - 推奨：1GB

- ストレージ
  - 記述なし

:::message
「低スペックマシンでも動く」という記述を見る限り 2Core でも十分かもしれません。
:::

## Wiki.js

- CPU
  - 最低：2 Core

- メモリ
  - 最低：1GB

- ストレージ
  - 最低：1GB

## Book Stack

- CPU
  - 記述なし

- メモリ
  - 記述なし

- ストレージ
  - 記述なし


# デモページ

## Outline

明言されていませんが、多分ドキュメントが Outline で作られています。

https://docs.getoutline.com/s/hosting

## Wiki.js

公式ドキュメント自体が Wiki.js で実装されています。

https://docs.requarks.io/

## Book Stack

https://demo.bookstackapp.com/

# その他の特徴

## 日本語対応

### Outline

一部日本語に対応しています。

### Wiki.js

画面で日本語に変更できます。

### Book Stack

設定で日本語に変更できます。

## カスタマイズ

### Outline

カスタマイズに言及する記述は見受けられません。

APIなどはありますが、拡張プラグインの実装方法などは解説されておらず、拡張するとなるとゴリゴリのやり方になりそうです。

### Wiki.js

[モジュール](https://docs.requarks.io/dev)という形で拡張ができます。

`server/modules` に独自のプラグインを配置できるそうです。

テーマのカスタマイズに関してはいくつか制約があるので要確認です。

### Book Stack

テーマや見た目に関しては CSS をいじることで変えることができます。

その他ロゴなどに関しては設定値をいじる形になります。

## PDFの出力

### Outline

デフォルトでサポートされていますが、**Business + Enterprise** エディション（有料）でしか利用できません。

### Wiki.js

v3 で実装予定、現行の v2 では利用できません。

拡張で独自実装すれば使えるようになりそうです。

### Book Stack

デフォルトで利用できます。

# まとめ

## ユースケースごとの検討

### 日本語しかできないメンバーが多い

日本語対応している Book Stack か Wiki.js がいいでしょう。

### 必要に応じてPDFにしたい

Node.js と Docker での開発に慣れているエンジニアがある程度いるのであればカスタム可能範囲が広い Wiki.js、そうでない場合は Book Stack がいいかもしれません。

### 安定したツールを使いたい

開発期間の長い Book Stack がおすすめです。

## 個人的な感想

**XWiki** など他のツールも見受けられましたが、見た目と使い勝手があまり好みでなかったため除外しました。

ここで挙げたツールはいずれもUIがモダンで、かつ普段こういったツールを利用しない人でもすぐに使い方を覚えられそうな感じがしたので候補に挙げています。

全体的に Book Stack か Wiki.js がいいかなと思いました。

Wiki.js は拡張性の高く、依存も Node.js と DB くらいで、Docker を使わなくても管理が難しくないと思いました。

Book Stack は標準の機能が豊富なので、要件が増えていかない限りは使いが手が良さそうです。

見た目的に Outline が一番好みだったのですが、日本語対応なし、PDF出力は有料、拡張性はほぼなし、といった感じだったのでちょっと導入できなそうです。
