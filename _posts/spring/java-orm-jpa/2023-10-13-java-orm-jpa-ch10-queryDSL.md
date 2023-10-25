---
title:  "[자바 ORM 표준 JPA] ch10.객체지향 쿼리 - queryDSL "

categories:
  java-orm-jpa
tags:
  - [java-orm-jpa]

toc: true
toc_sticky: true

breadcrumbs: true

date: 2023-10-13
last_modified_at: 2023-10-13
---

**! 책에선 Maven을 이용했지만 전 Gradle을 사용하고 있기 때문에 구글링으로 찾아가면서 설정했음을 알려드립니다 !**

### QueryDSL이란
엔티티를 기반으로 쿼리 타입이라는 쿼리용 클래스를 생성합니다.
`Querydsl JPA`는 프로젝트 내에서 @Entity 애노테이션이 붙은 클래스를 기반으로 **JPAAnnotationProcessor** 를 이용하여 Qclass를 생성합니다. 
이렇게 생성된 Qclass는 정적코드로 쿼리문을 작성할 수 있게 합니다.

![생성된 Qclass](docs/assets/images/java-orm-jpa/10/qclass.png)
### gradle 파일
```groovy

buildscript {
    ext {
        queryDslVersion = "5.0.0";
    }
}

plugins {
    id 'java'
    id 'com.ewerk.gradle.plugins.querydsl' version '1.0.10'
}

group = 'org.example'
version = '1.0-SNAPSHOT'

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.hibernate:hibernate-core:5.4.32.Final'
    implementation 'org.postgresql:postgresql:42.2.27'
    implementation 'org.hibernate.javax.persistence:hibernate-jpa-2.1-api:1.0.2.Final'
    implementation 'org.projectlombok:lombok:1.18.26'
    annotationProcessor 'org.projectlombok:lombok:1.18.26'
    testImplementation platform('org.junit:junit-bom:5.9.1')
    testImplementation 'org.junit.jupiter:junit-jupiter'

    // QueryDSL
    implementation 'com.querydsl:querydsl-jpa:5.0.0'
    annotationProcessor 'com.querydsl:querydsl-apt:5.0.0'
    annotationProcessor "jakarta.annotation:jakarta.annotation-api:2.1.1"
    annotationProcessor "jakarta.persistence:jakarta.persistence-api:2.2.3"
}

test {
    useJUnitPlatform()
}

def querydslDir = "src/main/generated"

querydsl {
    jpa = true
    querydslSourcesDir = querydslDir
}

sourceSets {
    main {
        resources {
            srcDirs 'src/main/resources'
        }
        java {
            srcDirs querydslDir
        }
    }
}

/* 10-8. querydsl이 complieClasspath를 상속하도록 설정 */
configurations {
    querydsl.extendsFrom compileClasspath
}

compileQuerydsl {
    options.annotationProcessorPath = configurations.querydsl
}


```

[참고 인프런](https://www.inflearn.com/questions/536382/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-2-6-7-java-8-gradle-7-4-x-querydsl-%EC%84%A4%EC%A0%95-%EB%B0%A9%EB%B2%95-%EA%B3%B5%EC%9C%A0)
[참고 블로그](https://tecoble.techcourse.co.kr/post/2021-08-08-basic-querydsl/)

### QyeryDsl 사용하기 
Querydsl JPA 모듈은 JPA와 Hibernate API를 모두 지원합니다. 
JPA API를 사용하려면 다음과 같이 쿼리에 JPAQuery 인스턴스를 사용합니다.
