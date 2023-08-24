## 문제의 상황

테스트 서버 배포 후, 에러가 나서 확인 해보려고 했는데 로그가 어디로 생기는지 몰라서 헤매다가

`docker logs [container_id or container_name]` 이 명령어를 통해 도커에서 확인이 가능 하다는걸 깨달음.

하지만 매번 이렇게 도커 명령어를 치는것 (컨테이너 아이디나 이름이 기억 안나면 찾는것 까지 해야함) 이렇게 보는 것보다 로그 파일로 만드는게 더 효율적일것 같음.

벨둥 아저씨 글을 읽다가 또 스프링부트를 도커로 올리는 거에 대한 글을 봄. 이건 나중에 해보기로 함

[Logging in Spring Boot | Baeldung](https://www.baeldung.com/spring-boot-logging)

[Creating Docker Images with Spring Boot | Baeldung](https://www.baeldung.com/spring-boot-docker-images)

현재 스프링부트를 실행하는 도커 파일

```yaml

```