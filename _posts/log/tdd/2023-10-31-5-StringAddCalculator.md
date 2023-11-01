---
title:  "[개발 기록] String Add Calculator"

categories:
  - log
tags:
  - [log]
  
toc: true
toc_sticky: true

breadcrumbs: true

date: 2023-10-31
last_modified_at: 2023-10-31
---

냅다 구현 해보고나서야 설계가 어느정도 완성이 됐습니다.
하지만 큰 프로젝트일수록 냅다 구현하기가 만만치 않을거 같은데.. 
냅다 구현하고 기운이 다 빠져서 리팩토링을 못하게 될까봐 걱정입니다.

# 문자열 덧셈 계산기

### 요구사항
1. 사용자가 특정 문자열를 입력하면 더해져야 합니다.
2. 빈 문자열 또는 null 값을 입력할 경우 0 반환합니다.
3. 숫자 하나를 문자열로 입력할 경우 해당 숫자 반환합니다.
4. 사용자는 커스텀 구분자를 입력할 수 있고 기본 구분자는 ","입니다.
5. 커스텀 구분자는 “//”와 “\n” 문자 사이에 지정할 수 있습니다.
6. 음수를 전달할 경우 RuntimeException이 발생합니다.

### 기능순서도
1. `UserNumbers`에 문자열 인풋
2. `UserNumber`가 문자를 확인
  - empty, null은 0(UserNumber)
  - 한자리면 해당 숫자 (UserNumber)
3. `Delimiter`가 문자열에서 구분자를 거릅니다.
4. `StringDelimiter`가 문자열에서 구분자를 제외하고 파싱합니다.
  - 구분자가 없으면 기본 구분자로 파싱
  - 구분자가 있으면 구분자를 이용해 파싱
5. 문자 유효성 검사는 어디서 해야할까 고민중입니다. 🤔 
6. UserNumbers가 덧셈
  - UserNumber를 List로 들고 있고 이걸 순회하면서 덧셈 
7. 결과 아웃풋 (StringAddCalculator)

### 개발을 하면서 느낀 점
🤔도대체 어떻게 해야 각각의 객체들이 능동적으로 계산을 할 수 있을까 많은 고민을 하는 중입니다.
클래스는 객체의 능동적인 관리자라는 문구에 사로잡혀 있습니다.
사용자의 문자열들이 스스로 계산을 한다는게 아직 어렵네요.. 🥲

### 고쳐야 한다고 느끼는 점
1. delimiter클래스가 능동적으로 걸러주는게 아니고 뭔가 달라고 요청하는 기분이 듭니다.
```
List<String> strings = stringDelimiter.filteredString(origin);
```
2. 구분자와 구분하는 클래스의 분리가 어려웠습니다.
**Delimiter를 구분자를 구분하고 구분자를 가지고 있는 클래스** , **StringDelimiter를 구분자를 이용해 문자열을 걸러내는 클래스**로 구분했습니다. 
하지만 잘 분리 되지 않았습니다. 🥺
또한 자바에서 제공하는 정규표현식 라이브러리를 이용하면 구분자와 걸러진 문자열이 동시에 나오기 때문에 고민을 했습니다.

3. 원래 의도는 사실 UserNumber에서 값을 검증하는 것이었으나 아래처럼 Delimiter와 StringDelimiter에 의해 NPE가 먼저 발생되어 설계한대로 진행하지 못했습니다.
```java
public class StringAddCalculator {

    public static int splitAndSum(String origin) {
        StringDelimiter stringDelimiter = new StringDelimiter(new Delimiter(origin));
        List<String> strings = stringDelimiter.filteredString(origin);
        return UserNumbers.from(strings).sum();
    }
}
```
🥹 원래는 아래처럼 UserNumber에서 행하고 싶은 동작이긴 했습니다..
```java
public class UserNumber {

    private static int parsing(String userString) {
        if (isBlank(userString)) {
            return 0;
        }
        int parseInt = Integer.parseInt(userString);
        if (parseInt < 0) {
            throw new RuntimeException();
        }
        return parseInt;
    }
}
```


### 원하는 이상적인 형태
자바지기님의 이름 짓기에서 영감을 얻었습니다.
UserNumbers가 스스로 계산을 하는 아주 능동적으로 프로세스에 참여하는 객체라고 생각합니다! 🤔
```java
public class StringAddCalculator {
    public static int splitAndSum(String origin) {
        return UserNumbers(origin).sum();
    }
}
```