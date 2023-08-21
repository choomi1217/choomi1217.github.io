# Verify에 boolean대신 varchar

- boolean을 쓰면 확장성이 좋지 않다고 생각이 들어서 varchar로 결정
- enum으로 값들을 관리
- enum을 jpa를 이용해 db에 어떻게 저장하는지 고민했습니다.
- `EnumType.ORDINAL` `EnumType.STRING`

EnumType.ORDINAL

- enum을 정수형태로 db에 저장
- 대신 enum의 순서를 바꾸면 안됨

EnumType.STRING

- ORDINAL 보다 많은 공간 차지