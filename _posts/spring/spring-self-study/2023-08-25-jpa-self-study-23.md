---
title:  "[JPA Self Study] FormBackingObject"

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

# SpringBootJpa 개발 | 3 DAY - 1
```
하다가 막히는 부분이나 알아야 할 사항, 혹은 버전의 문제 등을 적기 위한 간단한 노트이며

이는 공부용 소스이기 때문에 소스에 주석을 달아가며 설명을 적어뒀습니다.
따라서 해당 날짜의 소스를 보며 노트를 참고하는 식으로 봐야합니다.

소스는 깃헙에 올려놓았습니다.
```

### FormBackingObject

Html Form을 채우려고 사용하는 객체

```java
@GetMapping(SETTINGS_NOTIFICATION_URL)  
public String updateNotificationForm(@CurrentUser Account account, Model model){  
  model.addAttribute(new Notification(account));  
    model.addAttribute(account);  
    return SETTINGS_NOTIFICATION_VIEW;  
}
```