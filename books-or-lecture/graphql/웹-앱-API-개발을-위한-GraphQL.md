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

GraphQL 에서 제공하는 강력한 기능으로 현재 API 스키마의 세부 사항에 관한 쿼리를 작성할 수 있습니다.

<br>

# Ch 4. 스키마 설계하기

GraphQL API 를 만들기 전에는 스키마를 설계해야 합니다.

GraphQL 은 스키마 정의를 위해 SDL (Schema Definition Language) 를 지원합니다.

## 4.1.1. 타입

타입 (Type) 은 커스텀 객체이며 이를 보고 애플리케이션의 핵심 기능을 알 수 있습니다.

예를 들어 소셜 미디어 애플리케이션은 `Users`, `Posts` 로 구성되고 블로그는 `Categories`, `Articles` 로 구성됩니다.

타입에는 필드 (field) 가 들어갑니다.

필드는 각 객체의 데이터와 관련이 있습니다.

스키마에는 타입 정의를 모아 둡니다.

<br>

```graphql
type Photo {
    id: ID!
    name: String!
    url: String!
    description: String
}
```

마지막에 붙은 느낌표는 non-nullable 을 의미합니다.

설명은 필수값이 아닙니다.

ID 스칼라 타입은 고유 식별자 값이 반환되어야 하는 곳에 씁니다.

ID 스칼라 타입은 문자열 타입이지만 고유한 값인지 유효성 검사를 받습니다.

<br>

## 4.1.2. 스칼라 타입

GraphQL 내장 스칼라 타입 (Int, Float, String, Boolean, ID) 대신 직접 스칼라 타입을 만들 수 있습니다.

스칼라 타입은 객체 타입이 아니기 때문에 필드를 가지지는 않지만 유효성 검사 방식을 지정할 수 있습니다.

<br>

## 4.1.3. 열거 타입 (Enumeration)

열거 타입은 스칼라 타입에 속합니다.

```graphql
enum PhotoCategory {
    SELFIE
    PORTRAIT
    ACTION
    LANDSCAPE
    GRAPHIC
}
```

열거 타입을 만들면 다른 객체 타입에 필드로 추가할 수 있습니다.

```graphql
type Photo {
    id: ID!
    category: PhotoCategory!
}
```

<br>

## 4.2. 연결과 리스트

리스트는 GraphQL 타입을 대괄호로 감싸서 만듭니다. `[String]`, `[PhotoCategory]` 등등

리스트의 null 적용 규칙만 헷갈리지 않게 사용합니다.

- `[Int]`: 리스트와 정수 전부 null 이 될 수 없음
- `[Int!]`: 리스트는 null 이 될 수 없지만 정수는 null 이 될 수 있음
- `[Int]!`: 리스트는 null 이 될 수 있고 정수는 null 이 될 수 없음
- `[Int!]!`: 리스트와 정수 전부 null 이 될 수 없음

근데 대부분은 `[Int]` 처럼 둘다 null 이 될 수 없는 타입을 많이 사용합니다.

<br>

### 4.2.1. 일대일 연결

```graphql
type User {
    name: String
}

type Photo {
    id: ID!
    name: String!
    url: String
    postedBy: User!
}
```

사용자는 사진을 게시할 수 있기 때문에 타입끼리 연결 관계를 만들 수 있습니다.

<br>

### 4.2.2. 일대다 연결

```graphql
type User {
    name: String
    postedPhotos: [Photo!]!
}
```

만약 유저 쿼리를 요청할 때 사진들을 받고 싶다면 다음과 같이 리스트로 넣어주면 됩니다.


<br>

### 4.2.3. 다대다 연결

사진에 다른 사람을 태그할 수 있는 기능이 생긴다면 다대다 관계가 형성됩니다.

사진 속에는 다른 사람을 태그할 수 있고 사용자는 여러 사진에 태그당할 수 있습니다.

다대다 연결 관계를 만드려면 `User` 와 `Photo` 타입 양쪽 모두에 리스트 타입 필드를 추가하면 됩니다.

```graphql
type User {
    ...
    inPhotos: [Photo!]!
}

type Photo {
    ...
    taggedUsers: [User!]!
}
```

<br>

다대다 연결을 만들 경우 관계에 대한 정보를 담는 **통과 타입 (Through Type)** 이라는 걸 만들 수도 있습니다.

(DB 로 따지면 Relation Table 같은 느낌)

사용자와 사용자의 관계를 표현하기 위해 서로 연결해야 한다고 가정합니다.

사람들은 한사람이 여러 사람과 관계를 맺을 수 있으므로 다대다로 연결됩니다.

```graphql
type User {
    friends: [User!]!
}
```

<br>

그런데 만약 사람 사이에 관계 자체에도 만난 시간, 장소 등을 표현하고 싶다면 별도의 타입을 만들 수도 있습니다.

```graphql
type User {
    friends: [Friendship!]!
}

type Friendship {
    friend_a: User!
    friend_b: User!
    howLong: Int!
    whereWeMet: Location
}
```

<br>

### 4.2.4. 여러 타입을 담는 리스트

Union 타입이나 Interface 타입을 사용해서 만들 수 있습니다.

일정 앱을 예시로 사용합니다.

- 일정에는 여러 종류의 이벤트가 있음
- 이벤트는 스터디 그룹 모임 이벤트와 운동 이벤트가 존재하고 둘의 필드는 완전히 다를 수 있음
- 일정 앱에는 다른 이벤트 타입을 모두 추구할 수 있음
- 하루의 일정은 여러 활동 타입을 모아둔 리스트

<br>

**유니언 타입 (Union)**

유니언 타입을 사용하면 여러 타입 가운데 하나를 반환합니다.

```graphql
union Event = StudyGroup | Workout

type StudyGroup {
    name: String!
    subject: String
    students: [User!]!
}

type Workout {
    name: String!
    reps: Int!
}
```

<br>

**인터페이스 (Interface)**

한 필드 안에 여러 타입을 넣을 때 사용할 수 있습니다.

추상 타입이며 특정 필드가 무조건 특정 타입에 포함되도록 만들 수 있습니다.

```graphql
interface Event {
    name: String!
    start: DateTime!
    end: DateTime!
}

type StudyGroup implements Event {
    name: String!
    start: DateTime!
    end: DateTime!
    subject: String
    students: [User!]!
}

type Workout implements Event {
    name: String!
    start: DateTime!
    end: DateTime!
    reps: Int!
}
```

<br>

## 4.3. 인자

쿼리 요청 시 인자값을 넘겨줄 수 있습니다.

<br>


### 4.3.1. 데이터 필터링

특정 파라미터를 넘겨주면 해당하는 객체들만 필터링 해서 제공합니다.

```graphql
query {
    User(name: "Gildong") {
        name
        avatar
    }
}
```

<br>

### 4.3.2. 데이터 페이징

응답 리스트의 양을 조절하기 위해 **데이터 페이징**을 할 수도 있습니다.

```graphql
type Query {
    ...
    allUsers(first: Int=50 start: Int=0): [User!]!
    allPhotos(first: Int=25 start: Int=0): [Photo!]!
}
```

`allUsers` 는 0 부터 시작해서 50 명을 뽑고 `allPhotos` 는 0 부터 시작해서 25 장만 뽑습니다.

<br>

### 4.3.3. 데이터 정렬

인자값을 사용하면 데이터 **정렬**도 가능합니다.

정렬 방향을 넣지 않으면 기본적으로 내림차순으로 정렬됩니다.

```graphql
enum SortDirection {
    ASCENDING
    DESCENDING
}

enum SortablePhotoField {
    name
    description
    category
    created
}

Query {
    allPhotos(
        sort: SortDirection = DESCENDING
        sortBy: SortablePhotoField = created
    ): [Photo!]!
}
```

<br>

## 4.4. 뮤테이션

뮤테이션은 데이터 생성, 수정, 삭제에 관련된 명령어입니다.

아래 명령어는 사진을 게시하는 명령어 스키마입니다.

```graphql
type Mutation {
    postPhoto(
        name: String!
        description: String
        category: PhotoCategory
    ): Photo!
}

schema {
    query: Query
    mutation: Mutation
}
```

<br>

## 4.5. 인풋 타입

쿼리와 뮤테이션의 파라미터 길이가 꽤 길어졌으면 Input 타입을 사용해서 짧게 만들 수 있습니다.

```graphql
input PostPhotoInput {
    name: String!
    description: String
    category: PhotoCategory = PORTRAIT
}

type Mutation {
    postPhoto(input: PostPhotoInput!): Photo!
}
```

`PostPhotoInput` 타입은 객체 타입과 비슷하게 생겼지만 전달하는 인자에만 사용합니다.

<br>

## 4.6. 리턴 타입

파라미터 타입처럼 리턴 타입도 따로 정의해서 받을 수 있습니다.

```graphql
type AuthPayload {
    user: User!
    token: String!
}

type Mutation {
    ...
    githubAuth(code: String!): AuthPayload!
}
```

<br>

## 4.7. Subscription

패스

<br>

# Ch 5 ~ Ch 7

특정 프레임워크 의존적이라서 PASS

