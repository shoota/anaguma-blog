---
title: GithubのEmojisをFontAwesomeっぽい感じで使う
author: shoota
date: 2013-12-16
template: article.jade
---

Emojisってのはcommit commentとか、issueとか、プルリとかでたまに見る、あの絵文字。
Github API v3 をびゃーっっと眺めていたらリストを取得できるのをみつけたので、FontAwesomeっぽい記法でアイコン（というか画像）を埋め込んでくれるjQueryメソッドを書いてみた。

<span class="more"></span>

### 仕様的な何か（つくりかたとか制約とか）

+ [GithubのAPI](https://api.github.com/emojis)でEmojisの名前とpngのURLをJSON形式で取得できる。認証やAPIキーは不要。
+ FontAwesomeっぽく、アイコンの種類はクラスで指定する。命名規則は、markdownで書くときの名前の頭に"gh-"をつける。
+ アイコンであることを明示するクラスとして'ge'をつける。FontAwesomeでの'fa'に相当するもの。
+ Emojisのpng画像のサイズは親DOMの`font-size`をpx単位で取得し、それを高さとして設定する。

### 実装してみる

実装は超簡単で、JSONとってきて'ge'のクラスがついている`<i>`タグのクラス名から画像URLを引いて、その画像を埋め込むだけ。
`<i>`タグに親DOMのフォントサイズを高さとして指定して、`img`タグの高さは`height: inherit`を指定する。

``` javascript
  $(function(){
    $.getJSON(
      "https://api.github.com/emojis",
      function(emojis){
        $('i.ge').each(function(){
          var fsz=$(this).parent().css('font-size');
          var cls = $(this).attr('class').replace(/gh-|ge| /g,'');
          $(this).append('<img src="'+emojis[cls]+'" style="height:inherit">');
          $(this).height(fsz);
        });
    });
  });

```

### 表示してみる

<div class="lg-img">![gists_capture](/img/emojiawesome_js.png)</div>

そこそこ遜色ない感じで表示できてるっぽい。（ブラウザはChrome v.31.x）


###　HTML全体はこんな感じ

<script src="https://gist.github.com/shoota/7987115.js"></script>
サンプルではテキスト高さを揃えるために`.ge {　vertical-align: middle;　}`してるけど、好みとか周りの見た目とか、一緒に使うフォントとかで違ってきそう。


---