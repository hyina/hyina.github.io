---
layout: post
title: "Homebrewでmysql5.6をインストールしてみる"
date: 2013-12-19 19:09
comments: true
categories: 開発
---
## Mysqlのインストールめも
---
MacたんにインストールしてあったMysqlの調子が悪かったので、１から入れなおすことにしました。
HomebrewでMysql5.6をインストールした時のメモです。

<!-- more -->

## 実施内容
下記を順番に実行していく。

### Homebrewを最新の状態に更新する。
```
$ brew update
```

### 最新のMysqlをインストールする。
```
$ brew install mysql
```
インストール後、メッセージが表示されるが、下記のコマンドでいつでも確認可能。
```
$ brew info mysql
```

### データベースのインストールとオプションを設定する。
```
$ unset TMPDIR
$ mysql_install_db --verbose --user=`whoami` --basedir="$(brew --prefix mysql)" --datadir=/usr/local/var/mysql --tmpdir=/tmp
```

### MySQLの設定を行うため、MySQLを起動する。
```
$ mysql.server start
```

### rootのパスワードを変更する。
```
mysqladmin -u root password 'new-password'
```

### MySQLのセキュリティ向上（念のため）
```
$ mysql_secure_installation
```
基本デフォルトで良いと思う。
*rootパスワードの設定
*rootユーザのログインをlocalhostに制限
*anonymousユーザの削除
*testデータベースを削除

### my.cnfを配置する。
my.cnfを配置後、カリカリと設定を書いていく。
```
$ touch /usr/local/etc/my.cnf
```
また、my.cnfの読み込み順は下記コマンドで確認する。
```
$ mysql --help | grep my.cnf
```

いじょうー