1. 쿠키 세션 방식 단점
- 세션 저장소 사용 (메모리 필요)
- 매 요청마다 저장소 값과 검증해야함 (성능 저하)
- 세션은 서로 다른 도메인간에는 공유되지 않음
- 모바일 환경에서 사용하기 불편함

2. Access Token (JWT) 사용 장점
- 세션 저장소 필요 없음 (메모리 절약)
- 매 요청마다 저장소를 확인하지 않아도 됨 (성능 향상)

3. Access Token (JWT) 사용 단점
- 추가 검증이 없기 때문에 탈취되면 누구나 해당 토큰 정보를 사용 가능 (유일한 단점이지만 치명적임)

4. 만료 시간을 짧게 하자
- 탈취 피해를 최소화 하기 위해 만료 시간을 짧게 변경 (일반적으로 30분)
- 그러나 너무 짧으면 사용자가 매번 로그인해야 하는 번거로움 발생

5. Refresh Token 사용
- 사용자 로그인을 자동연장 시켜주기 위해 Refresh Token 발급
- Access Token 이 만료되면 Refresh Token 으로 새로 발급 가능
- 다른 환경에서 로그인 하는 등 추가적인 검증 가능
- Oauth2 와 같이 확장 가능
- 모바일 환경에서도 사용 가능

6. Refresh Token 한계
- 결국 Refresh Token 도 탈취당할 가능성이 있음

7. 탈취 대안
- Refresh Token 을 저장소에 저장
- 탈취된 토큰은 저장소에서 삭제하여 강제 로그아웃 처리 가능

8. 저장소 사용으로 메모리 및 성능 문제 롤백?
- 세션 방식의 단점인 메모리 문제와 저장소 조회로 인한 성능 저하 발생하지 않나?
- 메모리 문제는 어쩔수 없음 (보안을 위해 포기해야함)
- 세션은 매 API 마다 검증하지만 리프레시 토큰의 검증 주기는 길어서 좀 더 나음
- 그리고 평소 요청마다 세션은 write/update 와 같은 i/o 에서 단순 복호화만 하는 JWT 에 비해 서버 부담이 됨

9. 결론
- 효율만을 따지면 Access Token only 방식이 젤 나음
- 하지만 보안을 위해 어느정도 타협할 필요가 생겼기에 Refresh Token 이 등장
- 쿠키/세션 방식에 비해 메모리는 큰 이점이 없지만 성능적으로는 향상
