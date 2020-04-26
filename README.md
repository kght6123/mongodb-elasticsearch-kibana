# MongoDB and Elasticsearch on Docker

MongoDBに全文検索エンジンを導入しようと思い構築したDocker環境です。

後日、イメージサイズを抑える工夫や、k8sに対応したりする予定です。

（MongoDBのクラスタは認証無しです。）

## Environment

ホストOSは、Linuxを想定しています。

Docker（docker-compose）のインストール済みが前提条件です。

## Settings

```sh
# クラスタの設定をする
$ sudo docker exec -it mongodb1 mongo
> config = {"_id" : "mongodb-repl-set","members" : [{"_id" : 0,"host" : "mongodb1:27017","priority" : 1},{"_id" : 1,"host" : "mongodb2:27017","priority" : 1},{"_id" : 2,"host" : "mongodb3:27017","priority" : 2}]}
> rs.initiate(config)
# クラスタのプライマリの設定を変えるとき
> config = rs.config()
> config.members[0].priority = 1
> config.members[1].priority = 1
> config.members[2].priority = 2 # mongodb3をプライマリにする
> rs.reconfig(config)
```

## MongoDBに接続する

### 認証なし

PRIMARYにつなぐ

#### MongoDB Compassでつなぐ

```
mongodb://127.0.0.1:37017/admin
```

#### mongoコマンドでつなぐ

```sh
sudo docker exec -it mongodb3 mongo
> db.mycollection.insert({name : 'sample'})
```

#### MongoExpressでつなぐ

http://localhost:8081/

## Kibana(ElasticSearch)に接続する

http://127.0.0.1:5601/

## Author

* [**@kght6123**](https://twitter.com/kght6123)

## Contacts

公開内容の詳細に関しては[**@kght6123**](https://twitter.com/kght6123)まで、お気軽にお問い合わせ下さい。

For more details on the public content please feel free to contact us at [**@kght6123**](https://twitter.com/kght6123).

## Copyright
**```Copyright (c) 2020 Hirotaka Koga```**
