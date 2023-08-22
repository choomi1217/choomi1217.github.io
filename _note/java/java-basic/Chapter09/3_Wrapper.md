# wrapper

---

### 🤔 wrapper

객체지향 개념에서 모든 것은 객체로 다루어져야 합니다.

그러나 자바에서는 8개의 기본형을 객체로 다루지않습니다.

때로는기본형(primitive type) 변수도 어쩔 수 없이 객체로 다뤄야하는 경우가 있습니다.

이때 사용 되는 것이 래퍼(wrapper)클래스입니다.

아래는 `Integer` 클래스 내용의 일부입니다.

내부적으로 `int` 를 가지고 있는 모습입니다.

```java
/**
 * The value of the {@code Integer}.
 *
 * @serial
 */
private final int value;

/**
 * Constructs a newly allocated {@code Integer} object that
 * represents the specified {@code int} value.
 *
 * @param   value   the value to be represented by the
 *                  {@code Integer} object.
 */
public Integer(int value) {
    this.value = value;
}
```

---

### 🤔 autoboxing & unboxing

기본형과 참조형간의 연산이 자연스럽게 되는 이유

컴파일시 Integer 객체를 int타입의 값으로 변환 해주는 `intValue( )`를 추가해줍니다.

기본형값을 래퍼클래스의 객체로 자동변환 해주는 것은 `오토박싱(autoboxing)`

반대로 변환하는 것은 `언박싱(unboxing)`