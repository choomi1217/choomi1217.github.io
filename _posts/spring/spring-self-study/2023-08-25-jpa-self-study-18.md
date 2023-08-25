---
title:  "[JPA Self Study] OpenEntityManager"

categories:
  spring-self-study
tags:
  - [spring-self-study]

toc: true
toc_sticky: true

breadcrumbs: true

date: 2023-08-25
last_modified_at: 2023-08-25
---


# SpringBootJpa 개발 | 10 DAY
```
하다가 막히는 부분이나 알아야 할 사항, 혹은 버전의 문제 등을 적기 위한 간단한 노트이며

이는 공부용 소스이기 때문에 소스에 주석을 달아가며 설명을 적어뒀습니다.
따라서 해당 날짜의 소스를 보며 노트를 참고하는 식으로 봐야합니다.

소스는 깃헙에 올려놓았습니다.
```

### OpenEntityManager

문제가 되는 상황은 이렇습니다.
이메일 토큰 인증을 했음에도
프로필에 가입한 날짜가 보이지 않고 계속 가입을 완료하려면 이메일을 확인하라는 문구가 뜸.
```html
<p th:if="${isOwner || account.emailVerified}">  
    <span style="font-size: 20px;">  
        <i class="fa fa-calendar-o col-1"></i>  
    </span>  
    <span th:if="${isOwner && !account.emailVerified}" class="col-9">  
      <a href="#" th:href="@{'/checkemail?email=' + ${account.email}}">가입을 완료하려면 이메일을 확인하세요.</a>  
    </span>
  <span th:text="${#temporals.format(account.joinedAt, 'yyyy년 M월 가입')}" class="col-9"></span>  
</p>
```
인증 메일 처리하는 곳을 확인

Account 객체의 내용은 바뀌었고
로그인 하면서 시큐리티가 관리해주는 컨텍스트에 넣었으나
정작 DB에는 가입한 날짜를 넣어주지 않고
프로필을 보여주는 곳에선 DB를 조회하고 있음.

JPA에 있는 EntityManager Hibernate의 Session 이 둘을 영속성 컨텍스트라고 합니다.
프로필 처리를 하는 곳의 아래와 같은 db에서 읽어온 객체들을 관리합니다.
```java
Account byNickname = accountRepository.findAllByNickname(nickname);
```

이 안에서 Persistance 상태의 객체들을 관리 하는데 이런 상태의 객체들은 **트랜잭션 범위 안에서 객체의 변경상태를 감지하고 있다가 트랜잭션이 완료될 때 DB에 반영을 합니다.**

현재 컨트롤러까지 트랜잭션으로 잡혀있지 않으므로 생긴 일입니다. ( 서비스까지만 트랜잭셔널 처리함 )
컨트롤러에서 업데이트를 해서 벌이진 일입니다.

기본저긍로 영속성 상태가 유지되는 한 컨트롤러에서 읽던 서비스에서 읽던 레포에서 읽던 그 모든 데이터는 퍼시스턴트 상태입니다.

이 영속성컨텍스트는 뷰를 렌더링하면 끝을 냅니다.

즉, 뷰에서도  DB를 조회하는 것이 가능해진겁니다.
이걸 **OpenEntityManagerInView, OpenSessionInView** 라고 합니다.

뷰에 어떤 데이터를 렌더링 해야할 때,
컨트롤러에서 모델에 냅다 모든 데이터를 넣는것이 아니라 뷰에서 읽는것이 가능해졌습니다. 





