# 반성🔥
컨벤션 관련 피드백만 2개 받았다.

# 컨벤션1

```java
class  T {
  상수(static final) 또는 클래스 변수  
  인스턴스 변수  
  생성자  
  팩토리 메소드  
  메소드  
  기본 메소드 (equals, hashCode, toString)  
}
```

# 멤버 변수와 메서드 인자

### 피드백
> 랜덤 생성 객체를 멤버 변수로 관리하기 보다 randomGenerator 객체를 활용하는 메서드에 인자로 받으면 어떨까요?

```java
public class Car {
    private final RandomGenerator randomGenerator;
```

# 인스턴스 재사용
### 피드백
> 랜덤 생성 객체는 한 번만 선언해두고 재사용하면 어떨까요? 
> 스트림에서 매번 `new RandomGenerator(new Random())`하는 부분

```java
public class GameBoard {

    private final InputView inputView = new InputView();
    private final OutputView outputView = new OutputView();

    public void racingGameStart() {
        outputView.printRequestTextInitCar();

        Cars cars = IntStream.range(0, inputView.initRound())
            .mapToObj(i -> new Car(new RandomGenerator(new Random())))
            .collect(Collectors.collectingAndThen(Collectors.toList(), Cars::from));
    }
}
```

# 컨벤션과 역할분담
### 피드백
> -는 출력과 관련된 기능이라고 생각하는데요 
> Score는 자동차 경주 게임의 로직을 담당하는 객체이니 출력을 담당하는 OutputView 클래스로 이동하면 어떨까요? 
> 출력 기호를 변경하는 요구사항이 생긴다면 OutputView 클래스만 변경하면 되니 수정에도 좋을 것 같아요

[Google Conventions](https://google.github.io/styleguide/javaguide.html#s4.8.7-modifiers)
Worng
```java
public final static String MOVE_SCORE = "-";
```
Right
```java
public static final String MOVE_SCORE = "-";
```
### 답장
이 부분, 클래스의 역할에 대해서 고민이 많았습니다.

Score에서 MoveStatus를 가지고 있고, 이걸 결과를 출력하게 된다면
MOVE, STOP, MOVE, MOVE, MOVE 이런 식으로 나오게 될텐데..
이걸 "----" 이렇게 변환하는게 OutputView의 역할일까 Score의 역할일까 고민했습니다. 🤔

그래서 아예 MoveStatus Enum 쪽으로 옮겨가는 건 어떨가 싶습니다.

그렇게 되면 OutputView가 출력 외에 Score의 MoveStatus를 판단하는 일을 맡지 않아도 되고,
출력 기호 변경하는 요구사항이 생길 때 MoveStatus만 변경하면 된다고 생각합니다! 😲

# 테스트 코드
### 피드백
> 프로덕션 코드가 의도치 않게 변경되었을 때 테스트는 실패해야하는데요 
> 기능은 의도하지 않는대로 동작하더라도 예외만 발생하지 않는다면 테스트는 통과한다면 테스트를 신뢰하기 어려울 것 같아요 
> Round#race 메서드에서 진행 결과를 반환하고 이를 검증하면 어떨까요?

```java
@DisplayName("라운드가 정상 진행")
@Test
void a(){
    Round round = new Round(movingCars);
    ScoreBoard scoreBoard = new ScoreBoard();
    assertThatNoException().isThrownBy(() -> round.race(scoreBoard));
}
```
### 답장
`round.race()`를 하고 난 뒤에 무언가를 반환을 해야 한다는 점에서 고민하고 있습니다.
TDD 강의 내의 '이름 짓기' 라는 글의 내용에 '빌더', '조정자' 부분에서
빌더는 무언가를 만들어서 돌려주고
조정자는 클래스의 데이터를 변경하는 대신 void 타입입니다.
그래서 저는 `round.race()`란 메소드를 '조정자'라고 판단 했습니다!

이 부분의 테스트에 대해서는 용주님 말씀대로 큰 문제라고 생각하고 있습니다!
테스트 할 다른 방법을 찾아보겠습니다!

[TDD - 이름 짓기 문서 ](https://edu.nextstep.camp/s/twbNYuxs/ls/yGGywvSB)

# 테스트명 변경
### 피드백
> DisplayName이 있더라도 테스트명은 작성해 주시는게 좋을 것 같아요
> 테스트 메서드명을 작성하기 어려우시다면 테스트하고자 하는 메서드의 이름이나 결과로 작성하시면 어떨까요?

# View와 Model
### 피드백
> 출력 기호를 변경하는 요구사항이라고 코멘트 드려서 제 의도가 정확히 전달이 되지 못했네요 😢
> View와 관련된 변경 요구사항이 있다면 View 영역의 수정만으로 해결하는 것이 좋다는 의견입니다
> 현재 구조에서는 `-`를 변경하기 위해서 Model 영역에 해당하는 MoveStatus 를 찾아서 수정해야하는 과정이 쉽진 않을 것 같아요


# 빌더와 조정자
### 피드백
> Round 객체의 책임에 대해 고민해볼 수 있는 부분인 것 같아요
> Round는 자동차들을 1회 전진을 시도하는 역할을 하고 있는데요
> 그렇다면 Round는 1회만 전진을 시도할 수 있다는 제약을 추가할 수 있을 것 같아요
> Round가 race의 결과를 알 수 있다면 2회 이상 시도하는 것을 방지할 수도 있고
> 빌더 메서드를 통해 레이스의 결과나 라운드가 진행되었음을 반환하는 기능을 제공해도 좋을 것 같은데 어떻게 생각하시나요?

### 답변
위 글에선 아래처럼 동사로 된 메서드(조정자)엔 반환값이 없어야 하고 만약 동사로 된 메서드에 반환값이 었어야만 한다면 
클래스를 추가해 메서드로 분리하라는 내용의 글이 있어서 클래스를 하나 더 추가해서 PR 올리겠습니다!
감사합니다! 🙇

잘못된 예시
```text
int save(String content);
boolean put(String key, Float value);
float speed(float val);
```

리팩토링
앞의 put 메서드는 PutOperation과 같은 클래스를 추가해 save() 메서드와 success() 메서드로 분리
앞의 speed 메서드는 SaveSpeed와 같은 클래스를 추가해 speed를 저장하는 메서드와 이전 값을 반환하는 메서드로 분리


