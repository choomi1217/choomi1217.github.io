# SpringBootJpa 개발 | 3 DAY - 1
```
하다가 막히는 부분이나 알아야 할 사항, 혹은 버전의 문제 등을 적기 위한 간단한 노트이며

이는 공부용 소스이기 때문에 소스에 주석을 달아가며 설명을 적어뒀습니다.
따라서 해당 날짜의 소스를 보며 노트를 참고하는 식으로 봐야합니다.

소스는 깃헙에 올려놓았습니다.
```

### ModelMapper Library

기존의 DTO 객체에 파라미터를 하나하나 get 해서 넣어주는 것이 보기 불편했고  `ModelMapper`를 적용해보고자 합니다.

컨트롤러에서 아래와 같이 생성자에 리퀘스트 파라미터를 넣어주면

```java
model.addAttribute(new Profile(account));
```

아래처럼 DTO 생성자에서 하나하나 get해서 넣어주고 있습니다. 
지금이야 4건밖에 되지 않으나 많아지면 후에 불편해질 것을 알기 때문에 modelMapper를 사용하고자 합니다.

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

ModelMapper bean 등록
```java
@Bean  
public ModelMapper modelMapper(){  
  return new ModelMapper();  
}
```

**modelMapper.map( source, destination )**
source에 있는 객체를 destination 객체에 넣어주는 역할을 하는 메소드 입니다.
```java
public void updateProfile(Account account, Profile profile) {  
  modelMapper.map(profile,account);  
    accountRepository.save(account);  
}
```


#### ModelMapper 문제점

```java
public void updateNotification(Account account, Notifications notifications) {  
  modelMapper.map(notifications,account);  
    accountRepository.save(account);  
}
```

```java
Account {
...
private boolean studyCreatedByEmail;  
  
private boolean studyCreatedByWeb;  
  
private boolean studyEnrollmentResultByEmail;  
  
private boolean studyEnrollmentResultByWeb;  
  
private boolean studyUpdatedByEmail;  
  
private boolean studyUpdatedByWeb;
}

Notification{  
private boolean studyCreatedByEmail;  
  
private boolean studyCreatedByWeb;  
  
private boolean studyEnrollmentResultByEmail;  
  
private boolean studyEnrollmentResultByWeb;  
  
private boolean studyUpdatedByEmail;  
  
private boolean studyUpdatedByWeb;
}

```

![ModdelMapperProblem1](../images/2022/08/20/ModdelMapperProblem1.png)

```
The destination property com.studyolle.domain.Account.setEmail() matches multiple source property hierarchies: com.studyolle.settings.Notifications.isStudyEnrollmentResultByEmail() com.studyolle.settings.Notifications.isStudyCreatedByEmail() com.studyolle.settings.Notifications.isStudyUpdatedByEmail()
```
Account.setEmail() 에 Email 값을 매핑해줘야 하는데
1. com.studyolle.settings.Notifications.isStudyEnrollmentResultByEmail()
2. com.studyolle.settings.Notifications.isStudyCreatedByEmail()
3. com.studyolle.settings.Notifications.isStudyUpdatedByEmail()
   3개중 뭘 매핑 해줘야 하는지 몰라서 나는 에러입니다.

### ModelMapper configuration

```java
@Bean  
public ModelMapper modelMapper(){  
  ModelMapper modelMapper = new ModelMapper();  
    modelMapper.getConfiguration()  
  .setDestinationNameTokenizer(NameTokenizers.UNDERSCORE)  
  .setSourceNameTokenizer(NameTokenizers.UNDERSCORE)  
  ;  
    return modelMapper;  
}
```