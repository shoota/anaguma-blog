---
title: Node.jsでスクレイピング (pplog駆動)
author: shoota
date: 2013-12-19
template: article.jade
---

ポエムっぽいのを書くだけのサービス、[pplog](http://www.pplog.net/)の絶妙な感じ、文章を練るのがすごく楽しくて見事にはまっています。
いろんな人のポエムを読むのも楽しいし、自分が書いた文章のくせに何度も読み返すのも楽しい。
是非Web API化していただきたい|дﾟ)チラリ。
でもAPI化するかどうかなんてわからないし、するとしても待てないので、とりあえず自分のポエムをスクレイピングしてみた(pplog-driven development)。Node.jsが楽チン。

<span class="more"></span>

###　必要な npm モジュール

リクエスト飛ばすのと、返ってきたHTMLをパースするのができればAPI化できるので、よさげなやつをインストール。HTTPリクエスト用に`request`、jQueryっぽいHTMLparserとして`cheerio`を採用。

``` bash
express myPplog-API
cd myPplog-API
npm install
npm install request cheerio --save
```

### スクレイピング実装

index.jsにrequest使ってHTMLを取ってきて、cheerioでパースしてresponceするモジュールを書く。
<script src="https://gist.github.com/shoota/8040292.js"></script>


取得したHTML全体から`.content-body`の中のtextを取ってきて完了。
今考えると、.text()じゃなくて.html()の方がよかった。
一応API化できたけど、勝手にスクレイピングするのグレーな気がするのでデプロイはしてない。



本家サイトのデザインと隠し要素のバランスが楽しいみたいなとこあるのでAPI化しても面白みがない。ということで、みんなもやろうぜー。[<i class="fa fa-external-link"></i>](http://www.pplog.net/)


---