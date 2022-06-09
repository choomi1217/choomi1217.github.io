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
	- 체이닝 지원










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

집에 가서 마저 작성하기! 

https://wayhome25.github.io/git/2017/07/08/git-first-pull-request-story/

으윽...잘래...

