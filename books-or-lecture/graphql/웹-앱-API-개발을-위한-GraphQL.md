# 웹 앱 API 개발을 위한 GraphQL

[yes24 에서 구매](http://www.yes24.com/Product/Goods/116036952)

<br>

# Ch 1. GraphQL 에 오신 것을 환영합니다

GraphQL 이란 API 를 만들 때 사용할 수 있는 쿼리 언어입니다.

선언형 (declarative) 데이터 페칭 (fetching) 언어라고 흔히 일컬어집니다.

개발자는 '무슨' 데이터가 필요한지에 대한 요구사항만 작성하고 '어떻게' 가져올지는 신경쓰지 않아도 됩니다.

<br>

## REST vs GraphQL

- 오버페칭 (overfetching)
  - REST: API 의 응답값이 고정되어 있어서 필요 없는 데이터들도 전부 받게 됨
  - GraphQL: 원하는 필드값만 요청해서 받을 수 있음
- 언더페칭 (underfetching)
  - REST: 어떤 데이터 목록을 가져올 때 갯수만큼 API 요청을 해야함
  - GraphQL: 필드값을 요청하면 전부 한번에 가져옴
- 엔드포인트 관리
  - REST: 변경 사항이 발생하면 엔드포인트를 새로 만들어야 함
  - GraphQL: 유연성을 가짐
  
<br>

# Ch 2. 그래프 이론

PASS

<br>

# Ch 3. GraphQL 쿼리어

GraphQL 은 프로그래밍 언어에 종속되어 있지 않습니다.

데이터 조회에는 `query`, 데이터 수정에는 `mutation` 을 사용하면 됩니다.

<br>

## GrqphQL 쿼리

GraphQL 은 에러가 발생해도 200 응답이 옵니다.

정상 응답이 발생하면 `data` 키값이 오고 에러 응답이 발생하면 `error` 키가 들어있습니다.

<br>

## GraphQL 타입

### 루트 타입 (Root), 셀렉션 세트 (Selection Set)

```graphQL
query liftsAndTrails {
    liftcount(status: OPEN)
    allLifts {
        name
        status
    }
    allTrails {
        name
        difficulty
    }
}
```

`Query` 는 GraphQL 의 루트 타입입니다.

쿼리를 작성할 때는 필요한 필드를 중괄호로 감싸는데 이 블록을 셀렉션 세트 (Selection Set) 라고 합니다.

셀렉션 세트는 중첩 가능합니다.

<br>

### 별칭

쿼리의 응답 데이터는 JSON 포맷으로 내려옵니다.

보통 요청한 필드명과 동일하게 응답이 오는데, 필드명을 다르게 받고 싶다면 아래처럼 필드명에 별칭을 부여하면 됩니다.

```graphQL
query liftAndTrails {
    open: liftCount(status: OPEN)
    chairlifts: allLifts {
        liftName: name
        status
    }
    skiSlopes: allTrails {
        name
        difficulty
    }
}
```

위 쿼리에서는 `chairlifts`, `liftName`, `skiSlopes` 같은 별칭을 사용하였고 응답은 아래와 같습니다.

```json
{
    "data": {
        "open": 5,
        "chairlifts": [
            {
                "liftName": "Astra Express",
                "status": "open"
            }
        ],
        "skiSlopes": [
            {
                "name": "Ditch of Doom",
                "difficulty": "intermediate"
            }
        ]
    }
}
```

<br>

### 쿼리 인자 (Query Arguments)

쿼리 인자 (Query Arguments) 를 사용해서 필터링 작업을 할 수도 있습니다.

```graphQL
query closedLifts {
    allLifts(status: CLOSED) {
        name
        status
    }
}
```

<br>

### 스칼라 타입 (Scalar), 객체 타입 (Object)

GraphQL 필드는 스칼라 (scalar) 타입과 객체 (object) 타입으로 이루어집니다.

스칼라 타입은 원시 타입과 비슷하며 Int, Float, String, Boolean, ID (고유식별자) 총 5개가 존재합니다.

객체 타입은 스키마에 정의한 필드를 그룹으로 묶어둔 것

<br>

### 프래그먼트 (Fragment)

Fragment 를 사용하면 중복된 필드를 하나로 모을 수 있습니다.

<br>

### 유니언 타입 (Union)

두 타입을 하나의 집합으로 묶습니다.

<br>

### 인터페이스 (Interface)

필드 하나로 객체 타입을 여러개 반환할 때 사용합니다.

추상적인 타입이며 유사한 객체 타입을 만들 때 구현해야 하는 필드 리스트를 모아둔 것입니다.

타입을 구현할 때는 인터페이스에 정의된 필드는 모두 넣어야 하고 다른 필드도 추가할 수 있습니다.

<br>

### 뮤테이션 (Mutation)

지금까지 본 쿼리는 전부 "읽기" 행위에 관한 기술입니다.

데이터를 새로 쓰려면 뮤테이션을 사용해야 합니다.

뮤테이션은 쿼리를 작성하는 방법과 비슷하며 이름을 붙여야 합니다.

<br>

```graphQL
mutation burnItDown {
    deleteAllData
}
```

Mutation 은 루트 객체 타입입니다.

위 예제 쿼리가 잘 실행되면 true 를 리턴합니다.

<br>

새로 만드는 API 요청도 가능합니다.

```graphQL
mutation createSong {
    addSong(title: "No Scrubs", numberOne: true, performerName: "TLC") {
        id
        title
        numberOne
    }
}
```

위 API 를 호출해서 생성하려는 값과 반환값을 결정할 수 있습니다.

`addSong` 은 필드명으로 지정됩니다.

<br>

```json
{
    "data": {
        "addSong": {
            "id": "5ksadfjsk333934a",
            "title": "No Scrubs",
            "numberOne": true
        }
    }
}
```

응답값은 위처럼 나옵니다.

<br>

### 쿼리 변수 사용하는 뮤테이션

뮤테이션의 인자값으로 동적인 값을 넣을 수도 있습니다.

```graphQL
mutation createSong($title:String! $numberOne:Int $by:String!) {
    addSong(title:$title, numberOne:$numberOne, performerName:$by) {
        id
        title
        numberOne
    }
}
```

GraphQL 은 언제나 변수명 앞에 `$` 문자가 붙습니다.

GraphQL 요청 툴에 쿼리 변수용 창이 따로 있으면 JSON 객체 형식으로 보내서 테스트 해볼 수 있습니다.

<br>

### 서브스크립션 (Subscription)

구독 모델로 웹소켓을 사용해서 통신합니다.

<br>

### 인트로스펙션 (Introspection)

