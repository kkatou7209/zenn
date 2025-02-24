---
title: "PDFとは"
---

# PDFが作られた経緯

## PDFの定義

仕様書の７ページ（ *p.ⅶ* ）、導入の部分で以下のように書かれています。

> *The goal of PDF is to enable users to exchange and view electronic documents easily and reliably, independent of the environment in which they were created or the environment in which they are viewed or printed.*

> *PDFの目標は、それがどの環境で作成され、閲覧され、あるいは印刷されようと、容易かつ信頼性のある電子文書の交換と閲覧をユーザーに提供することです。（拙訳）*

PDFの動機はこの文書に半分以上含まれていると言っていいでしょう。この方針はPDFという名前にも表れています。

PDFは英語で ***Portable Document Format***（持ち運び可能な文書） です。

確かに物理的にも電子的にもどこにでも持っていけますね。

今となってはほとんど全てのOS、ブラウザ、モバイル端末、プリンターで閲覧・印刷が可能です。

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

ばーっと書いていますが、要するに「どこでも、何にも使えるようにししよう！」ってことですよね。

あとは作る人の権利や誰が発行したのかを保証するためのセキュリティやデジタル署名なんかにも言及しています。社会人の人はPDFにパスワードかけて相手方に送る、みたいなことやったことある人多いんじゃないでしょうか。（あれ絶対やめた方がいいですよね...）

## PDFの歴史

PDFは1993年に Adobe（当時は Adobe Systems）によって文書共有を目的に開発されました。（PDF1.0 と Acrobat1.0 のリリース）

当時は Adobe Acrobat というアプリケーションでしか閲覧できませんでした。しかもダウンロードと表示が遅く、HTMLのようなハイパーリンクがなかったため、インターネットの強みを活かしきれず、あまり普及しませんでした。

しかし 1994年にAcrobat Reader（現 Adobe Reader）を無償配布するようになると、急速に普及し始め、電子文書のデファクトスタンダードの地位を確立します。

日本では2004年にソースネクストが配布を始めた「いきなりPDF」シリーズが注目を集めましたが、Acrobat はすでに1996年から日本語対応しています。

2008年には国際標準化機構（ISO）によって **ISO 3200:2008** として策定され、以降は専門家ボランティアのもとで開発が続けられています。

本記事で参照しているのもこの **ISO 3200:2008** です。実際の文書（もちろんPDF）へのリンクは末尾の参考文献に乗っけているので、興味のある方は読んでみてください。

# 後方互換性
> *p.10 **6 Version Designation***

PDFは高い後方互換性を保ちながら開発・策定されています。

あるバージョンで採用された仕様（例えばPDF1.3など）はそれ以降のバージョンでも有効性があります。

