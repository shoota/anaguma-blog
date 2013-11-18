---
title: iPhone写真をDropbox/CentOSで同期管理する §2
author: shoota
date: 2013-09-04 17:57
template: article.jade
---

iPhoneで撮った写真をDropbox on CentOSで同期管理する §2 Dropboxのファイル名を置換しながらコピーする

## [前回](/articles/2013-09-03) に引き続き、iPhone→Dropbox→サーバ→欲しい写真だけを集めたディレクトリ へと半自動的に同期管理させる方法を考える。前回はDropbox CLIとImageMagickのインストールを行い、サーバ上に全画像を同期させ、ピクセル数から判定したファイルを抽出（echoで確認）した。
今回はDropboxから取得したファイルを管理用ディレクトリにコピーする方法を考える。ということでLet’s Try Hack.

ところでこれを見てくれ。こいつをどう思う[？](http://ja.wikipedia.org/wiki/%E3%81%8F%E3%81%9D%E3%81%BF%E3%81%9D%E3%83%86%E3%82%AF%E3%83%8B%E3%83%83%E3%82%AF "？")

```bash
% ls ~/Dropbox/カメラアップロード
2012-02-22 12.46.00.jpg 2012-06-30 09.24.48.jpg 2012-08-15 08.00.58.jpg........................(ry
```

凄く・・・半角スペースです・・・。
前回でも書いたとおり、半角スペースがファイル名に入っている。このままコピーをしてしまうとhtmlから参照できなかったり、これからの運用のためにいろいろと面倒であること請け合い。
なのでコピー時にリネームをする必要がある。
また、欲を言えばピリオドも置換したいところではあるが、半角スペースをアンダースコアに変更するつもりなので、アンダースコア以外の文字にしたい。
なぜなら、逆方向の変換、デコードを行う際に位置によって半角スペースに戻したりピリオドに戻さなければならないからだ。
しかしながら、ハイフンはすでにオリジナル画像名に使われているので、今回はハイフンでもアンダースコアでもなく、イコールに置換することとした。


### 文字列置換の方法を考察

shellスクリプトで利用できる文字列変換といえば、[sed](http://ja.wikipedia.org/wiki/Sed_(%E3%82%B3%E3%83%B3%E3%83%94%E3%83%A5%E3%83%BC%E3%82%BF) "sed")が有名だが、
shellスクリプト変数のパラメータ展開でも実行可能である。sedコマンドは正規表現を利用でき、「echo $変数名 | sed -e "s/regexp/substitute_string/"」の書式で文字列置換が出来る。
パラメータ展開は比較的知名度が低いようだが、単純な置換においては記述が簡易で高速（らしい）。
パラメータ展開については[こちらのブログ記事](http://dharry.hatenablog.com/entry/20090211/1234290856 "こちらのブログ記事")が非常にわかりやすかったので、参照してください。
ではどちらが今回の方法に適しているのか、実際にDropboxから対象ファイルを抽出し、そのファイル名文字列を置換した後、そのファイルがコピー先に無い場合にechoする（後々ここをcpにする）スクリプトを、
sed版とパラメータ展開版で書き、実行してみる。

#### まずはsedの場合
```bash
#!/bin/bash
files=`find ~/Dropbox/カメラアップロード -type f | grep \.jpg`
(IFS=$'\n';
  for i in ${files}; do
    width=`identify -format &quot;%w&quot; ${i}`
    height=`identify -format &quot;%h&quot; ${i}`
      if [ ${width} = ${height} ]; then
        fname=${i##*/}
        dist_fname=`echo $fname | sed -e &quot;s/\ /_/&quot; | sed -e &quot;s/\.[^jpg]/=/g&quot;`
        distname=&quot;~/dist/&quot;$dist_fname

        #isExists file?
        if [ ! -e ${distname} ]; then
          echo &quot;source: &quot;${i}&quot; dist: &quot;${distname}
        fi
      fi
  done
)
echo "done ps.";
```
置換を行っているのは9行目。半角スペースをアンダースコアに、.jpg以外のピリオドをイコールに変換している。

#### 次に、パラメータ展開の場合
```bash
#!/bin/bash
files=`find ~/Dropbox/カメラアップロード -type f | grep \.jpg`
(IFS=$'\n';
  for i in ${files}; do
    width=`identify -format &quot;%w&quot; ${i}`
    height=`identify -format &quot;%h&quot; ${i}`
      if [ ${width} = ${height} ]; then
        fname=${i##*/}
        dist_fname=${fname/\ /_}
        dist_fname=${dist_fname//\./=}
        dist_fname=${dist_fname/-jpg/\.jpg}
        distname=&quot;~/dist/&quot;$dist_fname
        #isExists file?
        if [ ! -e ${distname} ]; then
          echo &quot;source: &quot;${i}&quot; dist: &quot;${distname}
        fi
      fi
  done
)
echo "done ps."
```

こちらは正規表現が使えないので、スペース置換、ピリオド全置換、suffixのピリオドの戻し、の３段置換になる。

どちらも期待する置換結果が得られることを確認したうえで、実行時間を計ってみる。スクリプトの開始と終了にdateで時間を秒精度で出力させ、なんどか同じ処理を実行させる。
__ sed：21～22秒（22秒がほとんど）__
__ パラメータ展開：20～21秒（半々くらい__

微妙な差だ・・・・・。
Dropboxへのアップロードスパンを考慮するとファイルの同期処理はcronで日次実行が妥当だと思われる。
すると一日に撮った写真の枚数分置換処理が動く計算になるので、運用上は多くても数十回程度の文字列置換処理に収まるはず。
今回の時間測定では、文字列置換をしたファイル数は450件弱だったので、それで1秒強程度の差しかでないのであれば、実行時間が早い/遅いという議論があまり意味を持たない。
むしろ1行で記述できているsedのほうがスマートなスクリプトのように思えるため、今回のリネームの処理は、sedを用いた文字列置換によって実装することにした。

今回はsed、パラメータ展開、あとさらっと書いてるけど `if [ ! -e $file_name]` 構文による存在チェックの利用方法が学べた。

次回は逆方向の処理、すなわちDropbox上から削除されたファイルの削除処理に関連した処理と、ログ出力を考える。
---
