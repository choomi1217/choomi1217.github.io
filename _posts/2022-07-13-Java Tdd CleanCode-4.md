
### TDD하면서 지켜야 할 원칙

1. 함수는 '작게'
2. 함수는 한 가지만 해야 한다.
3. 함수 내 모든 문장이 동일한 추상화 수준.
4. 이름이 길어도 괜찮다.
5. 이름을 붙일 때는 일관성이 있어야 한다.
6. 함수에서 이상적인 인수 개수는 0개(무항)이다. 
7. 인수가 2 ~ 3개 필요한 경우가 생긴다면 인수를 독자적인 클래스를 생성할 수 있는지 검토해 본다.
8. side effect를 만들지 마라.
9. 함수는 뭔가를 수행하거나 답하거나 둘 중 하나만 해야 한다. 
10. 개체 상태를 변경하거나 아니면 개체 정보를 반환하거나 둘 중 하나다.
11. 오류 코드보다 예외를 사용하라.
12. 반복하지 마라.


## 솔직히 하면서 많은 벽을 느꼈고 한번에 배부를 수 없다는걸 일주일동안 삽질을 하면서 느꼈습니다.

if문을 2000줄씩 만들어낸 차장님의 코드를 보며 클린하게 누구든 보면 딱-! 하고 감이 오게끔 코딩하고 싶었습니다.
그런 마음으로 시작했으나 TDD를 짜야한다 클래스를 작게 짜야한다는 강박에 그만 로직을 잊고 클래스만 잔뜩 만들고 있었습니다.
아직 처음부터 완벽한 클래스를 분리하며 코드를 짜는게 익숙하지 않으니 리팩토링을 한단 마인드로 우선 돌아가는 코드를 짰습니다.

앞으로 리팩토링하며 클래스도 쪼개보고 더 좋은 로직이 있다면 사용해보도록 하겠습니다. 

### 아 모르게따, 냅다 구현

```
package StringCalculator2;

import java.util.ArrayList;
import java.util.Arrays;

public class main {
    public static void main(String[] args) {
        
        ArrayList<String> operators = new ArrayList<String>(Arrays.asList("+","-","*","/"));
        String[] splits = "1 + 2 - 3 + 1 * 4 / 4".split(" ");

        int result = Integer.parseInt(splits[0]);

        for (int i=0; i<splits.length; i++){
            if(operators.contains(splits[i])){
                if(splits[i].equals("+")){
                    result += Integer.parseInt(splits[i+1]);
                }else if(splits[i].equals("-")){
                    result -= Integer.parseInt(splits[i+1]);
                }else if(splits[i].equals("*")){
                    result *= Integer.parseInt(splits[i+1]);
                }else if(splits[i].equals("/")){
                    result /= Integer.parseInt(splits[i+1]);
                }
            }else {
            }
        }

        System.out.println(result);

    }
}

```

### 첫번째 문제점

- 메인메소드가 하는 일이 너무 많다
	- 메인메소드가 하는 일
		1. 문자열 분리
		2. 해당 문자열의 연산자 구분
		3. 연산

### 첫번째 목표

- 메소드 분리하기!



