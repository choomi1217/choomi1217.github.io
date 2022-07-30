# JAVA Stream

다양한 데이터 소스(Collection, Array)을 표준화된 방법(Collection Framework)으로 다루기 위한 것
List와 Set 그리고 Map이 사용법이 달라서 Collection Framework는 반쪽자리이지만
#### JDK 1.8부터 Stream의 등장으로 정말 표준화된 방법을 제공 할 수 있게 됨

Collection ( List, Set, Map ), Array 로부터 Stream이라는 것을 만들 수 있음.
일단 Stream을 만들고 나면 같은 작업들을 할 수 있게 됨.


|DataSource|→|Stream|→|중간 연산자|→|최종 연산자|→|결과💘|
|--|--|--|--|--|--|--|--|--|
|Collections|↗|||n번 반복||1번만 수행|
|Arrays|↑|

---
#### Stream의 흐름을 예제로 보기

```JAVA
stringStream.distinct().limit(5).sorted().forEach(System.out::println);
```

|DataSource|→|Stream|→|중간 연산자|→|최종 연산자|→|결과💘|
|--|--|--|--|--|--|--|--|--|
|||stringStream|→|distinct()|→|forEach|
|||||limit(5)|
|||||sorted()|

- 중간 연산자의 결과 값이  Stream이기 때문에 계속 수행이 가능합니다.
- 최종 연산은 Stream의 요소를 소모 ( 하나씩 꺼냄 ) 하므로 최종적으로 한 번만 사용합니다.

#### Stream을 만드는 여러가지 방법

``` JAVA
List<Integer> list = Arrays.asList(1,2,3,4,5);  
Stream<Integer> integerStream = list.stream(); // Collection으로부터 Stream 만들기  
Stream<String> stringStream = Stream.of("a","b","c"); //Stream의 배열 생성 | return Arrays.stream(values);Stream<Integer> evenStream = Stream.iterate(0, n->n+2); // 람다식
Stream<Double> randomStream = Stream.generate(Math::random); // 람다식
IntStream intStream = new Random().ints(5); //난수 스트림
```

[Stream 사용을 연습해본 Git](https://github.com/choomi1217/javaStudyWithJunit.git)

### Stream의 특징

1. 스트림은 데이터 소스로부터 데이터를 읽기만 하고 변경하지 않습니다.
```JAVA
List<Integer> list = Arrays.asList(3,1,5,6,2,4);  
List<Integer> sortedList = list.stream().sorted().collect(Collectors.toList());  
System.out.println(list);  
System.out.println(sortedList);
```
2. 스트림은 Iterator처럼 일회용이다.
	- 최종 연산을 하면 Stream의 요소를 소모하는 것이다.
	- 최종 연산 후엔 Stream을 닫는다.
```JAVA
List<Integer> list = Arrays.asList(1,2,3,4,5);  
System.out.println(list.stream().count());  
list.stream().forEach(s -> System.out.println(" ** "+s+" ** "));
```
3. 스트림은 지연된 연산을 합니다.
	- 최종 연산 전까진 중간 연산이 수행되지 않는다.
	- 지연된 연산
```JAVA
// 1 ~ 45 범위의 난수를 발생시키는 무한 스트림  
IntStream randomeStream = new Random().ints(1,45);  
// 중간연산과 최종연산까지 마친 무한(?)스트림 -> limit도 걸고 정렬도 했는데 어떻게 무한이라고 할 수 있는가!!
randomeStream.distinct().limit(6).sorted()  
    .forEach(i->System.out.print( "," + i));
```
***어떻게 무한 스트림이라면서 정렬이 되고 길이 제한을 할 수 있는가?***
중간연산자가 호출 되었을 때 바로 수행하는 것이 아니라 해당 중간연산자를 수행 해야 함을 체크만 해둡니다.  
그러다가 **stream의 최종연산자를 호출한 시점에 한꺼번에 수행합니다.**

4. 스트림은 작업을 내부 반복으로 처리합니다.
	- 성능은 비효율적이지만 코드가 간결해집니다. 
```JAVA
List<Integer> integerList = Arrays.asList(1,2,3,4,5);  
for (int i: integerList) {  
    System.out.print(i + ",");  
}  
System.out.println("\n");  
integerList.stream().forEach(i -> System.out.print(i + "," ));
```

5. 스트림의 작업을 병렬로 처리 할 수 있습니다.
	- parallel과 sequential에 대한 처리 방식은 참고했습니다.
[geeksforgeeks_참고](https://www.geeksforgeeks.org/parallel-vs-sequential-stream-in-java/)
```JAVA
Stream<String> stringStream = Stream.of("A","B","C");  
int parallelSum = stringStream.parallel().mapToInt(s->s.length()).sum();  
assertThat(parallelSum).isEqualTo(3);  
int sequentialSum = stringStream.sequential().mapToInt(s->s.length()).sum();  
assertThat(sequentialSum).isEqualTo(3);
```
- Parallel

| core1 | core2 | core3 | core4 | core5 |
|--|--|--|--|--|
| thread1 | thread2 | thread3 | thread4 | thread5 |
|  | thread6 | thread7 |  |  |
|  | thread8 |  |  |  |
|  |  |  |  |  |
|  |  |  |  |  |

- Sequential

| core1 | core2 | core3 | core4 | core5 |
|--|--|--|--|--|
| thread1 |  |  |  |  |
| thread2 |  |  |  |  |
| thread3 |  |  |  |  |
| thread4 |  |  |  |  |
| thread5 |  |  |  |  |

6. 기본형 스트림 제공
	- AutoBoxing, Unboxing의 비효율성이 제거 됩니다.
	- 숫자 관련된 메서드들을 많이 제공합니다. 
```JAVA

```



