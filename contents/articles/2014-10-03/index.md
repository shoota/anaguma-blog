---
title: PostgreSQLインストールからカスタムユーザのDB作成まで
author: shoota
date: 2014-10-03
template: article.jade
---

CentOSにPostgreSQLをインストールしてから特定ユーザのDBを作成するまでのフロー。いつもわからなくなるのでをまとめておく。
結構長くなった。

<span class="more"></span>

### インストール

インストールはyumで行う。PostgreSQLは8系と9系で結構違うらしく、8系と9系でyumの探し方が違う。
8系は`yum install postgresql-server`、9系は`yum install postgresql9x-server`でインストールできるはず。
()xは任意のマイナーバージョン。9.3なら`yum install postgresql93-server`になる。)

postgresql-serverが依存関係がもっとも上位なので、serverを入れておけば必要なものが全部入る。
yumで9系が見つからない場合は[rpm](http://yum.postgresql.org/)をインストールする。

```bash
# 8.4の場合
$ yum install postgresql-server

## ----------リポジトリ検索のログ--------------------

 - ===================================//========================
 -  Package                           //        Version         
 - ===================================//========================
 - Installing:
 -  postgresql-server                 //        8.4.20-1.el6_5  
 - Installing for dependencies:
 -  postgresql                        //        8.4.20-1.el6_5  
 -  postgresql-libs                   //        8.4.20-1.el6_5  
 - 
 - Transaction Summary
 - ===================================//========================
 - Install       3 Package(s)

```

### インストールの確認、DB作成の前処理

#### インストールの確認

インストールが完了すると、OSにpostgresユーザが作成されているはずなので、idコマンドで確認する。
```bash
$ id postgres
uid=26(postgres) gid=26(postgres) groups=26(postgres)
```

postgresqlはインストール時では外部からのアクセスを許可しない。localhostに対しては`ident`で認証するようになっている。
この`ident`はOSのuser/passwordで認証する仕組みで、OSに作られpostgresユーザが、postgresql-serverへの最初のログインアカウントになる。
この設定は`pg_hba.conf`にある。

```bash
$ less /var/lib/pgsql/data/pg_hba.conf 

# TYPE  DATABASE    USER        CIDR-ADDRESS          METHO                                                                                                                                                                               
# "local" is for Unix domain socket connections only                                                                                                                               
local   all         all                               ident                                                                                                                              
# IPv4 local connections:
#host    all         all         127.0.0.1/32          ident
```

#### DB作成の前処理

DBにログインするためのユーザをOS側に作成し、パスワードを設定する。

```bash
$ usradd new_user
$ passwd new_user

```
作成したアカウントを後々、DBのアカウントにする。


---