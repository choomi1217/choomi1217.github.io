---
title:  "[JPA Self Study] ThymeLeaf Fragment"

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


# SpringBootJpa 개발 | 7 DAY - 1
```
하다가 막히는 부분이나 알아야 할 사항, 혹은 버전의 문제 등을 적기 위한 간단한 노트이며

이는 공부용 소스이기 때문에 소스에 주석을 달아가며 설명을 적어뒀습니다.
따라서 해당 날짜의 소스를 보며 노트를 참고하는 식으로 봐야합니다.

소스는 깃헙에 올려놓았습니다.
```

## ThymeLeaf Fragment
html에 중복된 header, footer, navbar 등등을 제거하기 위해
중복된 코드들을  따로 html파일로 생성
```
<head th:fragment="head">  
 <meta charset="UTF-8">  
 <title>StudyOlle</title>  
 <link rel="stylesheet" href="/node_modules/bootstrap/dist/css/bootstrap.min.css" />  
 <style>  .container {  
      max-width: 100%;  
    }  
  </style>  
</head>
```
후 `th:replace` 를 이용해 주입
```
<head th:replace="fragments.html :: head"></head>
```

## Jdenticon & FontAwesome
```html
<link rel="stylesheet" href="/node_modules/font-awesome/css/font-awesome.min.css"/>  
<link rel="stylesheet" href="/node_modules/jdenticon/dist/jdenticon.min.js"/>
```
```html
<svg th:data-jdenticon-value="${#authentication.name}" width="24" height="24" class="rounded border bg-light">  
  프로필  
</svg>
<i class="fa fa-bell-o"></i>
```

## Thymeleaf ?
해당 값이 Null일지도 모른다는 걸 표시하기 위해
``` account?.emailVerified ``` 를 사용했습니다.
account는 Null일 수도 있고 Null이 아니면 account의 emailVerified를 가져옵니다.
```html
<div class="alert alert-warnning" role="alert" th:if="${account!=null && !acoount.emailVerified}">
```
```html
<div class="alert alert-warnning" role="alert" th:if="${account!=null && !acoount?.emailVerified}">
```