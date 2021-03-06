# Cache 란?

Cache 는 임시 저장소다.

복잡한 계산이나 데이터베이스 조회, 외부 API 요청 등을 캐시에 미리 저장해두고 사용자 요청이 들어오면 캐시에 있는 데이터를 꺼내준다.

<br>

## Cache Hit 과  Miss

메모리에 데이터가 존재하는 경우 Cache Hit 이라고 한다.

메모리에 데이터가 존재하지 않는 경우 Cache Miss 라고 한다.

Cache Miss 가 발생하는 경우 실제 DBMS 에서 데이터를 가져온다. 

그리고 캐시 전략에 따라서 데이터를 갱신한다.

<br>

## Cache 를 사용하는 경우와 사용하지 않는 경우

- Good Case
    - 자주 접근하는 데이터를 조회하는 시간이 오래 걸리는 경우 (API 호출, 복잡한 DB 쿼리 등)
    - 반복적으로 동일한 결과를 돌려주는 경우
- Bad Case
    - 요청할 때마다 다른 결과를 돌려주는 경우 (캐시 갱신이 자주 일어나서 오히려 더 안좋다)
    - 데이터가 자주 변경되는 경우
    - 애초에 데이터 가져오는 시간이 짧은 경우

<br>

## Cache 를 주로 사용하는 곳

- DB 캐시
    - DB 조회하는 쿼리가 오래 걸리는 경우 서버에 있는 메모리를 캐시로 사용한다.
    - 예를 들면 데이터가 거의 바뀌지 않는 document 나 공지사항 페이지에 있는 데이터는 캐시를 사용하는 게 좋다. 캐시를 사용하지 않으면 1000 명의 사용자가 접근 할 때마다 1000 번의 쿼리를 날린다.
- 브라우저 캐시
    - 브라우저가 자체적으로 리소스를 캐시로 갖고 있는다.
    - 이미지 같은 파일의 경우 매번 클라이언트에 전달되면 통신 비용이 많이 든다.
