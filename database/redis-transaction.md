# Redis Transaction

Redis Transaction 이란 여러 Redis Command 를 하나의 블록으로 모아둔 겁니다.

트랜잭션에서 사용하는 명령어에는 `multi`, `exec`, `discard`, `watch` 등이 있습니다.

실행된 명령어들은 Queue 에 담겨 순서대로 실행되기 때문에 다른 명령어의 영향을 받지 않아 독립성이 보장됩니다.

단점으로는 트랜잭션 롤백이 지원되지 않기 때문에 중간 명령어에서 에러가 발생해도 이전 명령어들이 취소되지 않습니다.

- `multi` : 트랜잭션 블록 시작
- `exec` : 트랜잭션 종료
- `discard` : 트랜잭션 중단
- `watch` / `unwatch` : key 의 변경을 감지

`watch` 를 선언한 key 값은 외부에서 변경이 감지되면 변경을 막습니다.