
# SpringBootJpa 개발 | 11 DAY
```
하다가 막히는 부분이나 알아야 할 사항, 혹은 버전의 문제 등을 적기 위한 간단한 노트이며

이는 공부용 소스이기 때문에 소스에 주석을 달아가며 설명을 적어뒀습니다.
따라서 해당 날짜의 소스를 보며 노트를 참고하는 식으로 봐야합니다.

소스는 깃헙에 올려놓았습니다.
```

### Field injection is not recommended

뜬금 없지만 Bean 생성 과정에서 인텔리제이가 ` Field injection is not recommended `를 띄우길래 필드주입을 왜 권장하지 않는 건가 하고 찾아본 글입니다.

글이 너무 좋아서 소장하고 싶어서 올립니다.
[객체 주입에 대한 이야기](https://yaboong.github.io/spring/2019/08/29/why-field-injection-is-bad/)

요약하자면 **생성자 주입을 권장한다.** 입니다

### RedirectAttributes
요청 처리 후 페이지를 리다이렉트 하면서 메세지를 담기 위해 사용했습니다.
리다이렉트한 곳에서 한번 쓰고 말 데이터를 담아서 보내기 좋습니다.
