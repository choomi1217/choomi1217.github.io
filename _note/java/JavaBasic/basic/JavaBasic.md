자바에서 사용되는 힙, 스택, 메소드 영역 등은 컴퓨터의 주 메모리(RAM)에 위치합니다.
JVM (Java Virtual Machine)은 프로그램을 실행할 때 컴퓨터의 주 메모리를 사용하여 아래와 같은 영역을 생성하고 관리합니다. 

1. 힙 영역
   - 객체와 JRE 클래스들이 위치 
2. 스택 영역 ( 스택 프레임이 쌓임 )
   - 메소드 호출과 지역 변수가 위치 
3. 메소드 영역
   - JVM이 읽어들인 각 클래스와 인터페이스에 대한 런타임 상수 풀, 필드와 메소드 데이터 등이 위치합니다.


간단한 예제
```java
// App
public class App {

    public static void main(String[] args) throws IOException, ClassNotFoundException {
        Setting setting1 = new Setting();
        Setting setting2 = new Setting();

        System.out.println(setting1 == setting2); // false
    }
}
// Setting
public calss Setting{

}
```

1. 클래스 App이 로드되면서 JVM은 메소드 영역에 클래스에 대한 정보를 로드합니다. 여기에는 App 클래스의 메타데이터가 포함되어 있습니다.
2. App 클래스와 Setting 클래스 모두, 프로그램이 실행될 때 JVM에 의해 메소드 영역에 로드됩니다.
2. main 메소드가 호출됩니다. 이때 스택 메모리 영역에 main 메소드를 위한 새로운 스택 프레임이 생성되며, 이 스택 프레임에는 args라는 지역 변수를 위한 공간이 할당됩니다.
  - JVM은 프로그램 실행을 위해 먼저 main 메소드를 포함하는 클래스를 로드합니다
  - 예를 들어, java App 명령어를 실행하면 JVM은 App 클래스를 로드하고 main 메소드를 찾아 프로그램을 시작합니다. 만약 main 메소드가 없다면, JVM은 오류를 반환하게 됩니다.
3. Setting 클래스의 인스턴스를 생성하기 위해 new Setting()가 호출됩니다. 이때, 힙 메모리 영역에 Setting 객체가 생성되고, 이 객체를 가리키는 참조가 setting1 변수에 할당됩니다. setting1 변수는 main 메소드의 스택 프레임에 위치합니다.
4. main 메소드가 종료되면, 이에 해당하는 스택 프레임이 스택에서 제거(pop)됩니다. 이 스택 프레임에 있던 setting1과 setting2 레퍼런스 변수도 메모리에서 해제됩니다. 더 이상 Setting 객체를 가리키는 참조가 없게 되므로, 이 객체들은 가비지 컬렉션의 대상이 될 수 있습니다.


![메모리의 흐름](image/javaBasic.png)
