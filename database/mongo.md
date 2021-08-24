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
# DB 목록 조회
> show dbs

# 특정 DB 사용
> use {db}

# DB 정보 보기
> db.stats()

# 컬렉션 조회
> show collections

# DB 에 컬렉션 추가
> db.createCollection("item")
{ "ok" : 1 }

# 특정 컬렉션에 데이터 추가 db.{collection}.{명령어}
> db.item.insert({"name": "Alf alarm clock", "price": 19.99})
WriteResult({ "nInserted" : 1 })

# 특정 컬렉션 조회
> db.item.find()
{ "_id" : ObjectId("6123af9af07dabcf065d83fe"), "name" : "Alf alarm clock", "price" : 19.99 }
```