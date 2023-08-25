---
title:  "[JPA Self Study] FailFast"

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

## Fail - Fast란
JavaScript를 이용한 validation-check는 사실 비정상적인 루트를 통해 들어오는 요청엔 소용이 없고 결국 백엔드에서 걸러야 하는데 그럼 왜 귀찮게 굳이 Front에서 걸러서 들어오냐면

> **Fail-Fast 에 의미가 있다.**

공항까지 와서 여권이 없다는 걸 알아채고 " 아, 여권 없네 " 하고 집으로 돌아가는 것
그리고
집을 나오며 " 아, 여권 없네 " 하고 가지고 나오는 것의 차이이다.
