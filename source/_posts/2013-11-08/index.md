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
<!-- HTML escapeされてしまってだめだった -->
 <%= JSON.stringify(data) %>

<!-- %=じゃなくて%-を使うといい -->
 <%- JSON.stringify(data) %>
```

EJSに渡しているjsonはJSONオブジェクトなので、EJS側で `stringify` して文字列にする。（別にRouter側でやってもいいが）
`<%= str %>` だと中の文字列に対してオートでHTMLエスケープされちゃうので、`&quoat;`がいっぱいでてくる。 `<%- str %>` で書いてあげればOK。</span>

---
