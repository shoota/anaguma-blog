---
title: RubyよくわかんないけどOctopressでブログ構築してみる
author: shoota
date: 2013-08-31 18:50
template: article.jade
---

Rubyをいじったことがないので環境周りからよくわからないけど、Github Pagesにブログを構築してみるテスト。
LolipopさんのWordpressがものすごい苛められた記念。	

## いま、僕の手元にはWindows PCしかないので、Rubyが動く環境も当然ない。かといってCygwinをインストールするのも面倒なので、CentOSのサーバ上でやってみた。

### Rubyまわりを整備する

#### 1.rvmの確認
 rvmはインストールされていたけれどPATHが通っていなかったので、~/.bash_profileに追記
```bash
% vi ~/.bash_profile
PATH=PATH:/usr/local/rvm/bin/
% source ~/.bash_profile
% rvm -v
rvm 1.22.3 (stable) by Wayne E. Seguin <wayneeseguin@gmail.com>, Michal Papis <mpapis@gmail.com> [https://rvm.io/]
```

#### 2.ruby 1.9.3の用意 rvmでインストール。
```bash
% rvm install 1.9.3
% rvm list knwon
% rvm --default 1.9.3
% ruby -v
ruby 1.9.3p448 (2013-06-27 revision 41675) [x86_64-linux]
```

### [Octopress](http://octopress.org/ "Octopress")を使う

　いろいろ調べたけどRubyとJekyllとOctopressの関係がよくわからんのでおっかなびっくりしながら。
てかRubyのことまったく知らなかったので頭の中は「rake is 何」、「gem is 何」でいっぱいいっぱい。
いろんなブログを参考にさせていただいた。

+ [http://fnya.cocolog-nifty.com/blog/2012/04/centos-62-rvmru.html](http://fnya.cocolog-nifty.com/blog/2012/04/centos-62-rvmru.html)
+ [http://tokkono.cute.coocan.jp/blog/slow/index.php/programming/github-pages-almost-perfect-guide/](http://tokkono.cute.coocan.jp/blog/slow/index.php/programming/github-pages-almost-perfect-guide/)
+ [http://www.creativegear.jp/2012/12/29/octopress-post/](http://www.creativegear.jp/2012/12/29/octopress-post/)
+ [http://onigra.github.io/blog/2013/04/28/introduction-of-octopress/](http://onigra.github.io/blog/2013/04/28/introduction-of-octopress/)

　要はRuby1.9.3入れて、Octopressをgitから落として、rakeでビルドとデプロイすればいいってことでした。



#### Octopressをインストールする （+ カスタムテーマもいれておく）
 カスタムテーマは [こちら]("https://github.com/imathis/octopress/wiki") にある。
octopress/.themes/にgitからcloneしてきたら、`rake install['theme name']` で適用してあげればよい。

"user_name.github.com"のリポジトリは作っておく。つくったら何もしない（デプロイするとき競合しないようにするので、重要）

```bash
% cd /path/to/blog
% git clone git://github.com/imathis/octopress.git octopress
% cd octopress
% gem install bundler
% bundle install
% rake install
% rake setup_github_pages
## githubのリポジトリ名の入力が促される。sshのURLで入れること!!!

## greyjoyのテーマを入れる。
% git clone git://github.com/rezajatnika/greyjoy.git .themes/greyjoy
% rake install[greyjoy]

## 最低限の設定をする。
% vi _config.yml
# ----------------------- #
# Main Configs #
# ----------------------- #
url: http://shoota.github.io
title: Anaguma
subtitle: いろいろ試してみる.
author: shoota
simple_search: http://google.com/search
.....and more

## Jekyllでページをつくってgithubにpush
% rake generate
% rake deploy
```

`% rake setup_github_pages` で指定するのが `git://********/.git` じゃなきゃいけないせいで非常につまった。sshで接続するためのRSA keyも登録していなかったので、
`~/.ssh/` にRSAをつくってgithubのアカウントにセッティング。 ちゃんとやってるつもりなのにうまくいかなかったりして、ここが一番焦ったわ.... 
リポジトリ作るの失敗したのかと思ってREADME.mdをpushしたらrake deployでrejectされるし...... ともあれ、http://shoota.github.io/ に無事に作られた。

### 使ってみた感想
#### Macでやればよかった。
そもそもサーバ上でやることじゃなかった。書きながらlocalhost:4000で確認するのが前提。サーバの余計なポート空けたくなかったのでちょっとした変更とか、テーマ変えるたびにpushした。しんどい。

#### ブログ書き終わってから公開するまでめんどい*
単純なテストページをJekyllが吐き出してくれるのが3秒くらい。日々記事が増えたら結構時間かかりそう。当然、pushもします。その間僕はなにをしてたらいいのでしょうか。Github Pagesに反映されるのが1分くらい。すぐに記事が確認できないのは結構ストレスを感じたし、かといって確認せずには寝られないタチ。

#### 結論＝そこまで便利じゃなかったなぁ......
有償サービスですら攻撃受けて止まる昨今、無償のブログツールに多くを求めちゃいけない。技術的には非常に面白いので、やってみて非常に勉強になりました。Rubyもインストールしたしね。
---