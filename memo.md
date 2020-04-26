
## Docker install

```sh
$ sudo apt-get install docker-compose
$ docker -v
Docker version 19.03.2, build 6a30dfca03
$ docker-compose -v
docker-compose version 1.21.0, build unknown
$ sudo service docker start

# 先にMongoDBクラスタをつくって確認する
$ sudo docker-compose up -d

$ sudo docker-compose exec mongodb1 bash
$ mongo admin -u root -p

$ sudo docker-compose down

# ユーザとパスワードを変更したときは下記で消す
$ sudo rm -Rf mongodb1

# クラスタの設定をする
$ sudo docker exec -it mongodb1 mongo #admin -u root -p
#> db = (new Mongo('localhost:27017')).getDB('admin')
> config = {"_id" : "mongodb-repl-set","members" : [{"_id" : 0,"host" : "mongodb1:27017","priority" : 2},{"_id" : 1,"host" : "mongodb2:27017","priority" : 1},{"_id" : 2,"host" : "mongodb3:27017","priority" : 1}]}
> #use admin
> #db.auth("root","example") # Error: Authentication failed.
> rs.initiate(config) # "errmsg" : "command replSetInitiate requires authentication",
# 後からクラスタの設定を変える
> config = rs.config()
> config.members[0].priority = 1
> config.members[1].priority = 1
> config.members[2].priority = 2 # mongodb3をプライマリにする
> rs.reconfig(config)
> rs.status()
# 余裕があったらこれ試す(認証キーがいるっぽい、認証付きのときは。今は認証なし)　https://qiita.com/usabarashi/items/3854a1da0e47feb93ba0
mongodb-repl-set:SECONDARY> db.mycollection.insert({name : 'sample'})
WriteResult({ "nInserted" : 1 })
mongodb-repl-set:PRIMARY> db.mycollection.find()
{ "_id" : ObjectId("5e207b6ee7e73c5cf69bdaed"), "name" : "sample" }
mongodb-repl-set:PRIMARY> db2 = (new Mongo('mongodb2:27017')).getDB('admin')
admin
mongodb-repl-set:PRIMARY> db2.setSlaveOk()
mongodb-repl-set:PRIMARY> db2.mycollection.find()

# 次にElasticSearch、Kibana、MongoConnectorのコンテナを作る
$ sudo docker-compose build
$ sudo docker-compose up -d
$ sudo docker ps
$ sudo docker exec -it mongodb-elasticsearch-kibana_mongo-connector_1 bash

# なぜかrootでフォルダが作られてた
$ sudo chown -R kght6123 esdata1
$ sudo chown -R kght6123 mongodb1
$ sudo chown -R kght6123 mongodb2
$ sudo chown -R kght6123 mongodb3

$ sudo docker exec -it mongo-connector sh # alpineだからbashない
$ ping elasticsearch # OK
$ ping mongodb1 # OK
$ exit
```

## MongoDB Compassで接続する

認証あり（未検証）
mongodb://root:example@localhost:27017/admin

認証なし
mongodb://127.0.0.1:17017/admin

## inotifyの上限値を超えてしまう

https://www.virment.com/how-to-fix-system-limit-for-number-of-file-watchers-reached/

```
(node:30428) UnhandledPromiseRejectionWarning: Error: ENOSPC: System limit for number of file watchers reached, watch '/home/kght6123/develop/mirai-share/node_modules/@nuxt/vue-app/template/views/loading'
```

```sh
kght6123@kght6123-MicroPC:~$ sudo sysctl fs.inotify.max_user_watches=524288
[sudo] kght6123 のパスワード: 
fs.inotify.max_user_watches = 524288
kght6123@kght6123-MicroPC:~$ cat /proc/sys/fs/inotify/max_user_watches
524288
```

## Collection作成

```sh
$ sudo docker exec -it mongodb3 mongo # PRIMARYに接続
> use admin
> db.createCollection('questions');
{ "ok" : 1 }
> show collections
questions
```

## Kuromojiの全文検索設定

一度、MongoDBの既存のコレクションのすべてのドキュメントを削除して、ElasticSearchのインデックスを作り直す

https://www.elastic.co/guide/en/elasticsearch/plugins/current/analysis-kuromoji-analyzer.html

https://www.elastic.co/guide/en/elasticsearch/reference/current/removal-of-types.html

```sh
sudo apt install curl
curl -H "Content-Type: application/json" -X DELETE 'localhost:9200/admin'
curl -H "Content-Type: application/json" -X PUT 'localhost:9200/admin' -d '
{
  "settings": {
    "analysis": {
      "tokenizer": {
        "kuromoji_user_dict": {
          "type": "kuromoji_tokenizer",
          "mode": "search",
          "user_dictionary_rules": []
        }
      },
      "analyzer": {
        "my_kuromoji_analyzer": {
          "type": "custom",
          "tokenizer": "kuromoji_user_dict",
          "filter": [
            "kuromoji_baseform",
            "kuromoji_number",
            "katakana_readingform",
            "katakana_stemmer",
            "ja_stop"
          ]
        }
      },
      "filter" : {
        "katakana_readingform" : {
          "type" : "kuromoji_readingform",
          "use_romaji" : false
        },
        "katakana_stemmer": {
          "type": "kuromoji_stemmer",
          "minimum_length": 4
        },
        "ja_stop": {
          "type": "ja_stop",
          "stopwords": []
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "my_kuromoji_analyzer"
      },
      "body": {
        "type": "text",
        "analyzer": "my_kuromoji_analyzer"
      },
      "tags": {
        "type": "text",
        "analyzer": "my_kuromoji_analyzer"
      }
    }
  }
}'
```

## Kuromijiのインストール済みを確認してアナライズを実行

https://medium.com/hello-elasticsearch/elasticsearch-833a0704e44b

```sh
$ curl -X GET localhost:9200/_nodes/plugins?pretty
{
  "_nodes" : {
    "total" : 1,
    "successful" : 1,
    "failed" : 0
  },
  "cluster_name" : "mongodb-repl-set",
  "nodes" : {
    "7pHWKPo4RXm8FA8Xgf8lbw" : {
      "name" : "c07bf8773083",
      "transport_address" : "172.27.0.2:9300",
      "host" : "172.27.0.2",
      "ip" : "172.27.0.2",
      "version" : "7.5.1",
      "build_flavor" : "default",
      "build_type" : "docker",
      "build_hash" : "3ae9ac9a93c95bd0cdc054951cf95d88e1e18d96",
      "roles" : [
        "ingest",
        "master",
        "data",
        "ml"
      ],
      "attributes" : {
        "ml.machine_memory" : "8164110336",
        "xpack.installed" : "true",
        "ml.max_open_jobs" : "20"
      },
      "plugins" : [
        {
          "name" : "analysis-kuromoji",
          "version" : "7.5.1",
          "elasticsearch_version" : "7.5.1",
          "java_version" : "1.8",
          "description" : "The Japanese (kuromoji) Analysis plugin integrates Lucene kuromoji analysis module into elasticsearch.",
          "classname" : "org.elasticsearch.plugin.analysis.kuromoji.AnalysisKuromojiPlugin",
          "extended_plugins" : [ ],
          "has_native_controller" : false
        }
      ],
```


```sh
curl -H "Content-Type: application/json" -X GET 'localhost:9200/_analyze' -d '
{
  "analyzer" : "standard",
  "text" : "this is a test"
}
'
```

```sh
curl -H "Content-Type: application/json" -X GET 'localhost:9200/_analyze' -d '
{
  "analyzer" : "im_default",
  "text" : "ところゞゝゝ、ジヾが、時々、馬鹿々々しい"
}
'
```

### open, closeしてインデックスの設定変更

```sh
curl -H "Content-Type: application/json" -X POST 'localhost:9200/admin/_close'
curl -H "Content-Type: application/json" -X PUT 'localhost:9200/admin/_settings' -d '
{
  "settings": {
    "analysis": {
      "tokenizer": {
        "kuromoji_user_dict": {
          "type": "kuromoji_tokenizer",
          "mode": "search",
          "user_dictionary_rules": []
        }
      },
      "analyzer": {
        "my_kuromoji_analyzer": {
          "type": "custom",
          "tokenizer": "kuromoji_user_dict",
          "filter": [
            "kuromoji_baseform",
            "kuromoji_number",
            "katakana_readingform",
            "katakana_stemmer",
            "ja_stop"
          ]
        }
      },
      "filter" : {
        "katakana_readingform" : {
          "type" : "kuromoji_readingform",
          "use_romaji" : false
        },
        "katakana_stemmer": {
          "type": "kuromoji_stemmer",
          "minimum_length": 4
        },
        "ja_stop": {
          "type": "ja_stop",
          "stopwords": []
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "my_kuromoji_analyzer"
      },
      "body": {
        "type": "text",
        "analyzer": "my_kuromoji_analyzer"
      },
      "tags": {
        "type": "text",
        "analyzer": "my_kuromoji_analyzer"
      }
    }
  }
}'
curl -H "Content-Type: application/json" -X POST 'localhost:9200/admin/_open'
```

## Sudachiを使うても。。。

https://github.com/WorksApplications/elasticsearch-sudachi
https://tsgkdt.hatenablog.jp/entry/2019/04/09/180738
https://qiita.com/t2hk/items/a5b647f4ca764b073a47

## mongo-connectorが起動しない件について

dockerコンテナを単体で起動してログを調べる

```sh
$ sudo docker-compose logs mongo-connector
Attaching to mongo-connector
mongo-connector    | Logging to /mongo-connector.log.
$ sudo docker ps -a
$ sudo docker run -it --entrypoint=/bin/sh docker_mongo-connector

$ sudo docker-compose run --service-ports --no-deps --entrypoint /bin/sh mongo-connector
$ mongo-connector -m mongodb1 -t elasticsearch:9200 -d elastic2_doc_manager --continue-on-error --auto-commit-interval=0
$ vi mongo-connector.log
# 2020-04-26 04:20:26,151 [ERROR] mongo_connector.connector:381 - No replica set at "mongodb1"! A rep
```