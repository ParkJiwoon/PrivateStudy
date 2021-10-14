# Kotlin 생성자, constructor, init

- Kotlin 은 primary constructor 와 secondary constructor 가 있다.
- init 블록은 여러개 선언할 수 있으며 위에서부터 순차적으로 실행된다.
- init 블록은 primary constructor 의 일부라서 항상 secondary constructor 보다 먼저 실행된다.
- 실행 순서: primary constructor (property 초기화) -> init -> secondary constructor