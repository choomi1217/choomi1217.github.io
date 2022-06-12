## JUnit 을 사용한 테스트 코드

테스트 코드 작성을 도와주는 프레임워크중 하나인 xUnit을 사용

```
@WebMvcTest(controllers = HelloController.class)
public class HelloControllerTest {

    @Autowired
    private MockMvc mvc;

    @Test
    public void ReturnHello() throws Exception {
        String hello = "hello";
        mvc.perform(
                get("/hello"))
                .andExpect(status().isOk())
				
                .andExpect(content().string(hello));
    }

}

```

JUnit 5부터는 @RunWith가 아닌 Extension이라는 일관된 방법을 통해 테스트를 실행하는 방법을 커스터 마이징 하는 것이다. 
<br/>
사용법은 @RunWith와 비슷하게 @ExtendWith라는 애노테이션을 사용해서 
<br/>
@ExtendWith(MyExtension.class)처럼 Extension 구현체를 지정
<br/>
@ExtendWith 는 메타어노테이션으로 다른 어노테이션에 포함될 수 있는 어노테이션이므로
<br/>
@ExtendWith 를 포함한 어떤 어노테이션이 설정되어 있다면 @ExtendWith 를 생략 할 수 있다.
<br/>


- @WebMvcTest
	- Web(Spring MVC)에 집중할 수 있는 어노테이션 ( MVC를 위한 테스트 )
	- 웹에서 테스트하기 힘든 컨트롤러를 테스트하는 데 적합.
	- 웹상에서 요청과 응답에 대해 테스트할 수 있음.
	- 시큐리티, 필터까지 자동으로 테스트하며, 수동으로 추가/삭제 가능.
	- Application Context 완전하게 Start 시키지 않고 web layer를 테스트 하고 싶을 때

- org.springframework.test.web.servlet.MockMvc
	- 웹 API 테스트시 이용
	- 스프링 MVC 테스트의 시작점
	- 이 클래스를 통해 HTTP GET, POST ... 등 API 테스트
	
- mvc.perform( get("/hello") )
	- MockMvc를 통해 /hello 주소로 HTTP GET 요청
	- perform 메소드로 테스트 할 url을 설정
	- 체이닝 지원
	- DispatcherServlet에 요청을 의뢰하는 역할을 한다.
	- MockMvcRequestBuilder클래스를 사용해 설정한 요청 데이터를 perform()의 인수로 전달한다.
	- get, post, put, delete, fileUpload 와 같은 메서드를 제공한다.
	- ResultActions 이라는 인터페이스를 반환한다.

- perform 메소드로 테스트 할 url을 설정했다면 이 url이 요청할 데이터 또한 설정할 수 있다.
---

## 체이닝 테스트 코드

```
public class ChainningTest {

    @Test
    public void testObjectSetter(){
        TestPeaple peaple = new TestPeaple();
        peaple.setAge(26).setHeight(168).setName("앵미");
    }

}
```
```
public class TestPeaple {
    private int age;
    private int height;
    private String name;

    public int getAge() {
        return age;
    }

    public int getHeight() {
        return height;
    }

    public String getName() {
        return name;
    }

    public TestPeaple setAge(int age){
        this.age = age;
        return this;
    }

    public TestPeaple setHeight(int height){
        this.height = height;
        return this;
    }

    public TestPeaple setName(String name){
        this.name = name;
        return this;
    }
}
```

return으로 자신을 객체로 해서 돌려주면 체이닝 객체생성 할 수 있음

---

## Test코드 작성하면서 사용한 라이브러리

JUnit5: 자바 단위 테스트를 위한 테스팅 프레임워크
AssertJ: 자바 테스트를 돕기 위해 다양한 문법을 지원하는 라이브러리

---

## 단위 테스트의 패턴

- given/when/then 패턴
given-when-then 패턴이란 1개의 단위 테스트를 3가지 단계로 나누어 처리하는 패턴으로, 각각의 단계는 다음을 의미한다.

given(준비): 어떠한 데이터가 준비되었을 때
when(실행): 어떠한 함수를 실행하면
then(검증): 어떠한 결과가 나와야 한다.

