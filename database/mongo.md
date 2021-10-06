# 설치법

```sh
# 이미지 다운 (docker images 로 확인 가능)
$ docker pull mongo

# 컨테이너로 mongoDB 실행 (--name: 컨테이너 이름 설정, -p: 포트 포워딩, -d: 백그라운드에서 실행)
# (docker ps 로 확인 가능)
$ docker run --name mongo-container -p 27017:27017 -d mongo

# MongoDB cli 접속하려면 컨테이너 bash 실행 후 "mongo" 입력
$ docker exec -it mongo-container bash
```

# 명령어

```sql
# DB 목록 조회 (admin, config, test 세개의 db 존재)
> show dbs
admin   0.000GB
config  0.000GB
test    0.000GB

# 특정 DB 사용 (위에서 조회한 test db 로 스위치)
> use test
switched to db test

# DB 정보 보기
> db.stats()
{
	"db" : "test",
	"collections" : 1,
	"views" : 0,
	"objects" : 2,
	"avgObjSize" : 119,
	"dataSize" : 238,
	"storageSize" : 20480,
	"freeStorageSize" : 0,
	"indexes" : 1,
	"indexSize" : 20480,
	"indexFreeStorageSize" : 0,
	"totalSize" : 40960,
	"totalFreeStorageSize" : 0,
	"scaleFactor" : 1,
	"fsUsedSize" : 8257208320,
	"fsTotalSize" : 62725623808,
	"ok" : 1
}

# 컬렉션 조회 (item 이라는 컬렉션이 있음)
> show collections
item

# DB 에 컬렉션 추가
> db.createCollection("item")
{ "ok" : 1 }

# 특정 컬렉션에 데이터 추가 db.{collection}.{명령어}
> db.item.insert({"name": "Alf alarm clock", "price": 19.99})
WriteResult({ "nInserted" : 1 })

# 특정 컬렉션 조회
> db.item.find()
{ "_id" : ObjectId("6123af9af07dabcf065d83fe"), "name" : "Alf alarm clock", "price" : 19.99 }

# 특정 컬렉션 삭제
> db.item.drop()
true

# 특정 DB 삭제 (삭제하려는 DB 로 use 명령어를 사용해서 switch 후)
> db.dropDatabase()
{ "ok" : 1 }

# 컬렉션 내용 삭제 ({} 를 넣으면 전체 삭제)
> db.item.remove({})
WriteResult({ "nRemoved" : 2 })
```