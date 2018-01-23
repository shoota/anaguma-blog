---
title: iPhone写真をDropbox/CentOSで同期管理する §1
author: shoota
date: 2013-09-03 01:36
template: article.jade
---


iPhoneで撮った写真をDropbox on CentOSで同期管理する§1 DropboxとImagemagickのインストール、対象ファイルの抽出
写真に適用できるフィルタのパターンや、サービスの動作状況（たまに落ちてるサービスが多い）によってiPhoneで撮影する写真アプリがInstagramだったりPathだったりするが、写真を撮るたびに管理用のストレージサービスや写真系SNSにポストするのは非常に面倒。面倒になるとだんだんやらなくなってしまって結局散在箇所が増えるだけになってしまうので、意識せずに自分のサーバ上にアップできるようにしたい。面倒は敵。

<span class="more"></span>

でもそれ、（容量制限を無視すれば）iCloud/フォトストリームでことが足りるじゃん、なのだが、写真アプリがカメラロール上に保存する「原画像」と「フィルタを適用したシャレオツ画像」のうち、
「フィルタを適用したシャレオツ画像」のみを抽出してひとつのディレクトリにまとめる</span>**というのもやってしまいたい。
ということでLet’s Try Hack.


### iPhoneの下準備
iPhoneからDropboxに画像を保存するにはアプリを起動して「設定（歯車アイコン）> カメラアップロード」をオンにする。
バックグラウンドでのアップロード許可をすると電池がもたなそうなのでこれはオフしておく（電池切れで撮影できなくなっては自爆。また、パズドラのために電源確保は重要です）。
<div class="img">![Dropbox](/img/2013-09-03-01-01-52.png)</div>

### CentOSにDropbox CLIをインストールする
ほぼ[こちら](http://www.maruko2.com/mw/Dropbox%E3%82%92Linux%E3%81%A7%E5%88%A9%E7%94%A8%E3%81%99%E3%82%8B%E6%96%B9%E6%B3%95)の手順どおり。
Python2.6以上が必要ですが、僕のサーバにはインストールされておりました。
wgetでCLIツールを取得して、あとはお任せするだけ。簡単。PATHを通すために`$HOME/bin` （通常は.bash_profileにPATH指定されている）に配置している。

```bash
% mkdir ~/bin/
% python --version
Python 2.7.
% wget -O ~/bin/dropbox.py http://www.dropbox.com/download?dl=packages/dropbox.py
% chmod a+x ~/bin/dropbox.py
% dropbox.py
% dropbox.py start -i
```

初回でサービスを起動するとこのURLからクライアントのリンクを承認してね、みたいなのが出るので、アクセスする。

```bash
% dropbox.py start
To link this computer to a dropbox account, visit the following url:
https://www.dropbox.com/cli_link?host_id=youraccountkeyhogefoobar
```

認証が成功すればサービスが起動できる。`~/Dropbox/` にどんどんとデータが同期されているのを確認すればOK。
コマンドの使い方は[先のページ](http://www.maruko2.com/mw/Dropbox%E3%82%92Linux%E3%81%A7%E5%88%A9%E7%94%A8%E3%81%99%E3%82%8B%E6%96%B9%E6%B3%95)を参考にし、
同期対象が `~/Dropbox/カメラアップロード/` になるように同期除外設定を施した。

### ImageMagickのインストール
[ImageMagick](http://www.imagemagick.org/script/index.php "ImageMagick")は画像のデータをbashから直接扱うライブラリとしては大御所。
今回は画像の縦横のピクセルを取得するためにインストールする。後々サイズ変更スクリプトなんかにも使える。
PathやInstagramは正方形の画像を標準で生成するため、縦横のピクセル数が一致する画像のみをDropBoxディレクトリから管理ディレクトリにコピーさせることで、シャレオツ画像だけの塊を作ることが出来る。

インストールは[こちら](http://akkunchoi.github.io/imagemagick-rmagick-centos.html "こちら")に書かれている通り、ソースからmake / make installした。とても簡単。

```"bash
% yum install libjpeg-devel libpng-devel
% wget http://www.imagemagick.org/download/ImageMagick.tar.gz
% tar zxvf ImageMagick.tar.gz
% cd ImageMagickImageMagick-6.8.6-9
% ./configure
% make
% make install
% make clean
```

### 取得したファイルを分類してみる
そうこうしているうちにDropBoxの同期がだいぶ進んでいるのでbashでコピースクリプトを実行する。
DropBoxが生成するディレクトリ名は日本語になっているが、サーバの言語設定を日本語にしているので、一応扱える。
言語設定は `/etc/sysconfig/i18n` で確認できる。`LANG="C"` がデフォルトだが、これを `"ja_JP.UTF-8"`にするとコマンド結果などが日本語化される。

```bash
%cat /etc/sysconfig/i18n
#LANG="C";
LANG="ja_JP.UTF-8";
SYSFONT="latarcyrheb-sun16";
```

また、同期する写真ファイル名に半角スペースが入る。これが意外と厄介で、全ファイルのリストをfindで取得して変数に代入すると、
1行のfind結果が2つの値に分解されてしまう。これを解消するために、コマンドの区切り文字を改行のみにする環境変数設定をスクリプトのブロック内で定義してあげるとよい。
下のサンプルではfind結果1行ごとにファイルのwidth(px)、height(px)を取得して一致したらechoしている。
identifyがImageMagickによって提供されるコマンドで、フォーマット指定"%w"、"%h"によってそれぞれ出力させている。

<script src="https://gist.github.com/shoota/6418583.js"></script>

まずはここまで。実は"ファイル名に半角スペース問題"でかなりの時間を取られてしまった。解決法は以前に見たことのあるページだったので、昔の失敗が全然活かせていないorz。反省。

---