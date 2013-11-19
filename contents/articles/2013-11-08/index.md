---
title: Node-express / EJS ViewでJSON出力
author: shoota
date: 2013-11-08 17:55
template: article.jade
---

JSONを返すAPIをサーバからたたいて結果をまるっとEJSに埋め込みたかった。地味にはまったのでメモ。

<span class="more"></span>

## router
```javascript
app.get('/hoge', function(req, res){
  //データとってくるとかAPI叩くとかしてJSONができる
  var json= foo.getJSON();
  res.render('hoge.ejs', {data:json});
});
```

## ejs
```html
&lt;!-- HTML escapeされてしまってだめだった --&gt;
 &lt;%= JSON.stringify(data) %&gt;

&lt;!-- %=じゃなくて%-を使うといい --&gt;
 &lt;%- JSON.stringify(data) %&gt;
```

EJSに渡しているjsonはJSONオブジェクトなので、EJS側で `stringify` して文字列にする。（別にRouter側でやってもいいが）
<%=だと中の文字列に対してオートでHTMLエスケープされちゃうので、&quoat;がいっぱいでてくる。 <%-で書いてあげればOK。</span>

---
