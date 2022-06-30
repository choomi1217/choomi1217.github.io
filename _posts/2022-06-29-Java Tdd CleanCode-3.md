# JUnit으로 테스트코드 짜는 연습!

1. assertThat
	- isEqualTo
```
	@Test
    void replace() {
        String actual = "abc".replace("b", "d");
        assertThat(actual).isEqualTo("adc");
    }
```
	- contains
```
	@Test
    void splitOneTest() {
        String string = "1";
        String[] strArr = string.split(",");
        assertThat(strArr).contains("1");
    }
```	
	- containsExactly
```
	@Test
    void splitTwoTest() {
        String string = "1,2";
        String[] strArr = string.split(",");
        assertThat(strArr).containsExactly("1","2");
    }
```

2. Set 컬렉션을 이용한 JUnit 공부
	- BeforeEach
	- ParameterizedTest
	-CsvSource
```

public class SetTest {
    private Set<Integer> numbers;

    @BeforeEach
    void setUp() {
        numbers = new HashSet<>();
        numbers.add(1);
        numbers.add(1);
        numbers.add(2);
        numbers.add(3);
    }

    @Test
    void setSize() {
        assertThat(numbers.size()).isEqualTo(3);
    }

    @DisplayName("Parameterized를 테스트 해본다")
    @ParameterizedTest
    @ValueSource(ints = {1,2,3})
    void usingParameterized(int input) {
        assertThat(numbers.contains(input)).isTrue();
    }

    @DisplayName("CsvSource를 테스트 해본다")
    @ParameterizedTest
    @CsvSource(value = {"1,2,3:true" , "4,5:false"}, delimiter = ':')
    void usingCsv(String input, boolean expect) {
        String[] testCaseString = input.split(",");
        Set<Integer> testCaseInteger = new HashSet<>();
        for(int i=0; i < testCaseString.length; i++){
            testCaseInteger.add(Integer.valueOf(testCaseString[i]));
        }
        assertThat(numbers.containsAll(testCaseInteger)).isEqualTo(expect);
    }
}

```
