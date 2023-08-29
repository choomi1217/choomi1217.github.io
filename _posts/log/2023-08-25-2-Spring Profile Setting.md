---
title:  "[개발 기록] Spring Profile Setting"

categories:
  - log
tags:
  - [log]

toc: true
toc_sticky: true

breadcrumbs: true

date: 2023-08-25
last_modified_at: 2023-08-25
---

스프링 프로필을 설정하려던 이유

Local 에서와 Test 서버에서의 테스트용 파일의 경로가 달라서 테스트 서버에 배포 했을때 파일 경로를 매번 바꿔 넣어야 했던 점이 참 신경이 쓰였다.

```yaml
# locla share_folder 경로
file:
  path: /Users/ymcho/dev/python/pyproject/pytest/test_tiles3

# test share_folder 경로
file:
  path: /home/secret_company/secret_project_name/share_folder/tiles
```

### 1. YAML을 local, dev, prod 3개로 나누기

### 2. local에선 Intellij의 vm에 프로필 설정 해줄 수 있는 방법이 따로 있음

### 3. test서버와 prod 서버는 각각 도커 파일을 다르게 설정하는 수밖에 없음

1. Docker Compose

```docker
version: "3.7"

services:
  pm2023_jsat_be:
    image: pm2023_jsat_be
    build:
      context: ./
      dockerfile: Dockerfile
    ports:
      - 8080:8080
    environment:
      - SPRING_PROFILES_ACTIVE=dev
```

1. DockerFile

```docker
# 빌드 스테이지
FROM maven:3.8.1-jdk-11 as build
WORKDIR /app
COPY . /app
RUN mvn clean package
RUN mvn -B -f pom.xml clean package -DskipTests

# 실행 스테이지
FROM openjdk:11-jre-slim
WORKDIR /app
COPY --from=build /app/target/innopam_jsat-1.0.0.jar /app/
EXPOSE 8080
CMD ["java", "-Dspring.profiles.active=dev", "-jar", "/app/innopam_jsat-1.0.0.jar"]
```

### 좀 더 생각해 본 결과

경로가 달라서 테스트가 깨진다면 그건 문제이다.

고로 로컬과 테스트 서버의 경로를 똑같이 설정하는게 맞다.

그러니까.. 그게 불가능한건데.. 클라우드를 쓸 수는 없고… 도저히 방법을 모르겠다..

docker에 올려서 테스트를 해야하는건가 생각중이다 볼륨으로 클래스 파일을 마운트하더라도 빌드 해야하니까 아마 안되지 않을까?

🥺

---

엄청난 삽질을 3시간동안 했다.

아래 구문 때문에 @ActiveProfile을 제거하면 테스트코드가 전부 깨지는 일이 발생했다.

내가 설정한 yml에 저런게 박혀있는줄도 모르고.. 🤦‍♀️

```yaml
 config: 
	activate: 
		on-profile: local 
```

```yaml
server:
  port: 8080

spring:
  main:
    allow-bean-definition-overriding: true
  mvc:
    pathmatch:
      matching-strategy: ant_path_matcher
  config:
    activate:
      on-profile: local

  # DB setting
  # Database
  datasource:
    driver-class-name: org.postgresql.Driver
    url: jdbc:postgresql:secret
    username: secret
    password: secret

# Mybatis 설정
mybatis:
  mapper-locations: classpath:/mybatis/mapper/**/*.xml
  config-location: classpath:/mybatis/mybatis-config.xml

# share_folder 경로
file:
  path: /secret
```