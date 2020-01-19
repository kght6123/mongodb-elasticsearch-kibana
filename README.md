# MongoDB and Elasticsearch on Docker

MongoDBに全文検索エンジンを導入しようと思い構築したDocker環境です。

後日、イメージサイズを抑える工夫や、k8sに対応したりする予定です。

（MongoDBのクラスタは認証無しです。）

## Environment

ホストOSは、Linuxを想定しています。

Docker（docker-compose）のインストール済みが前提条件です。

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
