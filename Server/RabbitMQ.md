# RabbitMQ 란

RabbitMQ 는 AMQP 프로토콜을 구현한 Message Broker 입니다.

1. Producer 가 메세지를 전송 한다 (Publish)
2. Exchange 가 Exchange Type 과 Routing Key 에 맞게 메세지를 Queue 에 전달한다.
3. 메세지는 소비될 때까지 Queue 에 대기하고 있다가 Consumer 에 의해 소비된다.

```html
Producer -> Exchange -> Queue -> onsumer
```

<br>

# 용어

#### *Producer*
- 메세지를 보내는 Application
- Message Queue 에 넣어주는(쏴주는) 사람

#### *Publish*
- 메세지를 보내는 행위
- Queue 에 넣는 행위

#### *Queue*
- 메세지를 저장하는 버퍼

#### *Consumer*
- 메세지를 받는 Application
- 동일 업무를 처리하는 Consumer 는 보통 하나의 Queue를 바라본다.
- 동일 업무를 처리하는 Consumer 가 여러개인 경우 같은 Queue를 바라보게 하면 자동으로 메세지를 분배하여 전달

#### *Subscribe*
- Consumer 가 메세지를 수신하기 위해 Queue 를 바라보게 하는 행위

#### *Exchange*
- Producer 가 전달한 메세지를 Queue 에 전달하는 역할
- 메세지는 Queue 에 직접 전달되지 않고 Exchange Type 정의대로 동작

#### *Exchange Type*
Type | 설명 | 특징
:---: | :--- | :---: |
`fanout` | 알려진 모든 Queue 에 메세지를 전달 (Routing Key 를 무시하고 모든 Exchange 에 바인딩 된 모든 Queue 에 전달한다) | `Broadcast`
`direct` | Exchange 에 바인딩 된 Queue 중에서 지정된 Routing Key 를 가진 Queue 에만 메세지를 전달 | `unicast`
`topic` | 지정된 패턴 바인딩 형태에 일치하는 Queue 에 모두 전달. #(여러단어), *(한단어)를 통한 문자열 패턴 매칭<br>Routing Key 로 '#'를 지정한다면 __fanout__ 과 동일하게 동작하고 #, * 없이 단어 하나만 입력하면 __direct__ 와 동일하게 동작 | `multicast`
`header` | 헤더에 포함된 key=value 의 일치조건에 따라서 메세지 전달 | `multicast`

#### *Bindings*
- Exchange 와 Queue 를 연결해주는 것

#### *Routing*
- Exchange 가 Queue 에 메세지를 전달하는 과정

#### *Routing Key*
- 어떤 Queue 와 Binding 될 지 결정하는 기준
