---
title:  "[JPA Self Study] FlashAttribute"

categories:
  - spring
tags:
  - [spring-self-study]

toc: true
toc_sticky: true

breadcrumbs: true

date: 2023-08-25
last_modified_at: 2023-08-25
---


# SpringBootJpa 개발 | 12DAY - 1
```
하다가 막히는 부분이나 알아야 할 사항, 혹은 버전의 문제 등을 적기 위한 간단한 노트이며

이는 공부용 소스이기 때문에 소스에 주석을 달아가며 설명을 적어뒀습니다.
따라서 해당 날짜의 소스를 보며 노트를 참고하는 식으로 봐야합니다.

소스는 깃헙에 올려놓았습니다.
```

## FlashAttribute

FlashAttribute 는 리다이렉트 한 곳에서 잠깐 사용해야하는 메세지를 담아 보낼때 유용합니다.

```java
@PostMapping(SETTINGS_PROFILE_URL)  
public String updateProfile(@CurrentUser Account account, @Valid @ModelAttribute Profile profile, Errors errors, Model model, RedirectAttributes redirectAttributes){  
  if(errors.hasErrors()){  
  model.addAttribute(account);  
        return SETTINGS_PROFILE_VIEW;  
    }  
  accountService.updateProfile(account,profile);  
    redirectAttributes.addFlashAttribute("message","프로필을 수정하셨습니다.");  
    return "redirect:" + SETTINGS_PROFILE_URL;  
}
```

