
# SpringBootJpa 개발 | DAY 
```
하다가 막히는 부분이나 알아야 할 사항, 혹은 버전의 문제 등을 적기 위한 간단한 노트이며

이는 공부용 소스이기 때문에 소스에 주석을 달아가며 설명을 적어뒀습니다.
따라서 해당 날짜의 소스를 보며 노트를 참고하는 식으로 봐야합니다.
-/-
소스는 깃헙에 올려놓았습니다.
```

### ModelMapper Library

기존의 DTO 객체에 파라미터를 하나하나 get 해서 넣어주는 것이 보기 불편했고 그래서 수업 중에 선생님이 흘리듯이 말한 `ModelMapper`를 혼자 적용해보고자 합니다.



컨트롤러에서 아래와 같이 생성자에 리퀘스트 파라미터를 넣어주면
```java
model.addAttribute(new Profile(account));
```
아래처럼 DTO 생성자에서 하나하나 get해서 넣어주고 있습니다.
```
public Profile(Account account) {  
    this.bio = account.getBio();  
    this.url = account.getUrl();  
    this.occupation = account.getOccupation();  
    this.location = account.getLocation();  
}
```
appConfig에 우선 Bean 등록 해주고
```java
@Bean  
public ModelMapper modelMapper(){  
    return new ModelMapper();  
}
```

```java
@Autowired  
private final ModelMapper modelMapper;  
  
@GetMapping("/settings/profile")  
public String profileUpdate(@CurrentUser Account account, Model model){  
    model.addAttribute(account);
    model.addAttribute(modelMapper.map(account, Profile.class));  
    return "settings/profile";  
}
```
### Field injection is not recommended

뜬금 없지만 Bean 생성 과정에서 인텔리제이가 ` Field injection is not recommended `를 띄우길래 필드주입을 왜 권장하지 않는 건가 하고 찾아본 글입니다.

글이 너무 좋아서 소장하고 싶어서 올립니다.
[객체 주입에 대한 이야기](https://yaboong.github.io/spring/2019/08/29/why-field-injection-is-bad/)

요약하자면 **생성자 주입을 권장한다.** 입니다
