
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