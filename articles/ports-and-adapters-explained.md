---
title: "Ports & Adapters パターン：Hexagonal Architecture Explained を手元に"
emoji: "❄️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["アーキテクチャ", "ヘキサゴナルアーキテクチャ", "ポートとアダプター"]
published: false
---

## 概要

- Ports & Adapters パターンについての概説
- Ports & Adapters のメリットとデメリットについて
- Ports & Adapters（ヘキサゴナルアーキテクチャ） の主要な考え方についての解説

## 対象読者

- アーキテクチャ初心者の方
- Ports & Adaptersについて知りたいが、まだなんのとっかかりもない方
- オブジェクト指向言語での開発経験がある方
- フランクな文章に抵抗のない方（重要）

## まえがき

アーキテクチャの勉強をするとさっそくその知識を活かして何か作ってみたくなりますよね。

そんでいざやってみると「このパターンの適応方法、本に載ってない！」とか「読んだらわかった気になったけど全然わかってなかった」という状況になるものです。

筆者は『クリーンアーキテクチャ』[^1]から入ったんですが、最初は「リスコフの置換原則」の理解に苦しんだ覚えがあります（今でも理解しているかわからないんですけど）。

<br>

というわけでこの記事ではアーキテクチャを扱っていきます。

いろんなアーキテクチャ思想がありますが、今回話していくのは題名にもあるとおり「**Ports & Adapters パターン**」についてです。

もしかすると「ヘキサゴナルアーキテクチャ」という呼び方の方が一般的かもしれません。筆者もちゃんと学び始めるまでこういう名前があることを知りませんでした。

アーキテクチャにもいろいろありますが、今回「Ports & Adapters」を取り上げようと思ったのは、自分が今一番汎用性が高く、意図が明確なアーキテクチャだと思っているからです。理由は後述するつもりです。

ところで「P＆A」って略してもいいですか？毎回「Ports ＆Adapters」って書くのがしんどいので。

公式の略し方ではないので、そこだけは注意。

<br>

本稿はP＆Aの提唱者である Alistair Cockburn[^2] さんが Juan Manuel Garrido de Paz さんと共著で出版した *Hexagonal Architecture Explained*[^3] という書籍に多くを負っています。

2024年にはプレビュー版が出版されていたんですが、最近になって正式版が出版されました（著者は間違ってプレビュー版を最初に買ってしまいました。）。気になる方は購入してみてください。

ただし、Amazonで買うとKindle版とペーパーバック版でえらい金額が違いますので（根っから紙派である著者も断念するレベル）、手頃なKindle版がおすすめです。

:::message
#### Ports & Adapters という名称について

この記事では「ヘキサゴナルアーキテクチャ」ではなく「Ports & Adapters」という名称の方を使っていきます。

イキっているのではなくて、提唱者の Alistair Cockburn さんが書籍の中でこっちの呼び方の方がより正しいと考えているからです[^4]。
:::

## Ports & Adapters とは

P&A は前述の通り Alistair Cockburn さんが先頭に立って提唱したアーキテクチャ思想です[^5]。オブジェクト指向言語を使ったプロジェクトがはまりやすい落とし穴を避けるために考えられました。

落とし穴と言われてもピンとこないかもしれませんので、従来のアーキテクチャを引き合いに出しながら説明してみます。

### レイヤードアーキテクチャ

レイヤードアーキテクチャという単語はWeb開発の入門書などにもよく出てくるので聞いたことがある方も多いことでしょう。

その名の通り、「役割ごとに階層で区切ってわかりやすくしよう」というアーキテクチャの考え方です。

一番有名なのは「プレゼンテーション層」、「ビジネスロジック層」、「インフラストラクチャ層」の３つで区切った「**３層アーキテクチャ**」でしょうか。

![](/images/ports-and-adapters-explained/3-layered-architecture-01.png)
*3層アーキテクチャ*

ものによってこれより層が増えたり減ったりします。

レイヤードアーキテクチャの基本的な考えは「役割の明確化」と「依存関係の簡素化」です（ですよね？）。

それぞれの層が特定の役割を担い、各層は一つ下の層にのみ依存します。

![](/images/ports-and-adapters-explained/3-layered-architecture-02.png)
*レイヤードアーキテクチャの依存関係と役割（矢印は依存の方向を示す）*

「なんて単純明快なんだ！」と思ったそこのあなた。これは理想の話に過ぎません。実際にはさまざまな問題が浮上します。

#### 柔軟性が低い

単純にデータをリストアップして表示する場合を考えてみてください。ただデータを取ってきて表示するだけならプレゼンテーション層で済ませたくなります。

ですがデータの保存や取得はインフラストラクチャ層が担っているので、プレゼンテーション層が勝手にデータベースにアクセスしたり、インフラストラクチャ層に依存したらルール違反です。

![](/images/ports-and-adapters-explained/3-layered-architecture-03.png)
*プレゼンテーション層が役割を飛び越えている*

なので、ただデータを取ってくるだけのうっすいビジネスロジック層の処理を実装して、それをプレゼンテーション層から利用しなければなりません。

もちろん、プレゼンテーション層からインフラストラクチャ層を利用することを許可するという手もありますし、それありきでレイヤードアーキテクチャを紹介している記事もよく見かけます。

確かに依存の方向は変わらないのでそこまで違反しているようには見えないのですが、怖いのは次の問題の温床になりかねないことです。

#### 他の層に役割が滲み出る

レイヤードアーキテクチャの目的は「役割ごとに層に分ける」ことにあると説明しました。しかしこの目的がちゃんと達成されることはほとんどありません。

人間は楽をしたがる生き物です。先ほどプレゼンテーション層からインフラストラクチャ層を利用する話がありましたが、これを容認すると次はプレゼンテーション層にビジネスロジックが漏れ出すことがあります。

「データが手元にあるならここでいじっちゃえ」という感じで、ビジネスロジック層を挟まずにプレゼンテーション層でデータを整形して返したり保存したくなってしまうのです。

プレゼンテーション層の役割は本来ユーザーとシステムの仲介なので、データを受け取って渡す以外のことはしてはいけません。証券取引所の窓口の人が預かったお金をトレーダーを挟まずに勝手に投資に回してしまったらゾッとしますよね。

プレゼンテーション層だけではありません。例えばトランザクションをビジネスロジック層で管理していたらそれはインフラストラクチャ層の関心ごとが漏れ出ていることになりますし、インフラストラクチャ層で税率計算をしていたら後々ビジネスロジックを変更した時に恐ろしいことになります。

![](/images/ports-and-adapters-explained/3-layered-architecture-04.png)
*あーあ、なにがなにやら...*

#### 構造から何も伝わってこない

各層の関心ごとが他の層に飛び出てしまうのは、そもそもレイヤードアーキテクチャの考え方そのものが原因かもしれません。

なにせ、レイヤードアーキテクチャは「**なにをするのか**」は伝えていても、「**なにがしたいのか**」は伝えてこないのです。



[^1]: [Robert C. Martin, *"Clean Architecture: A craftman's guid to software structure and design"*. Pearson, 2017.(Martin, R. C. (2018). *Clean Architecture: 達人に学ぶソフトウェアの構造と設計*.  角征典, 高木正弘)](https://www.google.com/aclk?sa=L&ai=DChsSEwiRj6Lo0YWQAxWa5RYFHZNWBpwYACICCAEQAxoCdGw&co=1&gclid=CjwKCAjwxfjGBhAUEiwAKWPwDv3rGocHrMONvMWxg9VRvXFBWzmihPWB-SEg6HBUG094vA5jdzKeMxoCBcQQAvD_BwE&cid=CAASWuRoiPcxyxnd4wUgZY9uwandH9HWJn0QXBvBY1pqnHsAHI0KWLA16NGms1Oc5aNEXbV9yh9RTfWR674Khvt9TON3SqlwM4yLWli8YuyJgQTIgtbQj4Xf84sRGw&cce=2&sig=AOD64_1jInmUCLok3EWL5jGp_d2NPnC51g&ctype=5&q=&ved=2ahUKEwiM25vo0YWQAxWYkq8BHVfKFrEQ5bgDKAB6BAgwEBQ&adurl=)

[^2]: [Alistair Cockburn](https://en.wikipedia.org/wiki/Alistair_Cockburn)

[^3]: [Alister Cockburn & Juan Manuel Garrido de Paz, *"Hexagonal Architecture Explained: How the Ports & Adapters architecture simplifies your life, and how to implement it, Updated 1st Ed"*, Humans and Technology Inc, 2025](https://www.amazon.co.jp/Hexagonal-Architecture-Explained-architecture-simplifies-ebook/dp/B0F5F7YRWW/ref=sr_1_1?__mk_ja_JP=%E3%82%AB%E3%82%BF%E3%82%AB%E3%83%8A&crid=2SRLO0SZKHZ09&dib=eyJ2IjoiMSJ9.wWZov3TQ81PRlPppjne2os_wkoULT4185oOY4nIqBf0.GG0icor8DmuHR28CjCyRwfGGKgd6zhzARs_cDTpu_Ik&dib_tag=se&keywords=Hexagonal+Architecture+Explained&qid=1759412912&s=english-books&sprefix=hexagonal+architecture+explained+%2Cenglish-books%2C206&sr=1-1)

[^4]:
    > "Hexagonal Architecture" has served well as a hook to the pattern. It's easy to remember and generates conversation. However, in this book we want to correct: The name of the pattern is "Ports & Adapters", beacause there really are ports, and there really are adapters, and your architecture will show them.

    *"Hexagonal Architecture Explained"*, pp. 32

[^5]: [Alistair Cockburn, *"Hexagonal architecture the original 2005 article"*, Alistair Cockburn (pronounced Cōburn) The Original site, https://alistair.cockburn.us/hexagonal-architecture, Feb. 2005 (accessed Oct. 2025)](https://alistair.cockburn.us/hexagonal-architecture)