---
title: "PDF仕様書読解 Part 1 : PDFとは"
emoji: "📄"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["pdf"]
published: true
---

# この記事について

## PDF仕様書読解シリーズ

閲覧ありがとうございます！

このシリーズではPDFの公式仕様書の読解を通してPDFへの理解を深めていきます。

参考にするのは[**PDF 32000-1:2008**](https://opensource.adobe.com/dc-acrobat-sdk-docs/pdfstandards/PDF32000_2008.pdf)です。

全文英語の756ページの仕様書なので、シリーズの更新頻度についてはあまり期待しないでください...。

## 動機

PDFについて詳しく知りたいと思ったのは、PDFのJavaScriptライブラリを探しても納得のいくものが見つからず、それなら自分で作ってみたいと思ったからです。

だからもちろん、このシリーズのゴールは実際にPDFのライブラリを作ることです。完成はいつになるかわかりませんが、納得のいくものが出来上がったらライブラリとして公開する予定です。

## 方針

基本的に仕様書の流れに沿って進んでいきます。

修正や追記によって不定期で更新されることがあります。あらかじめご容赦ください。

本文から取り除く際は~~取り消し線~~で対応します。

## ご指摘について

情報の誤り、誤字・脱字等があればコメントにてお知らせください。

情報の訂正に関するご指摘であれば、情報源を明示していただ開けるととても助かります。

<br>
<br>
<br>

# 前書き

PDF仕様書読解シリーズの記念すべき第一弾のテーマは「PDFとは」です。

PDFの詳細に入っていく前にまずはPDFというものがなんなのか知っていこうと思います。

# PDFの定義

仕様書の７ページ（ *p.ⅶ* ）、導入の部分で以下のように書かれています。

> *The goal of PDF is to enable users to exchange and view electronic documents easily and reliably, independent of the environment in which they were created or the environment in which they are viewed or printed.*

> *PDFの目標は、それがどの環境で作成され、閲覧され、あるいは印刷されようと、容易かつ信頼性のある電子文書の交換と閲覧をユーザーに提供することです。（拙訳）*

PDFの動機はこの文書に半分以上含まれていると言っていいでしょう。この方針はPDFという名前にも表れています。

PDFは英語で ***Portable Document Format***（持ち運び可能な文書） です。

確かに物理的にも電子的にもどこにでも持っていけますね。

今となってはほとんど全てのOS、ブラウザ、モバイル端末、プリンターで閲覧・印刷が可能です。

<br>

さらにPDFとしての要件を以下のようにまとめています。

> - *preservation of document fidelity independent of the device, platform, and software*
>   *デバイス、プラットフォーム、ソフトウェアに依存せしない文書の正確性の確保*
>   
> - *merging of device content from diverse sources - Web sites, word proccessing and spread sheet programs, scaned documents, photos, and graphics - into one self-contained document while maintaining the integrity of all original source documents*
>   *デバイス上の様々な資料（Webサイト、文書作成またはスプレッドシートプログラム、スキャンされた文書、写真、グラフィック）を単一のそれ自体で独立した文書にまとめつつ、元の資料との互換性を維持する*
>
> - *collaborative editing of documents from multiple locations or platforms*
>   *複数の場所やプラットフォーム上での共同編集*
>
> - *digital signatures to certify authority*
>   *権威を保証するためのデジタル署名*
>
> - *security and permissions to allow the creator to retain control of the document and associated rights*
>   *作成者が文書の編集権限を保持するためのセキュリティと認可*
>
> - *accessibility of content to those with disabilities*
>   *ハンディキャップを持つ人たちへのアクセシビリティ*
>
> - *extracion and reuse of content for use with other file formats and applications*
>   *他のファイル形式やアプリケーションで利用するための抽出可能性と再利用性*
>
> - *electronic forms o gather data and integrate it with business systems*
>   *データ収集とビジネスシステムとの統合のための電子形式*

ばーっと書いていますが、要するに「どこでも、何にも使えるようにしよう！」ってことですよね。

あとは作る人の権利や誰が発行したのかを保証するためのセキュリティやデジタル署名なんかにも言及しています。社会人の人はPDFにパスワードかけて相手方に送る、みたいなことやったことある人多いんじゃないでしょうか。（あれ絶対やめた方がいいですよね...）

## PDFの歴史

PDFは1993年に Adobe（当時は Adobe Systems）によって文書共有を目的に開発されました。（PDF1.0 と Acrobat1.0 のリリース）

**PostScript**という言語をもとに開発されたそうです。

当時は Adobe Acrobat というアプリケーションでしか閲覧できませんでした。しかもダウンロードと表示が遅く、HTMLのようなハイパーリンクがなかったため、インターネットの強みを活かしきれず、あまり普及しませんでした。

しかし 1994年にAcrobat Reader（現 Adobe Reader）を無償配布するようになると急速に普及し始め、電子文書のデファクトスタンダードの地位を確立します。

日本では2004年にソースネクストが配布を始めた「いきなりPDF」シリーズが注目を集めましたが、Acrobat はすでに1996年から日本語対応しています。

2008年には国際標準化機構（ISO）によって **ISO 3200:2008** として策定され、以降は専門家ボランティアのもとで開発が続けられています。

# 後方互換性

PDFは高い後方互換性を保ちながら開発・策定されています。

あるバージョンで採用された仕様（例えばPDF1.3など）はそれ以降のバージョンでも有効性があります。

PDFのどのバージョン（PDF1.0 ~ 7）も有効なPDFです。

# 仕様への準拠

PDFファイルそのものはもちろん、PDF閲覧・編集ソフトは PDF 32000-1:2008 の仕様にすべて準拠している必要があります。

ただし、具体的な実装方法やユーザーインターフェースについての強制はありません。

# まとめ

PDF（Portable Document Format）はその名の通り、どんな環境でも利用でき、簡単に運ぶことができるファイルフォーマットです。

その可搬性の高さから、インターネットの普及とともに急速に発展しました。

そして私たちがPDFをどこでも閲覧できるのは、すべての主要なソフトがこの仕様書に準拠しているからです。

# あとがき

今回はPDFの定義や歴史について学びました。

PDFがあのAdobeによって開発されたということを知らなかった人もいるのではないでしょうか。さすがは天下のAdobeですね。

次回はPDFの基礎文法について取り扱います。

# 参考文献

- [PDF 32000-1:2008](https://opensource.adobe.com/dc-acrobat-sdk-docs/pdfstandards/PDF32000_2008.pdf)