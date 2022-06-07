### 스프링 부트와 AWS로 혼자 구현하는 웹 서비스 - 1일차

```
buildscript {
    ext{
        springBootVersion = '2.1.7.RELEASE'
    }
    repositories{
        mavenCentral()
        jcenter()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
}
```

#### ext 
	- build.gradle에서 사용하는 전역변수를 설정하겠다는 의미
	- springBootVersion 전역변수를 생성하고 그 값을 '2.1.7.RELEASE'로 하겠다


repositories 중 
'jcenter' is deprecated 

`jcenter`는 mavenCentral의 단점인 라이브러리 업로드의 복잡성을 개선한 저장소입니다.
하지만 지금은 보안 문제로 사용 X

`repositories`는 각종 의존성들을 어떤 원격 저장소에서 받을지 결정
`mavenCentral()`을 많이 사용


```
apply plugins: 'java'
apply plugins: 'eclipse'
apply plugins: 'org.springframework.boot'
apply plugins: 'io.spring.dependency-management'
````

앞서 선언한 플러그인 의존성들을 적용할 것인지 결정하는 코드

```
apply plugins: 'io.spring.dependency-management'
```
이 플러그인은 스프링 부트의 의존성들을 관리해 주는 플러그인

```
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```
교재 상에선 
compile 이라고 쓰여있지만 implementation으로 


완성된 build.gradle

```
buildscript {
    ext{
        springBootVersion = '2.1.7.RELEASE'
    }
    repositories{
        mavenCentral()
        maven {
            url "https://plugins.gradle.org/m2/"
        }
//        jcenter()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
}

plugins {
    id 'java'
    id 'eclipse'
    id "org.springframework.boot" version "2.7.0"
    id "io.spring.dependency-management" version "1.0.11.RELEASE"
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
//    implementation('org.springframework.boot:spring-boot-starter-websocket:2.6.7')
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

group 'com.edu.book'
version '1.0-SNAPSHOT'

repositories {
    mavenCentral()
}

dependencies {
    testImplementation 'org.junit.jupiter:junit-jupiter-api:5.8.1'
    testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine:5.8.1'
}

test {
    useJUnitPlatform()
}
```

git commit은 생략

-----

