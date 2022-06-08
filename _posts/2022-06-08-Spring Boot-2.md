
## SpringBoot의 시작점 만들기

#### Application.java
```
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class,args);
    }
}
```
#### @springBootApplication 의 기능
	- 스프링 부트의 자동설정
	- 스프링 Bean 읽기와 생성을 모두 자동으로
	- 이 어노테이션이 있는 부분부터 설정을 읽음
		- 고로 이 클래스는 항상 프로젝트 최상단에 위치해야함.

#### SpringApplicaion.run
	- main에서 실행하는 이 메소드가 내장WAS를 실행합니다.
		- 내장WAS : 
			- WAS를 외부에 두지 않음
			- 톰캣 설치 필요가 없어짐
			- JAR 파일 실행

---

#### Hello.java
```
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello(){
        return "hello";
    }
}
```
#### @RestController
	- 컨트롤러를 JSON을 반환하는 컨트롤러로 생성
#### @GetMapping
	- Get Method




---


## JUnit 을 사용한 테스트 코드

테스트 코드 작성을 도와주는 프레임워크중 하나인 
xUnit을 사용

---

JUnit 5부터는 @RunWith가 아닌 Extension이라는 일관된 방법을 통해 테스트를 실행하는 방법을 커스터 마이징 하는 것이다. 
사용법은 @RunWith와 비슷하게 @ExtendWith라는 애노테이션을 사용해서 @ExtendWith(MyExtension.class)처럼  Extension 구현체를 지정
@ExtendWith 는 메타어노테이션으로 다른 어노테이션에 포함될 수 있는 어노테이션이므로
@ExtendWith 를 포함한 어떤 어노테이션이 설정되어 있다면 @ExtendWith 를 생략 할 수 있다.



