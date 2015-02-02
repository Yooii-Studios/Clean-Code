## 목차 ##
- 리턴코드 대신 Exceptions을 사용하라
- Try-Catch-Finally문을 먼저 써라
- Unchecked Exceptions를 사용하라
- Exceptions로 문맥을 재공하라
- 사용에 맞게 Exceptions 클래스를 선언하라
- 정상적인 상황을 정의하라(Default값을 설정하라)
- Null을 리턴하지 마라
- Null을 넘기지 마라
- 결론

======================================================

> 오류 처리는 중요하다. 하지만 로직을 헷갈리게 만드는 오류처리는 나쁘다.
(원문 : Error handling is important, but if it obscures logic, it's wrong.)

## 제목 ##
#### 부제목 ####
내용
```java
// comment
if (someCondition) {
  statement;
}
```
