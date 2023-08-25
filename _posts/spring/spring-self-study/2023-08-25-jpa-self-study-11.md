---
title:  "[JPA Self Study] Fetch Type EAGER"

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

# SpringBootJpa 개발 | 5 DAY - 2
```
하다가 막히는 부분이나 알아야 할 사항, 혹은 버전의 문제 등을 적기 위한 간단한 노트이며

이는 공부용 소스이기 때문에 소스에 주석을 달아가며 설명을 적어뒀습니다.
따라서 해당 날짜의 소스를 보며 노트를 참고하는 식으로 봐야합니다.

소스는 깃헙에 올려놓았습니다.
```


## What's the Fetch Type EAGER in Entity
코드를 다시 살펴보는 도중에
```
@Lob @Basic(fetch = FetchType.EAGER)  
private String profileImage;
```

우선 현재 강의를 넘길 수준으로만 간단하게 알고 가고 후에 JPA에 대해 공부 할 날을 잡아보겠습니다.

### FetchType
	: 관계를 맺고 있는 두개 이상의 테이블에 대해 SELECT 방법을 정의

#### Member Table
| ID (PK) | NAME |
|--|--|
| oomi | 앵미 |
| hong | 길동 |
| ggamzi | 깜지 |

#### Order Table
| NUMBER (PK) | MEMBER_ID (FK) |
|--|--|
| 1 | oomi |
| 2 | ggamzi  |
| 3 | ggamzi |

- EAGER
  - Member를 조회하고 Member에 Order에 대한 컬렉션을 Entity로 가지며 FetchType이 EAGER라면 다시 조회하는 쿼리를 날리지 않고도 조인을 통해 Order에 대한 DB정보도 가져 올 수 있습니다.


[JPA의 Eager와 Lazy에 대한 참고 영상](https://www.youtube.com/watch?v=ucuVbL-tsUY)
