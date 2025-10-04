---
title: "Ports & Adapters パターン：Hexagonal Architecture Explained を手元に"
emoji: "❄️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["アーキテクチャ", "ヘキサゴナルアーキテクチャ", "ポートとアダプター"]
published: true
---

:::message
誤字や脱字、説明や引用の誤りなどに気づいた方がいれば気兼ねなくコメントにてご連絡ください。

コメント内での議論は一向に構いませんが、筆者以外に対する読者への誹謗中傷などはご遠慮ください。

この記事に掲載されているコードは自由に利用してもらってかまいません。

リンクの共有・引用なども大歓迎です。

記事の中で使用している画像などは特段記載のない限り、筆者が自身で作成したものになります。
:::

## 概要

- Ports & Adapters についての概説
- Ports & Adapters のメリットとデメリットについて
- Ports & Adapters の主要な考え方についての解説

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

ところで「P&A」って略してもいいですか？毎回「Ports ＆Adapters」って書くのがしんどいので。

公式の略し方ではないので、そこだけは注意。

<br>

本稿は P&A の提唱者である Alistair Cockburn[^2] さんが Juan Manuel Garrido de Paz さんと共著で出版した *Hexagonal Architecture Explained*[^3] という書籍に多くを負っています。

2024年にはプレビュー版が出版されていたんですが、最近になって正式版が出版されました（著者は間違ってプレビュー版を最初に買ってしまいました）。気になる方は購入してみてください。

ただし、Amazonで買うとKindle版とペーパーバック版でえらい金額が違いますので（根っから紙派である著者も断念するレベル）、手頃なKindle版がおすすめです。

:::message
**Ports & Adapters という名称について**

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

特にビジネスロジック層に当てはまります。どこまでが「ビジネスロジック」なのか、アプリケーション開発経験が少ない人ほど決断に迷うことになります。

「商品が発送されたらメールを送る」という仕様があるとして、言葉通りの実装を行うとついついメールの送信ロジックまでもビジネスロジック層に実装してしまいがちです。

「メールの送信」というのがビジネスルールにあるとしても、SMTPの処理やライブラリの利用は本来インフラストラクチャ層で実装すべきものです。

どこまでがインフラで、どこまでがプレゼンテーションなのか、「ビジネスロジック」という名称は明確に答えてくれません。

構造からシステムの意図が伝わってくるアーキテクチャをクリーンアーキテクチャでは「**叫ぶアーキテクチャ**」と呼ばれます。

レイヤードアーキテクチャではアーキテクチャは自身の意図を叫んでいるでしょうか。

:::message
今回は多くのプロジェクトで採用されていることからレイヤードアーキテクチャを例に挙げました。

短所の説明ばかりになってしまいましたが、筆者にレイヤードアーキテクチャを否定する意図はありません。むしろ小規模なプロジェクトで採用する分には理にかなった選択肢になりえます。

アーキテクチャにはそれに適した状況というものがあります。アーキテクチャの採用について考える時にはそれぞれのメリットとデメリットを考慮した上で選定することが重要です。
:::

#### まとめ

- レイヤードアーキテクチャというトップダウン式のアーキテクチャがよく採用されてきた
- レイヤードアーキテクチャは簡潔でわかりやすい構造をしているが、複雑なシステムに適用するには以下の弱点がある
    - 柔軟性が低い
    - 他のレイヤーに役割が滲み出る（レイヤー違反）
    - 構造からなにも伝わってこない

### 改めて Ports & Adapters とは

従来のレイヤードアーキテクチャの短所についてみていきました。

- 柔軟性が低い
- 他の層に役割りが滲み出る
- 構造から何も伝わってこない

レイヤードアーキテクチャは概念としては理解しやすいので小規模なプロジェクトには向いているのですが、複雑な要件のプロジェクトで採用するには少し力不足なところがあります。

P&A はこのようなレイヤードアーキテクチャの短所を補い、柔軟性の高い、クリーンなコードベースを構築する方法を提供します。

そのためにも重要な概念が、このアーキテクチャの名前にも含まれている「**ポート**」と「**アダプター**」です。

#### ポート（Port）

![](/images/ports-and-adapters-explained/port-image.png =300x)
*LANポート（Chat-GPT より生成）*

ポートは「LANポート」や「ポート番号」など、IT系の用語でよく登場します。英語では「港」を指しますが、おそらくデータや通信の出入り口として比喩的にこのような命名がされたのでしょう。

しかし P&A でのポートとは、「港」というより、「LANポート」などの通信機器の接続部のような意味合いが強いように感じます。

ポートはアプリケーションとの接合部で、アプリケーションとやり取りをする際には必ずポートを経由します。

#### アダプター（Adapter）

![](/images/ports-and-adapters-explained/adapter-image.png =300x)
*電源アダプター（Chat-GPT より生成）*

アダプターという言葉からすぐに連想するのは「電源アダプター」かと思います。

スマートフォンの電源ケーブル（USB）だけでは充電はできません。充電するにはコンセントから電気を受け取り、適した電圧に変換してくれるアダプターが必要になります。

adapter の語源である **adapt** には「適合させる」、「当てはめる」などの意味があります。**形や規格の異なるものの仲介**をして利用可能にするのがアダプターの役割です。

:::message
「アダプター」という概念をシフトウェア開発に流用したのは P&A が初めてではありません。元は『**デザインパターン**[^6]』に遡ります。

『デザインパターン』ではオブジェクト指向言語においてオブジェクトの拡張性や再利用性に焦点を当てた設計パターンについて紹介されています。
:::

#### ポートとアダプター それぞれの役割

ポートとアダプターというだけでなんとなく役割は思い浮かびそうですが、なぜこのような概念を用いているのかも含めてここで解説しておきます。

ポートは外側と内側の接合部の役割を果たします。外側とはアプリケーションに直接関係のない外部の依存や、アプリケーションを利用するユーザーなどのことです。ということは、「内側」はアプリケーション本体であるとわかりますね。

P&A において、外部とのやりとりは全てポートを介して行われます。

アダプターはこの原則に則ってシステムを構築するために重要な役割を果たします。

大概のライブラリやフレームワークはポートにそのまま合致するような仕様やインターフェースにはなっていません。そのままではアプリケーションと連携が取れないので、誰かがアプリケーションと外部ツールの仲立ちをしなければなりません。

もうわかりますね。この仲立ちを担うのがアダプターです。

#### Ports & Adapters の基本姿勢

先に述べてしまいましたが、P&A では従来のレイヤードアーキテクチャの上下の関係とは対照的に、内外の関係性に基づいて設計します。

下の図において、矢印は依存の方向を示します。

レイヤードアーキテクチャでは依存の方向が上下一方通行になるように構成されていました。

![](/images/ports-and-adapters-explained/layered-arcgitecture-05.png)
*レイヤードアーキテクチャ*

P&A では依存の流れがポートを介して「内から外」「外から内」のに方向になります。

そしてアダプタがポートと外部の仲介を行います。

![](/images/ports-and-adapters-explained/ports-and-adapter-01.png)
*P&Aのイメージ*

パッとみただけではむしろ複雑性が複雑生が増している様にも見えますね。

しかし、レイヤードアーキテクチャの説明で述べた通り、システムの複雑性が増すとレイヤードアーキテクチャの長所である簡潔さが失われやすくなります。

これはレイヤードアーキテクチャの構造そのものが仕様の説明性に欠けるからであると言いました。

ですから P&A では内と外ではっきりと役割を決めています。

#### アプリケーション

「アプリケーション」（レイヤードアーキテクチャにおけるビジネスロジック層に相当）では外部の使用に依存してはいけません。必ずポートの仕様に依存する様にします。

つまり**言語使用のみで独自のロジックを実装する**ことになります。

例えば Laravel で P&A を実装するとして、アプリケーションの中で `DB` ファサードなどを使うのは NG です。

判断に迷ったら「この機能をアプリケーションロジックで使っても別のフレームワークで動かせるだろうか」と考えるのが良いでしょう。

可搬性は P&A において重要な観点になります。そのためには「アプリケーションが外のことについてなにも知らない」という状態が望ましいのです。

#### ポート「私だけを見ていて」

とは言いつつ、アプリケーション単体ではロクなシステムが作れないので、なんらかの方法で外部と連携をとる必要が出てきます。

ポートはアプリケーションが外部と間接的にやり取りをするためにあります。

ポートはアプリケーションにとってのインターフェースです。それどころかアプリケーションはポートとしかやり取りをしません。

「アプリケーションはポートとのみやり取りをする」という原則を忠実に守ることで、アプリケーションが外部の依存に汚染されることを防ぎます。

#### まとめ

- P&A では「ポート」と「アダプター」の概念が重要になる
- 「ポート」はアプリケーションと外部の接続部の役割を果たす
- 「アダプター」は外部とポートの仕様の違いを吸収する
- 「アプリケーション」は外部ツールなどに依存せず、純粋に言語使用のみで構築する
- アプリケーションが外部と連携するときは必ずポートを経由して行う

## 基本用語

P&A の基本的な考え方を紹介したところで、ここで基本的な用語について解説していきます。

登場する用語はそこまで多くありませんので身構える必要はありません。

先に用語を一覧にして載せておきます。

- Application
- Driving Port (Primary Port)
- Driven Port (Secondary Port)
- Driving Adapter (Primary Adapter)
- Driven Adapter (Secondary Adapter)
- Configulater

### Application

**Application** はすでに何度も登場していますね。

Application はシステムのビジネスロジック担当で、外部への依存性を一切遮断します。

外部と連携するときはポートに処理を依頼します。

P&A では Application 自体に適用するアーキテクチャやディレクトリ構成を強制していません。純粋に保ちさえすれば、どのように構成しても構わないことになっています。

例えば、ドメイン駆動設計（DDD）を適用したり、独自の構成でレイヤードアーキテクチャを採用することもできます。

筆者は個人的にDDDを採用することが多いです。

### Driving Ports (Primary Ports)

ポート自体の概念はあらかじめ解説しましたが、ここでは Driving という言葉が付け加えられました。

Port 自体の意味は変わりません、ポートは Application と外部の仲介役を担います。

Driving という言葉はここでは Application との関係の方向性を表しています。

**Drinving Ports** は Application を利用する外部が接続するポートのことです。**Primary Ports** とも呼ばれます。

下記の図では左側にあるポートが　Drinving Ports になります。

![](/images/ports-and-adapters-explained/ports-and-adapter-02.png)
*Web や CLI がポートを経由して Application の処理を利用している*

以降のに登場する Driving や　Drivin という単語も Application との関係性の文脈から使われています。

### Driven Ports (Secondary Ports)

**Driven Ports** は Drinving Ports とは対照的に、Application が外部の機能を利用する際に仕様するポートのことです。**Secondary Ports** とも呼ばれます。

データベースやファイルシステムを使用する際は Application から Drinven Ports を利用して連携をとります。

下記の図では右側のポートが Drivin Port ということになります。

![](/images/ports-and-adapters-explained/ports-and-adapters-architecture-03.png)
*Application がポートを経由して外部と連携をとる*

### Driving Adapters (Primary Adapters)

ここまで説明した内容からおおよそ理解できるかと思います。

**Driving Adapters** は Driving Ports と外部を仲介するアダプターのことです。**Primary Adapters** とも呼ばれます。

下記の図では左側のアダプターが Drinving Adapters になります。

![](/images/ports-and-adapters-explained/ports-and-adapters-architecture-04.png)
*アダプターが Driving Ports と外部の連携を行う*

### Driven Adapters (Secondary Adapters)

Drinving Ports と Driven Ports の関係と同じように、**Drinven Adapters** は Driving Adapters とは対称関係にあります。

Driven Adapters は Driven Ports と外部の連携を行います。

下記の図では右側のアダプターが Drinven Adapters です。

![](/images/ports-and-adapters-explained/ports-and-adapters-architecture-05.png)
*アダプターが Driven Ports と外部の連携を行う*

### Configulater

新しい用語が出てきました。

**Configulater** は P&A において影の立役者的な立ち位置にあります。

「Application はポートを介してのみ外部と連携をとる」「Application は外部についての知識を持ってはならない」という原則を守るためには、どうしても外からの介入が必要になります。その介入を行うのが　Configulater のに方向になります役割です。

データベースを利用する際を例に挙げましょう。

Application は「どんなデータベースを使うか」「どんなドライバを使うか」について知っていてはいけません。あくまで「ポートに接続された機能からデータを受け取る（ポートが提供するインターフェースを実装した処理を利用する）」という態度に徹しています。

ですが現実問題、データベースとやり取りするためにはドライバーやライブラリを利用せざるを得ません。ですから何らかの方法でポート（Driven Port）と連携してデータベースとやり取りを行う処理を実装する必要が出てきます。

Configulater はこの時に Application がどの処理を利用するか決定する役割を持ちます。

![](/images/ports-and-adapters-explained/configulater.png)


:::message
**Driving / Drinven**

英語としての **drive** には「運転する」「作動させる」という意味があります。

P&A の文脈においてこの単語は Application から見た利用の方向性を表すために使われています。

Deiving であれば「Application を**利用する**」ためのポートやアダプター、Driven は「Application から**利用される**」ポートやアダプターに対して使われます。

![](/images/ports-and-adapters-explained/driving-and-driven.png)
:::

## テレビ家電を例に

基本用語の解説が終わったところで、P&A のイメージを具体化するために、ここで家電を例に解説して見たいと思います。

インターネットに接続することで Net◯lix や App◯e TV が鑑賞できるデバイスをイメージしてください。

![](/images/ports-and-adapters-explained/tv-device.png =300x)
*Chat-GPTより生成*

このデバイスにはLANポートや電源ポート、HDMIポートやUSBポートが付属しています。

これらの各種ポートが P&A でのポートに当たります。

さらに Bluetooth 接続機能がついていて、アプリやリモコンから操作が可能です。これらも一種のポートと言えるでしょう。

まずはデバイスを起動するために、電源を繋げなければいけません。サービスを利用するにはLANケーブルを繋いでインターネットに接続することも必要です。

電源やインターネットにとってはこのデバイスは利用者にあたります。つまり Driven Ports を経由して接続する外部です。Driven Ports は電源ポートとLANポートに当たります。

次に映像を鑑賞するためにHDMI、あるいはUSBを接続する必要があります。操作にはBluetoothに繋ぐことも必要です。テレビであれ他のディスプレイであれ、このデバイスを利用することになるので、P&A 的に言えば Driving Ports に接続する外部になります。Driving Ports はHDMIポートやUSBポート、Bluetooth が対応します。

ではアダプターはどれに対応するのでしょうか。

例えば Type-C のUSBポートしかないノートパソコンでデバイスを利用する場面を考えて見ましょう。このPCはそのままではHDMLにもUSBポートにも繋ぐことができません。

そんな時、皆さんなら Type-C 変換アダプターを使いますよね？その変換器が P&A におけるアダプター（Driving Adapters）の立ち位置になります。

他にも海外でこのデバイスを利用したときに、電源コンセントが持っている電源プラグの規格と合わなかったら専用の変換器を使うと思います。こちらの場合はこの変換器が Driven Adapters ということになります。

ところで、どんなアダプターを使うか、どんな周辺機器とデバイスを繋ぐかを判断していたのは誰でしょうか。もちろんこのデバイスのユーザーですね。このユーザーは Configulater の役割を果たしていたということになります。


## 基本的な処理の流れ

抽象的な話が続いたので少し具体的に P&A で実装されたシステムの処理の流れについて見ていきましょう。

あくまで例ですので、実際の開発ではチームの開発スタイルや個人の好みに合わせて実装してください。

### 1. Configulater で依存関係を解決する

はじめにシステムの初期化が走ります。ここは Configulater の出番です。

Application が実際の処理を走らせるためには Application が利用するポートに実際の機能を接続しなければなりません。

システムは Configulater にオブジェクトやインスタンス化の方法などを登録し、後々 Application が利用できる様に準備をします。

### 2. Application の利用

Application の初期化が済み、ユーザー（あるいは別システム）がシステムに処理を依頼します。

システムは Application の機能を利用するため、ポート（Driving Ports）の処理を呼び出します。もし処理が別システムからの依頼であれば、ポートの仕様に適合させるためにアダプターを介入させるかもしれません。

ポートはシステム（あるいはアダプタ）から依頼された処理内容を元に Application のどの機能を呼び出すか決定し、処理の継続を依頼します。

### 3. Application の処理

Application は依頼された処理を行うために、Application 内部の処理（一般的にアプリケーションサービス）を呼び出し、結果を受け取ります。

### 4. Configulater の介入

Application は処理内容を保存（あるいは出力）するためにポート（Driven Ports）に処理を問い合わせます。

この時、Configulater によって Application が利用する機能が指定され、実際に外部の処理が呼び出されます。

### 5. 処理の完了

Driven Ports の処理が完了したら、Application は適宜 Driving Ports に処理結果や終了応答を返します。

## なぜ Ports & Adapters パターンを使うのか

他のアーキテクチャに馴染みのない方は　P&A にどんなメリットがあるのか想像しにくいかもしれません。

レイヤードアーキテクチャに慣れている人は P&A が奇妙な構成に見えてしまいがちです。

なのでここで一度 P&A のメリットについてお話ししておこうと思います。

### Ports & Adapters のメリット

#### 可搬性が高い

P&A ではアプリケーションのロジックが内部にカプセル化され、利用側はポートの使用に合致しないとロジックを利用することができません。

これは一見不便にも見えますが、Application が内部で保護されていることによって別のプロジェクトでの流用が容易になります。

レイヤードアーキテクチャでは各レイヤーが純粋に保たれることは強制されませんので、レイヤーを他のプロジェクトに移動するとレイヤー内部で使用されている外部依存も一緒に移動してくることになります。

その時、もしその外部依存が特定の環境に依存する構成になっていたとしたら、移動先の環境でも同じ環境を再現する必要が出てきます。それがどんなに大変なことかはある程度開発経験を積んだことのあるエンジニアであれば想像に難くないでしょう。

しかし P&A ではアプリケーションのコア部分が外部に依存しない構成になっているので、そのプログラミング言語を実行できる環境であればどこにでも持っていけます。

#### 拡張性が高い

P&A の Application は外部に関心を持たず、ただ純粋にビジネスロジックに従事し、必要とあればポートを介して外部と連絡を取ります。

ポートによって外部連携が抽象化されているので、ポートに適応する形で実装できるものであればどこでもアプリケーションのロジックを了することができます。

例えば Web アプリケーションで利用していたロジック使ってネイティブアプリケーションを実装することも可能です。

#### 自動テストが容易

Application のカプセル化は自動テストを行う上でもアドバンテージになります。

外部依存がないので Application 単体でテストを実装できます。

また、外部連携のテストの際もポートの仕様に合ったテストダブルを用意すれば良いので、稼働中の動きをトレースしやすくなります。

#### システム間の連携が行いやすい

ユーザーが利用するシステムだけでなく、システムから利用されるサービスを実装する際にも P&A は有用です。

あるシステムに機能を追加するよりも、別個のシステムとして切り出してしまった方が管理が楽になる場合があります。その時に、新しく追加するシステムを P&A を使って構築します。

もし元のシステムの方の仕様が変わって、P&A のシステムのポートに適合しなくなった場合は、アダプターを使って差異を吸収し、継続してシステムを利用できるようになります。

また、全く別のシステムでも利用したくなった時も、P&A システムはそれ自身独立しているので、別のシステムを気にかけずに利用することができます。

#### 分担して開発が行いやすい

ポートの仕様が明らかになっていれば、アプリケーションロジックの実装、データベースによる永続化ロジックの実装、UIの実装など分担して開発が行える様になります。

一つのチーム、あるいは一人のエンジニアが複数の関心ごとを跨いで開発していると気にかけることが多くなって開発効率が低下し火事ですが、P&A によって関心事が切り分けられていると、一つの関心ごとに集中して取り組めるので開発効率が上がります。

また、同じ規約をもとに開発を行なっているので、結合テストの際もスムーズに連携が行えます。

### P&A のデメリット

このように P&A には様々なメリットがあります。

しかし、やたらめったらに採用しても望んだメリットは享受できません。P&A を実現するにはそれなりのオブジェクト指向言語に対する理解と経験値が求められます。

それに、実現できたとしても万事OKとはなりません。P&A にもある程度のデメリットがあります。

P&A のデメリットについてもここで説明しておきましょう。

#### 学習コストが高め

P&A は前提としてオブジェクト指向言語を想定したアーキテクチャパターンです。

ある程度オブジェクト指向言語への理解がなければ実装するのは難しいでしょう。

P&A について知らず、オブジェクト指向言語にも慣れていないプロジェクト新規参入者にとってはプロジェクト内で何が行われているのか理解することは困難になります。

絶えずチームで認識を共有し、新規参入者の教育を積極的に行う姿勢が求められます。

#### 小規模プロジェクトには不向き

P&A が威力を発揮するのは中規模程度から大規模までのプロジェクトにおいて、あるいは今後継続的な拡張が予定されているプロジェクトにおいてです。

小規模なプロジェクトで採用しても不要な複雑性を持ち込むことになりかねません。

規模が小さい間やプロトタイプの開発の際は単純なレイヤードアーキテクチャなどを採用し、規模が拡大する段階で P&A への移行を検討しましょう。

#### 処理の流れが追いにくくなる

Configulater が依存関係を解決する都合上、どのクラスが呼ばれてどの処理がよばれているか、把握しづらくなることがあります。

ですのでできる限り Configuater の実装と Application 内での依存解決をシンプルに保つことが必要です。

チーム内で使い慣れているDIライブラリなどがあればそれを採用するのもありでしょう。しかしそうなると Application に外部依存性を持ち込むことになるので、採用は慎重になるべきです。

### まとめ

- P&A のメリット
    - 可搬性が高い
    - 拡張性が高い
    - 自動テストが容易
    - システム間の連携が行いやすい
    - 分担して開発が行いやすい

- P&A のデメリット
    - 学習コストが高め
    - 小規模なプロジェクトには不向き
    - 処理の流れが追いにくくなる

## TypeScript を使ったサンプルプロジェクト

最後に実際のコードで P&A の具体例を見ていきましょう。

今回はみなさんお馴染みのTODOアプリケーションを例にしてみます。

知っている人が多いかと思ったので、TypeScript を使いました。別の言語を主戦力にしている人は適宜頭の中で読み替えてみてください。

実装する内容は以下です：

- TODOを追加する
- TODOを取得する
- TODOを完了する
- TODOを削除する

### ポートの実装

まずは Application と外部の橋渡し役であるポートから実装していきます。

実際の開発でも、作業を分担する際はまず契約となるポートの仕様から決めて行くと良いと思います。

Application が実装されるまで待ちが発生してしまってはせっかくの P&A の強みが活かされないので勿体無いです。

今回はポートは `app` というディレクトリを作り、その下の `ports` というディレクトリに配置する様にします。

これが正解というわけではないので皆さん好きな名前を使ってください。

```
.
└── app/
    └── ports
```

#### Driving Ports の実装

まずは Application を利用するための接続部にあたる　Deriving Ports を実装していきます。

Driving Ports は `driving` というディレクトリに配置することにします。

```
.
└── app/
    └── ports/
        └── driving
```

今回は「TODOの追加」「TODOの完了」「TODOの削除」という３つの機能を提供するので、それぞれに対応するポートを定義していきます。

*Hexagonal Architecture Explained* ではポートの名前を `for_doing_something` の形式で定義することが推奨されています。今回はそれに従いましょう。

```
.
└── app/
    └── ports/
        └── driving/
            ├── for-adding-todo.ts
            ├── for-completing-todo.ts
            ├── for-deleting-todo.ts
            └── for-getting-todo.ts
```

さらにポートで受け渡しするデータの定義が必要です。

データを運搬する目的のみで使われるオブジェクトを **DTO (Data Transfer Object)** といいます。

DTOを定義する場所として `ports` の配下に `dto` というディレクトリを設けましょう。

```
.
└── app/
    └── ports/
        ├── driving/
        │   ├── for-adding-todo.ts
        │   ├── for-completing-todo.ts
        │   ├── for-deleting-todo.ts
        │   └── for-getting-todo.ts
        └── dto
```

DTOの実装方法はいろいろありますが、今回は単純なオブジェクトとして定義しましょう。

`dto` にTODOのデータ型を以下のように追加しました。

```
.
└── app/
    └── ports/
        ├── driving/
        │   ├── for-adding-todo.ts
        │   ├── for-completing-todo.ts
        │   ├── for-deleting-todo.ts
        │   └── for-getting-todo.ts
        └── dto/
            └── todo-dto.ts
```

```ts:app/ports/dto/tood-dto.ts
export type TodoDTO = {
    readonly id: string;
    readonly title: string;
    readonly done: boolean;
};
```

DTO あくまでデータ運搬のための型ですので、データの中身が変更されるようなライフサイクルは想定されていません。

なのでプロパティは `readonly` にしておきます。

データ型を定義したので次はポートを定義します。

```ts:app/ports/driving/for-adding-todo.ts
export interface ForAddingTodo {
    addTodo(todo: TodoDTO): TodoDTO;
}
```

```ts:app/ports/driving/for-completing-todo.ts
export interface ForCompletingTodo {
    completeTodo(id: string): TodoDTO;
}
```

```ts:app/ports/driving/for-deleting-todo.ts
export interface ForDeletingTodo {
    deleteTodo(id: string): void;
}
```

```ts:app/ports/driving/for-getting-todo.ts
export interface ForGettingTodo {
    getTodo(id: string): TodoDTO;
}
```

これで Driving Ports の定義は完了です。

先に定義を作っておいたことで、ここからは別のエンジニアやチームが個別に開発を進められる様になります。

#### Driven Ports の実装

次に Driven Ports を定義しましょう。

ディレクトリは `driven` にします。

各ポートの名前は「取得」「作成」「更新」「削除」の4つで命名しました。

```
.
└── app/
    └── ports/
        ├── driven/
        │   ├── for-creating-todo.ts
        │   ├── for-deleting-todo.ts
        │   ├── for-updating-todo.ts
        │   └── for-getting-todo.ts
        ├── driving/
        │       ...
        └── dto/
                ...
```

```ts:app/ports/driven/for-creating-todo.ts
export interface ForCreatingTodo {
    createTodo(todo: TodoDTO): TodoDTO;
}
```

```ts:app/ports/driven/for-deleting-todo.ts
export interface ForDeletingTodo {
    deleteTodo(id: string): void;
}
```

```ts:app/ports/driven/for-updating-todo.ts
export interface ForUpdatingTodo {
    updateTodo(todo: TodoDTO): TodoDTO;
}
```

```ts:app/ports/driven/for-getting-todo.ts
export interface ForGettingTodo {
    getTodoById(id: string): TodoDTO;
}
```

:::message
**ポートのディレクトリ名**

今回はポートのディレクトリ名に `driving` と `driven` というそのままの名前を使いましたが、他にもよく使われるものでは `inbound`/`outbound`、`use-case`/`repository` などがあります。

ちなみに筆者はよく `ports` というディレクトリだけ用意して、`use-case`/`export`/`repository`/`mailer`/`logger` など、より具体的な名前をつける様にしています。

名前から Driving なのか Driven なのかは明白だからです。
:::

### ユースケースの実装

ここまではデータ型とインターフェースしか定義していないので、このままでは使い物になりません。

ここからは Application 内部の実装を行なっていきます。

まずは Driving Ports に対応する処理を記述していきます。

Driving Ports が提供する機能はアプリケーションのロジックですので、`use-cases` というディレクトリを作ってそこに実際の処理を記述していきましょう。

```
.
└── app/
    ├── ports/
    │   ├── driven/
    │   │       ...
    │   ├── driving/
    │   │       ...
    │   └── dto/
    │           ...
    └── use-cases/
        ├── add-todo-use-case.ts
        ├── complete-todo-use-case.ts
        ├── delete-todo-use-case.ts
        └── get-todo-use-case.ts
```

```ts:app/use-cases/add-todo-use-case.ts
export class AddTodoUseCase implements ForAddingTodo {

    private _forCreatingTodo: ForCreatingTodo;

    public constructor(forCreatingTodo: ForCreatingTodo) {
        this._forCreatingTodo = 
    }

    public addTodo = (todo: TodoDTO): TodoDTO => {
        return this._forCreatingTodo.createTodo(todo);
    }
}
```

```ts:app/use-cases/complete-todo-use-case.ts
export class CompleteTodoUseCase implements ForCompletingTodo {

    private _forUpdatingTodo: ForUpdatingTodo;

    private _forGettingTodo: ForGettingTodo;

    public constructor(forUpdatingTodo: ForUpdatingTodo, forGettingTodo: ForGettingTodo) {
        this._forUpdatingTodo = forUpdatingTodo;
        this._forGettingTodo = forGettingTodo;
    }

    public completeTodo = (id: string): TodoDTO => {
        const todo = this._forGettingTodo.getById(id);
        todo.done = true;
        return this._forUpdatingTodo.updateTodo(todo);
    }
}
```

```ts:app/use-cases/delete-todo-use-case.ts
import { ForDeletingTodo as ForDeletingTodoDrivenPort } from "../ports/driven/for-deleting-todo.ts";

export class DeleteTodoUseCase implements ForDeletingTodo {

    private _forDeletingTodo: ForDeletingTodoDrivenPort;

    public constructor(forDeletingTodo: ForDeletingTodoDrivenPort) {
        this._forDeletingTodo = forDeletingTodo;
    }

    public deleteTodo(id: string): void {
        this._forDeletingTodo.deleteTodo(id);
    }
}
```

```ts:app/use-cases/get-todo-use-case.ts
import { ForGettingTodo as ForGettingTodoDrivenPort } from "../ports/driven/for-getting-todo.ts";

export class GetTodoUseCase implements ForGettingTodo {

    private _forGettingTodo: ForGettingTodoDrivenPort;

    public constructor(forGettingTodo: ForGettingTodoDrivenPort) {
        this._forGettingTodo = forGettingTodo;
    }

    public getTodo(id: string): TodoDTO {
        return this._forGettingTodo.getTodoById(id);
    }
}
```

### 永続化処理の実装

Driving Ports の実装が終わったので次は　Driven Ports の実装を行います。

Driven Ports のインターフェースを実装するのは Application 外部の処理ですので、`app` ディレクトリとは別の場所に置いておきましょう。

今回は `database` という名前にして、MySQL を利用する前提で実装します。

```
.
├── app/
│   ├── ports/
│   │       ...
│   └── use-cases/
│           ...
└── database/
    └── mysql
```

ファイルは Driven Ports に従って４つ用意します。処理内容は割愛します。

```
.
├── app/
│   ├── ports/
│   │       ...
│   └── use-cases/
│           ...
└── database/
    └── mysql/
        ├── create-todo-command.ts
        ├── update-todo-command.ts
        ├── delete-todo-command.ts
        └── get-todo-query.ts
```

```ts:database/mysql/create-todo-command.ts
export class CreateTodoCommand implements ForCreatingTodo {

    public createTodo(todo: TodoDTO): TodoDTO {
        // ...
    }
}
```

```ts:database/mysql/update-todo-command.ts
export class UpdateTodoCommand implements ForUpdatingTodo {

    public updateTodo(todo: TodoDTO): TodoDTO {
        // ...
    }
}
```

```ts:database/mysql/delete-todo-command.ts
export class DeleteTodoCommand implements ForDeletingTodo {

    public deleteTodo(id: string): void {
        // ...
    }
}
```

```ts:database/mysql/get-todo-command.ts
export class GetTodoQuery implements ForGettingTodo {

    public getTodoById(id: string): TodoDTO {
        // ...
    }
}
```

### Configulater の実装

次はここまで定義してきたポートとインターフェースの実装を結びつけるために　Configulater を実装します。

Configulater は アプリケーション側で提供するものなので `app` に作ります。

```
.
├── app/
│   ├── configulater.ts
│   ├── ports/
│   │       ...
│   └── use-cases/
│           ...
└── database/
        ...
```

```ts:app/configulater.ts
export class Configulater {

    private _drivenPorts: {
        forCreatingTodo: ForCreatingTodo;
        forUpdatingTodo: ForUpdatingTodo;
        forDeletingTodo: ForDeletingTodo;
        forGettingTodo: ForGettingTodo;
    }

    public constructor(drivenPorts: {
        forCreatingTodo: ForCreatingTodo;
        forUpdatingTodo: ForUpdatingTodo;
        forDeletingTodo: ForDeletingTodo;
        forGettingTodo: ForGettingTodo;
    }) {
        this._drivenPorts = { ...drivenPorts };
    }

    public drivenPorts = () => { ...this._drivenPorts };
}
```

### Application ファサードの実装

必要なものは全て実装したのですが、このままでは利用側が自分で　Driving Ports のインタフェースを解決しなければなりません。

なので利用者が Drinving Ports の具象オブジェクトを意識しなくて済むように**ファサード**を作りましょう。

```
.
├── app/
│   ├── configulater.ts
│   ├── facade.ts
│   ├── ports/
│   │       ...
│   └── use-cases/
│           ...
└── database/
        ...
```

```ts:app/facade.ts
export class Application {

    private _config: Configuater;

    public constructor(config: Configuater) {
        this._config = config;
    }

    public forAddingTodo = (): ForAddingTodo => {
        return new AddTodoUseCase(this._config.drivenPorts().forCreatingTodo);
    }

    public forCompletingTodo = (): ForCompletingTodo => {
        return new CompleteTodoUseCase(
            this._config.drivenPorts()forUpdatingTodo,
            this._config.drivenPorts().forGettingTodo);
    }

    public forDeletingTodo = (): ForDeletingTodo => {
        return new DeleteTodoUseCase(this._config.drivenPorts().forDeletingTodo);
    }

    public forGettingTodo = (): ForGettingTodo => {
        return new GetTodoUseCase(this._config.drivenPorts().forGettingTodo);
    }
}
```

### 実装したアプリケーションの使用

実装したアプリケーションを使用した例が以下になります。

```ts
const app = new Application(
    new Configuater({
        forCreatingTodo: new CreateTodoCommand(),
        forUpdatingTodo: new UpdateTodoCommand(),
        forDeletingTodo: new DeleteTodoCommand(),
        forGettingTodo: new GetTodoQuery(),
    })
);

const todo = app.forAddingTodo.addTodo({
    id: '',
    title: 'Todo1',
    done: false,
});

app.forCompletingTodo.completeTodo(todo.id);

app.forDeletingTodo.deleteTodo(todo.id);
```

もし異なるデータベースを使う様に変更する場合は Configulater に渡すインスタンスを変更するだけで済むことがわかります。

Driving Ports の実装に関しても、定義したインターフェースに従ってさえいれば自由に変更可能です。

ここで紹介した方法はあくまで簡単に P&A の実装の流れを解説するためのものなので、ご自身で実装する際は是非自分なりのいい方法を模索してみてください。

:::message
**ファサードとは**

ファサードも『デザインパターン』の実装パターンの一つです。

**facade** は英語で「正面」という意味です。ここでは「窓口」と考えた方が良いでしょう。

複数のクラスの処理を一つのクラスにまとめ、利用者側から裏側の依存関係やインスタス方法などを隠蔽するために使います。
:::

## 最後に

もっと短くまとめるつもりでしたが結構な文量になってしまいました。

筆者が Ports & Adapters に出会ったのは『実践ドメイン駆動設計[^7]』を読んでいた時でした。この書籍では P&A を前提に話が進められていたので、内容をよく理解するためにこのアーキテクチャの勉強を始めました。

しかしいざ勉強しようと思っても日本語の解説書がほとんどありません。唯一見つけたのは『手を動かしてわかるクリーンアーキテクチャ[^8]』という書籍で、この本にはかなり助けられました。皆さんにもぜひ一読していただきたです。

拙い解説でしたが皆さんが Ports & Adapters パターンの理解するにあたって一助となれば幸いです。


---

[^1]: Robert C. Martin, *"Clean Architecture: A craftman's guid to software structure and design"*. Pearson, 2017.(Martin, R. C. (2018). *Clean Architecture: 達人に学ぶソフトウェアの構造と設計*.  角征典, 高木正弘)

[^2]: [Alistair Cockburn](https://en.wikipedia.org/wiki/Alistair_Cockburn)

[^3]: Alister Cockburn, Juan Manuel Garrido de Paz, *"Hexagonal Architecture Explained: How the Ports & Adapters architecture simplifies your life, and how to implement it, Updated 1st Ed"*, Humans and Technology Inc, 2025

[^4]:
    > "Hexagonal Architecture" has served well as a hook to the pattern. It's easy to remember and generates conversation. However, in this book we want to correct: The name of the pattern is "Ports & Adapters", beacause there really are ports, and there really are adapters, and your architecture will show them.

    *"Hexagonal Architecture Explained"*, pp. 32

[^5]: Alistair Cockburn, *"Hexagonal architecture the original 2005 article"*, Alistair Cockburn (pronounced Cōburn) The Original site, https://alistair.cockburn.us/hexagonal-architecture, Feb. 2005 (accessed Oct. 2025)

[^6]: Erich Gamma, Richard Helm, Ralph Johnson, John Visside, *"Design Patterns: Elements of Reusable Object-Oriented Software"*, Addison-Wesley, 1994 (Erich Gamma, Richard Helm, Ralph Johnson, John Visside (1999). *オブジェクト指向における再利用のためのデザインパターン*. 本位田真一, 吉田和樹. SBクリエイティブ)

[^7]: Vaghn Vernon, *"Implementing Domain-Driven Design"*, Addison-Wesley Professional, 2013 (Vaghn Vernon (2024). *実践ドメイン駆動設計：エリック・エヴァンスが確立した理論を実際の設計に応用する*. 高木正弘. 翔泳社)

[^8]: Tom Hombergs, *"Get Your Hands Dirty on Clean Architecture: A hands-on guide to creating clean web application with code examples in Java"*, Packt Publishing, 2019 (Tom Hombergs (2024). *手を動かしてわかるクリーンアーキテクチャ：ヘキサゴナルアーキテクチャによるクリーンなアプリケーション開発*. 須田智之. インプレス)