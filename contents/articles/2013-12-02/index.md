---
title: Node.jsでgithub pagesのブログ運用
author: shoota
date: 2013-12-02
template: article.jade
---

以前に、Github PagesにMarkdownで書くブログシステムを[Octorpressで構築してみた](/articles/2013-08-31/)が、諸事情あって結局Wordpressで運用していた。
今回、Node.js製のStatic Site Generator、[wintersmith](http://wintersmith.io/)をみつけ、その手軽さと軽快さからとても気に入ったので使うことにした。


このブログは[Node.js Adventar Calendar 2013](http://www.adventar.org/calendars/56)の12/2の記事です。

<span class="more"></span>

### wintersmithのインストールとひな形作成

Node.jsでGithub PagesにMarkdownで書くブログを構築したい。そんな僕の気持ちに応えてくれる[wintersmith](http://wintersmith.io/)に出会ったぁ（ウルルン調）。
[readme](https://github.com/jnordberg/wintersmith)に書かれている通り、npmでさくっとインストールします。
その後、`wintersmith new <name>`でサンプル記事とともに一式が出来上がるので`wintersmith preview` でlocalhost:8080でプレビューできます。

```bash
$ npm install -g wintersmith
$ wintersmith new anaguma-blog
$ cd anaguma-blog
$ wintersmith preview
```

`wintersmith new` はオプションをもっていて、`-T` でテンプレートを変更することもできます。

```bash
$ wintersmith  new --help
usage: wintersmith new [options] <path>

creates a skeleton site in <path>

options:

  -f, --force             overwrite existing files
  -T, --template <name>   template to create new site from (defaults to 'blog')

  available templates are: basic, blog, webapp
```

デフォルトは`-T blog` と同じなので今回はオプションを指定していません。ブラウザで確認するとこんな感じ。


<div class="lg-img">![preview](/img/preview.png)</div>


### ブログの設定

wintersmithにブログ全体の設定をします。`/config.json` を開いて、変更していきます。

```javascript
{
  "locals": {
    "url": "http://localhost:8080",
    "name": "Anaguma blog",
    "owner": "shoota",
    "description": "return new Bug();"
  },
  "plugins": [
    "./plugins/paginator.coffee"
  ],
  "require": {
    "moment": "moment",
    "_": "underscore",
    "typogr": "typogr"
  },
  "jade": {
    "pretty": false
  },
  "markdown": {
    "smartLists": true,
    "smartypants": true
  },
  "paginator": {
    "perPage": 3
  }
}

```

ブログコンテンツとして設定するのは基本的にlocalsのname、owner、descriptionです。nameはブログのトップなどに出てくるブログ名、ownerは著者、descriptionはブログ全体の説明です。
pluginsにはwintersmithのプラグインを設定でき、HTML出力の時に利用できます。
[sassのプラグイン](https://github.com/jnordberg/wintersmith-node-sass)や[stylusのプラグイン](https://github.com/jnordberg/wintersmith-less)などがあるようです。
paginator.coffeeはデフォルトで付随するプラグインで、ブログのページャを作成してくれます。
jade.prettyはjadeのコンパイルオプションで、trueにするとHTMLが圧縮されずに出力されます。デフォルトではtrueです。


### 記事を書いてHTMLに出力

`/contents/articles`の下に、記事ごとにディレクトリを作成し、index.mdを配置します。
markdown記法がどういう表示になるかは、ひな形を作成したときのサンプル記事を参考にするとよいと思います。気に入らない場合はjadeやCSSを変更します。
書き終えたら`wintersmith build`　でビルドを実行すると、`/build` の下にHTMLなどが出力されます。`wintersmith build --clean`のオプションでcontentsディレクトリ以下にないものが削除されるので、記事を消したり、ディレクトリ名を変更した場合に使えます。

ちなみに、previewは出力したHTMLではなく、サーバがリクエストごとにレスポンスを生成しているので、記事を変更してリロードすれば反映されます。便利ですね。


### githubリポジトリに登録

まずはgithubリモートリポジトリを作成し、masterブランチにpushしてあげます。

```bash
$ git init
$ git remote --add <repo_url>
## .gitignoreを設定したりする
$ git add .
$ git commit -m "myblog"
```


### github pagesに公開する

buildの中身をgh-pagesブランチにpushすれば晴れてブログ構築完了なのですが、masterブランチをpushしたら自動でgh-pagesにデプロイされるのがスマートですよね。
そこで僕の場合は[wercker](http://wercker.com/)というサービスを利用して、ビルドとデプロイを自動化しました。
werckerはgithubと連携していて、githubアカウントでログインするとリポジトリの参照などがスムーズになります。

#### githubトークン発行

デプロイにはwerckerからリモートリポジトリへpushを行うために、OAuthが必要なので、[github側でトークンの設定](https://github.com/settings/applications)を行います。
<div class="lg-img">![gittoken](/img/githubtoken.png)</div>

この値は、後でwerckerのデプロイタスクの環境変数として使用します。


### werckerをセットアップ

masterブランチのpushをトリガーとしてwerckerのタスクを起動させるには、アプリケーションを登録し、wercker.ymlをリポジトリにpushします。
werckerにログインして[ADD APPLICATION](https://app.wercker.com/#applications/create)を押すと5ステップで設定が完了します。
細かい説明は不要かと思いますが、ステップ3のコラボレータ設定ではwerckerbotをコラボレートとしてあげる必要があります。
<div class="lg-img">![gittoken](/img/wercker_app.png)</div>


#### アプリケーションのデプロイ設定
次はデプロイの登録を行います。werckerはビルドやデプロイの各ステップをDirectoryという単位で保存することができます。
wintersmithのコンテンツをgh-pagesへpushするステップは、すでに作成している方がいるので、[このステップ](https://app.wercker.com/#applications/51f71ee369cd738a32001822/tab)をそのまま使用します。


登録したアプリケーションのsettingsタブを開き、「Add deploy target」から「Custom deploy」を選択し、画像の通り設定します。
<div class="lg-img">![customdeploy](/img/wercker_build1.png)</div>

このデプロイステップでは、環境変数　`GIT_TOKEN` に先ほどのトークンを設定することで、gh-pagesへのpush権限を与えています。
デプロイターゲット内の「Add new variable」で環境変数を設定してあげます。このとき、Protectedにチェックを入れておくとログに値が出力されないので安全です。

<div class="lg-img">![git_token](/img/wercker_build2.png)</div>



### wercker.yml

最後に、wercker.ymlをpushして完成です。

```
box: wercker/nodejs
build:
  steps:
    - npm-install
    - script:
        name: wintersmith build
        code: ./node_modules/.bin/wintersmith build -v -o ./build
deploy:
  steps:
    - lukevivier/gh-pages:
        token: $GIT_TOKEN
        domain: blog.anaguma.org
        basedir: build
```

masterへのpushごとにcloneしてビルドします。cloneは自動で行われるので、その後npm-installステップでモジュールをインストールします。npm-installはグローバルインストールではないので、
wintersmithのCLIは階層を指定して実行します。`-o` オプションで出力ディレクトリ名を指定していて、これはdeployのbasedirと一致していなくてはいけません。

deployステップの`domain` は設定しておくと自動的にgh-pagesブランチにCNAMEファイルを作成してくれますので、サブドメインでブログを運用する場合は指定しておきます。


### まとめ

以上を盛り込んだこのブログのリポジトリが[こちら](https://github.com/shoota/anaguma-blog)です。
実際に動かしてみるとわかりますが、Node.jsっぽい軽快さであるのと、導入がすごく簡単なので、ぜひ一度ためして使ってみてほしいと思います。
wintersmithはもともとは[blacksmith](https://github.com/flatiron/blacksmith)にインスパイアされて作られたらしく、興味があるかたはこちらもチェックしてみてください。


wintersmithの[プラグインのひな形](https://github.com/jnordberg/wintersmith-plugin)も公開されているので、これをもとに何か作ってみる、というのもいいかもしれません。
Nodeで動くモジュールは当然使えるので、gravatarやtwitterと連携させたブログ生成も可能です。夢が広がりますね。


Node.jsのブログシステムには[ghost](https://ghost.org/)や[Calipso](http://calip.so/)なんかがありますが、導入コストでいうとやはりstatic site generatorに分があると思っています。
僕のように飽きっぽい人はいろいろ使ってみるのが結構楽しかったりするので、本格的にブログシステムを構築する前に遊び気分で触ってみてもらえたらいいな、と思います。


<p style="font-size:14px;">このブログは[Node.js Adventar Calendar 2013](http://www.adventar.org/calendars/56)の12/2の記事です。</p>


---