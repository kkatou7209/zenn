---
title: "スクラム学習日記「スクラムの概要」"
emoji: "🏃‍♂️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["スクラム", "アジャイル開発"]
published: true
---

# この記事について

スクラム開発について勉強した内容をアウトプットしていく記事です。

スクラムにちょっと興味のある方、スクラムについて学びたいという方、スクラムの復習をしたい方にとって役に立つような記事になれば嬉しいです。

# まえがき

アジャイルという言葉は知っていたものの、今までちゃんと勉強したことがありませんでした。

以前は実際の開発経験がなく、ウォーターフォール型開発の欠点や適応型開発手法のメリットなどを肌感として理解できず、積極的に学ぶ動機がなかったからです。

しかし開発経験を一年程経験すると、手続的な開発方法に疑問を持つ様になり、「流行りの開発方法」程度でしか認識していなかったアジャイル開発手法が今の自分と同じような問題意識のもと生み出された手法であることが理解できるようになりました。

スクラムを学ぼうと思ったのはそのような動機があってのことですが、自分一人が理解したところで意味がないので、どうせなら自分も他の人も後から見返せる形で勉強した内容をまとめておこうと思った次第です。

<br>

では早速、スクラムとはなんなのか説明を始めていこうと思います。

# スクラムとは

スクラムとは[**アジャイル開発技法**][アジャイル]の一つです。

[**ジェフ・サザーランド**][Jeff Sutherland]と[**ケン・シュウェイバー**][Ken Schwaber]によって1990年に作られました。

[**野中郁次郎**][野中郁次郎]と[**竹内弘高**][竹内弘高]の共著で1986年にハーバード・ビジネス・レビュー誌に掲載された論文である [***"The New New Product Deelopment Game"***][The New New Product Deelopment Game] から着想を得たそうです。

「スクラム」という用語もその論文に登場しており、ラグビーのスクラムが由来になっていると言われています。

スクラムのルールは[公式][Scrum Guide]に定義されており、1〜2年おきに内容が更新されています。

従来のウォーターフォール型が「**要件定義→設計→実装→テスト**」というふうに進むのに対し、スクラムでは **スプリント** という決められた期間内で以上の工程を行い、それを繰り返しながら開発を進めます。

:::message
#### 他のアジャイル開発技法

アジャイル開発にはスクラムの他に [**エクストリームプログラミング(XP)**][エクストリーム・プログラミング] や [**カンバン**][カンバン] というものがあります。
:::

:::message
#### スプリント

*sprint* は日本語で **全力疾走**、**短距離走** という意味があります。

要件定義からテストまでの短い期間で行うという点では確かに短距離走に似ています。
:::


# スクラムの構成要素

スクラムは **軽量フレームワーク** と呼ばれている通り、単純な要素で構成されています。

それが [**3つのロール(役割)**](#3つのロール)、[**5つのイベント**](#5つのイベント)、[**3つの作成物**](#3つの作成物) です。


## 3つのロール

### プロダクトオーナー

- プロダクトの責任者
- プロダクトバックログ（[後述](#プロダクトバックログ)）を管理し、リストの並び順似ついての最終決定権を持つ
- ステークホルダーとの交渉や協業を行う
- プロダクトにつき一人

### 開発チーム

- プロダクト実装者
- 3〜9人で構成される
- 肩書などは無し
- プロダクトオーナーが決定したプロダクトバックログの内容を順に実装する
- 個人個人が複数のことを行えることが望ましい

### スクラムマスター

- スクラムのプロセスがうまく回るようにプロダクトオーナーと開発チームをサポートする
- タスクの割り振りや進捗管理は行わない
- チームが妨害されないように防波堤の様な立ち回りをする
- スクラムに慣れていないチームではトレーナーのような役割を果たす

:::message
以上の3つのロールを合わせて **スクラムチーム** と呼びます。
:::


## 5つのイベント

### スプリント

- 最大1ヶ月間の固定された期間
- 各スプリントはバラバラの期間には設定できない
- この期間で設計からテストまで全て行う
- 最終日に作業が残っている場合でも延長はしない

### スプリントプラニング

- プロダクトバックログを元にスプリント内で何を行うか決定する
- スプリントバックログ（[後述](#スプリントバックログ)）の作成

### デイリースクラム

- スプリントバックログの進捗と達成可能性を確認
- 毎日15分で行う
- 報告を主にして行い、問題解決のための会議は別途設定する

### スプリントレビュー
  
- スプリントの最後に行う
- ステークホルダーも招待
- プロダクトについてのフィードバックを行う
- インクリメントをもとに、実際にプロダクトを動作させて内容を説明する
- 実施時間はスプリントの長さによって増減する

### スプリントレトロスペクティブ

- スプリントレビューの後に行う
- うまく行ったことや改善点を整理
- 今後のアクションプランを作る

:::message

#### レトロスペクティブについて

*retrospective* には **回想する** という意味があります。

「後方を（*retro-*）」「見る（*-spect-*）」で「回想する」です。

レトロ（*retro-*）という部分はみなさんにも馴染みが深いかと思います。

スクラムに置いてスプリントレトロスペクティブは、今回のスプリントを回想して次のスプリントに活かすために必要な工程になります。
:::


## 3つの作成物

### プロダクトバックログ

- 実現したいものをリスト化したもの
- プロダクトにつき必ず一つだけ用意する
- 開発を通して常に最新に保つ必要がある

### スプリントバックログ

- プロダクトバックログの項目を実際の作業タスクに分割したもの
- ひとつのタスクは1日で終わる程度
- タスク数は増減可能

### インクリメント

- これまでのスプリントと今回のスプリントで完成したプロダクトバックログを合わせたもの
- 実装した部分は動作して検証可能である必要がある

:::message
#### インクリメントについて

プログラミングに慣れている方にとってはイメージしやすい用語かもしれません。

*increment* は **増加** という意味です。

スプリントが繰り返されていくにつれて、スプリントごとの完了済みのバックログアイテムは増加（インクリメント）していきます。
:::

# スクラムの背景

以上の紹介した構成要素のには3つの背景的な要素があります。

スクラムを実施していくにあたっては以下の3つの原則を意識して行っていくことが重要です。


## 透明性

スクラムでは現在の状況や問題点が常に明らかになっている必要があります。

状況の確認はデイリースクラムやスプリントレビューによって、問題点の把握は前述の2つに加えてスプリントレトロスペクティブによって行われます。

もちろん、これらのイベント以外においても、問題点などに関しては常にチーム間で共有しておく必要があります。


## 検査

プロダクトの進捗はプロダクトバックログやスプリントバックログ、インクリメントによって常に明らかになっている必要があります。

問題などがあれば必ずチームやステークホルダーに対して共有しなければなりません。


## 適応

明らかになったも問題点や改善点は効果の高いものから解決策を実施していきます。

具体的な施策はスプリントレトロスペクティブにおいて考案されます。


# まとめ

- スクラムは**スプリント**という単位で繰り返される開発プロセスである
- スクラムは**3つのロール**、**5つのイベント**、**3つの作成物**で構成される
- スクラムで重視されるのは**透明性**、**検査**、**適応**である

# あとがき

いざ学び始めると、スクラムがこんな単純な構成要素で成り立っていることに驚きました。

馴染みのない用語も少々ありますが、原則や実施方法が具体的になっていて内容はそれほど難しく感じません。

調べ物中に **スクラムマスター** という資格があることを知ったので、どこか改めて調べて記事にしたいですね。

今回は概要をさらっと紹介した程度ですが、次回位はもっと具体的な内容に踏み込んでみたいです。

---

### 参考文献

- [アジャイル][アジャイル]
- [スクラム（ソフトウェア開発）](https://ja.wikipedia.org/wiki/%E3%82%B9%E3%82%AF%E3%83%A9%E3%83%A0_(%E3%82%BD%E3%83%95%E3%83%88%E3%82%A6%E3%82%A7%E3%82%A2%E9%96%8B%E7%99%BA))
- [Jeff Sutherland][Jeff Sutherland]
- [Ken Schwaber][Ken Schwaber]
- [野中郁次郎][野中郁次郎]
- [竹内弘高][竹内弘高]
- [エクストリーム・プログラミング][エクストリーム・プログラミング]
- [カンバン][カンバン]
- [SCRUM BOOTCAMP THE BOOK](https://www.amazon.co.jp/SCRUM-BOOT-CAMP-BOOK%E3%80%90%E5%A2%97%E8%A3%9C%E6%94%B9%E8%A8%82%E7%89%88%E3%80%91-%E3%82%B9%E3%82%AF%E3%83%A9%E3%83%A0%E3%83%81%E3%83%BC%E3%83%A0%E3%81%A7%E3%81%AF%E3%81%98%E3%82%81%E3%82%8B%E3%82%A2%E3%82%B8%E3%83%A3%E3%82%A4%E3%83%AB%E9%96%8B%E7%99%BA/dp/4798163686)
- [スクラムの重要人物を詳細解説（スクラムマスターの資格・研修はどれがおすすめ？ 第1話）](https://qiita.com/sugulu_Ogawa_ISID/items/923c646f51df5fc8c99e)

[アジャイル]:https://ja.wikipedia.org/wiki/%E3%82%A2%E3%82%B8%E3%83%A3%E3%82%A4%E3%83%AB%E3%82%BD%E3%83%95%E3%83%88%E3%82%A6%E3%82%A7%E3%82%A2%E9%96%8B%E7%99%BA
[Jeff Sutherland]:https://en.wikipedia.org/wiki/Jeff_Sutherland
[Ken Schwaber]:https://en.wikipedia.org/wiki/Ken_Schwaber
[野中郁次郎]:https://ja.wikipedia.org/wiki/%E9%87%8E%E4%B8%AD%E9%83%81%E6%AC%A1%E9%83%8E
[竹内弘高]:https://ja.wikipedia.org/wiki/%E7%AB%B9%E5%86%85%E5%BC%98%E9%AB%98
[The New New Product Deelopment Game]:https://hbr.org/1986/01/the-new-new-product-development-game
[Scrum Guide]:https://scrumguides.org/
[エクストリーム・プログラミング]:https://ja.wikipedia.org/wiki/%E3%82%A8%E3%82%AF%E3%82%B9%E3%83%88%E3%83%AA%E3%83%BC%E3%83%A0%E3%83%BB%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0
[カンバン]:https://ja.wikipedia.org/wiki/%E3%81%8B%E3%82%93%E3%81%B0%E3%82%93_(%E3%82%BD%E3%83%95%E3%83%88%E3%82%A6%E3%82%A7%E3%82%A2%E9%96%8B%E7%99%BA)