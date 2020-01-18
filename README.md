# LAMP VScode debug (and Wordpress) from Docker

PHPと、Wordpressの学習の為に構築したDocker環境です。

ホストOSから、VSCodeでリモートデバッグが可能です。

## Environment

ホストOSは、Macを想定しています。

Docker（docker-compose）と、VSCodeのインストール済みが前提条件です。

## Installation

1. Clone

2. `apache/xdebug.ini`の`xdebug.remote_host`を、Macのコンピュータ名に置き換え。

3. `build.sh`と`up.sh`を実行してください。

4. 下記のURLでアクセスできます。

    * PHP http://localhost:8080/sample/index.php
    * PHPMyAdmin http://localhost:18080
    * WordPress http://localhost:28080
    * WordPress 管理コンソール http://localhost:28080/wp-admin/

## Detail

* WordPress

    kght6123/kght6123で、ログインできます。

* docker-composeコマンド実行

    `down.sh`でコンテナの破棄、`stop.sh`でコンテナの停止です。

    `ps.sh`でコンテナの稼働状況を確認できます。

    含まれているShellファイルの殆どは、docker-composeコマンドのショートカットです。

* VSCode

    VSCodeで`html/html.code-workspace`を開くと、PHPのデバッグ環境になります。

* MySQL

    WordpressとPHPのデバック環境のMySQLのDatabaseは共存しています、mysqlのコンテナを別名で追加することで分割しても良いかも？

    `sql/mysql-wordpress-dump.sql`はWordpressのDBのダンプ情報です。`wpdump.sh`を実行して作成しています。


* 初期化方法

    `clean.sh`を実行すると、MySQLのDatabaseと、Wordpressのテーマとプラグインフォルダ、phpmyadminのセッションをクリアします。

## Author
* [**@kght6123**](https://twitter.com/kght6123)

## Contacts

公開内容の詳細に関しては[**@kght6123**](https://twitter.com/kght6123)まで、お気軽にお問い合わせ下さい。

For more details on the public content please feel free to contact us at [**@kght6123**](https://twitter.com/kght6123).

## Copyright
**```Copyright (c) 2018 Hirotaka Koga```**
