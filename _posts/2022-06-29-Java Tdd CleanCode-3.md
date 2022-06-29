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

    @ParameterizedTest
    @ValueSource(ints = {1,2,3})
    void contains(int input) {
        assertThat(numbers.contains(input)).isTrue();
    }

    @Test
    @ParameterizedTest
    @CsvSource(value = {"1,2,3:true" , "4,5:false"}, delimiter = ':')
    void name(String input, String expect) {

        assertThat(numbers.contains(Integer.parseInt(input))).isEqualTo(Boolean.getBoolean(expect));
    }
}
```
