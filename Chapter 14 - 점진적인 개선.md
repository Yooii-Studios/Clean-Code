## 목차  
- [Intro](#0)
- [Args 구현](#1)
  - [어떻게 짰느냐고?](#1-1)
- [Args: 1차 초안](#2)
  - [그래서 멈췄다](#2-1)
  - [점진적으로 개선하다](#2-2)
- [String 인수](#3)
- [최종 코드](#4)
- [결론](#5)

<a name="0"></a>
## Intro
이 장은 점진적인 개선을 보여주는 사례 연구다. 우선, 출발은 좋았으나 확장성이 부족했던 모듈을 소개한다. 그런 다음, 모듈을 개선하고 정리하는 단계를 살펴본다. 

프로그램을 짜다 보면 종종 명령행 인수의 구문을 분석할 필요가 생긴다. 편리한 유틸리티가 없다면 main 함수로 넘어오는 문자열 배열을 직접 분석하게 된다. 여러 가지 훌륭한 유틸리티가 있지만 내 사정에 딱 맞는 유틸리티가 없다면? 물론 직접 짜겠다고 결심한다. 새로 짠 유틸리티를 Args라 부르겠다. 
 
Args는 사용법이 간단하다. Args 생성자에 (입력으로 들어온) 인수 문자열과 형식 문자열을 넘겨 Args 인스턴스를 생성한 후 Args 인스턴스에다 인수 값을 질의한다. 다음 간단한 예를 살펴보자. 
 
##### 목록 14-1 간단한 Args 사용법
```java
public static void main(String[] args) {
  try {
    Args arg = new Args("l,p#,d*", args);
    boolean logging = arg.getBoolean('l');
    int port = arg.getInt('p');
    String directory = arg.getString('d');
    executeApplication(logging, port, directory);
  } catch (ArgsException e) {
    System.out.print("Argument error: %s\n", e.errorMessage());
  }
}
```
매개변수 두 개로 Args 클래스의 클래스의 인스턴스를 만든다. 첫째 매개변수는 형식 또는 스키마를 지정한다. "l,p#,d*"은 명령행 인수 세 개를 정의한다. 첫 번째 -l은 부울 인수다. 두 번째 -p는 정수 인수다. 세 번째 -d는 문자열 인수다. 두 번째 매개변수는 main으로 넘어온 명령행 인수 배열 자체다.

 ArgsException이 발생하지 않는다면 명령행 인수의 구문을 성공적으로 분석했으며 Args 인스턴스에 질의를 던져도 좋다는 말이다. 인수 값을 가져오기 위해 get~() 등의 메서드를 사용한다. 

<a name="1"></a>
## Args 구현
목록 14-2는 Args 클래스다. 아주 주의 깊게 읽어보기 바란다. 스타일과 구조에 신경을 썼으므로 흉내 낼 가치가 있다고 믿는다. 

이름을 붙인 방법, 함수 크기, 코드 형식에 각별히 주목한다. 노련한 프로그래머라면 여기저기 자잘한 구조나 스타일이 거슬릴지 모르지만 전반적으로 깔끔한 구조에 잘 짜인 프로그램으로 여겨주면 좋겠다. 

##### 목록 14-2 Args.java
```java
package com.objectmentor.utilities.args;

import static com.objectmentor.utilities.args.ArgsException.ErrorCode.*; 
import java.util.*;

public class Args {
  private Map<Character, ArgumentMarshaler> marshalers;
  private Set<Character> argsFound;
  private ListIterator<String> currentArgument;
  
  public Args(String schema, String[] args) throws ArgsException { 
    marshalers = new HashMap<Character, ArgumentMarshaler>(); 
    argsFound = new HashSet<Character>();
    
    parseSchema(schema);
    parseArgumentStrings(Arrays.asList(args)); 
  }
  
  private void parseSchema(String schema) throws ArgsException { 
    for (String element : schema.split(","))
      if (element.length() > 0) 
        parseSchemaElement(element.trim());
  }
  
  private void parseSchemaElement(String element) throws ArgsException { char elementId = element.charAt(0);
    String elementTail = element.substring(1); validateSchemaElementId(elementId);
    if (elementTail.length() == 0)
      marshalers.put(elementId, new BooleanArgumentMarshaler());
    else if (elementTail.equals("*")) 
      marshalers.put(elementId, new StringArgumentMarshaler());
    else if (elementTail.equals("#"))
      marshalers.put(elementId, new IntegerArgumentMarshaler());
    else if (elementTail.equals("##")) 
      marshalers.put(elementId, new DoubleArgumentMarshaler());
    else if (elementTail.equals("[*]"))
      marshalers.put(elementId, new StringArrayArgumentMarshaler());
    else
      throw new ArgsException(INVALID_ARGUMENT_FORMAT, elementId, elementTail);
  }
  
  private void validateSchemaElementId(char elementId) throws ArgsException { 
    if (!Character.isLetter(elementId))
      throw new ArgsException(INVALID_ARGUMENT_NAME, elementId, null); 
  }
  
  private void parseArgumentStrings(List<String> argsList) throws ArgsException {
    for (currentArgument = argsList.listIterator(); currentArgument.hasNext();) {
      String argString = currentArgument.next(); 
      if (argString.startsWith("-")) {
        parseArgumentCharacters(argString.substring(1)); 
      } else {
        currentArgument.previous();
        break; 
      }
    } 
  }
  
  private void parseArgumentCharacters(String argChars) throws ArgsException { 
    for (int i = 0; i < argChars.length(); i++)
      parseArgumentCharacter(argChars.charAt(i)); 
  }
  
  private void parseArgumentCharacter(char argChar) throws ArgsException { 
    ArgumentMarshaler m = marshalers.get(argChar);
    if (m == null) {
      throw new ArgsException(UNEXPECTED_ARGUMENT, argChar, null); 
    } else {
      argsFound.add(argChar); 
      try {
        m.set(currentArgument); 
      } catch (ArgsException e) {
        e.setErrorArgumentId(argChar);
        throw e; 
      }
    } 
  }
  
  public boolean has(char arg) { 
    return argsFound.contains(arg);
  }
  
  public int nextArgument() {
    return currentArgument.nextIndex();
  }
  
  public boolean getBoolean(char arg) {
    return BooleanArgumentMarshaler.getValue(marshalers.get(arg));
  }
  
  public String getString(char arg) {
    return StringArgumentMarshaler.getValue(marshalers.get(arg));
  }
  
  public int getInt(char arg) {
    return IntegerArgumentMarshaler.getValue(marshalers.get(arg));
  }
  
  public double getDouble(char arg) {
    return DoubleArgumentMarshaler.getValue(marshalers.get(arg));
  }
  
  public String[] getStringArray(char arg) {
    return StringArrayArgumentMarshaler.getValue(marshalers.get(arg));
  } 
}
```

여기저기 뒤적일 필요 없이 위에서 아래로 코드가 읽힌다는 사실에 주목한다. 한 가지 먼저 읽어볼 코드가 있다면 ArgumentMrashaler 정의인데, 목록 14-3에서 14-6까지는 ArgumentMarshaler 인터페이스와 파생 클래스다. 

##### 목록 14-3 ArgumentMarshaler.java
```java
public interface ArgumentMarshaler {
  void set(Iterator<String> currentArgument) throws ArgsException;
}
```

##### 목록 14-4 BooleanArgumentMarshaler.java
```java
public class BooleanArgumentMarshaler implements ArgumentMarshaler { 
  private boolean booleanValue = false;
  
  public void set(Iterator<String> currentArgument) throws ArgsException { 
    booleanValue = true;
  }
  
  public static boolean getValue(ArgumentMarshaler am) {
    if (am != null && am instanceof BooleanArgumentMarshaler)
      return ((BooleanArgumentMarshaler) am).booleanValue; 
    else
      return false; 
  }
}
```

##### 목록 14-5 StringArgumentMarshaler.java
```java
import static com.objectmentor.utilities.args.ArgsException.ErrorCode.*;

public class StringArgumentMarshaler implements ArgumentMarshaler { 
  private String stringValue = "";
  
  public void set(Iterator<String> currentArgument) throws ArgsException { 
    try {
      stringValue = currentArgument.next(); 
    } catch (NoSuchElementException e) {
      throw new ArgsException(MISSING_STRING); 
    }
  }
  
  public static String getValue(ArgumentMarshaler am) {
    if (am != null && am instanceof StringArgumentMarshaler)
      return ((StringArgumentMarshaler) am).stringValue; 
    else
      return ""; 
  }
}
```

##### 목록 14-6 IntegerArgumentMarshaler.java
```java
import static com.objectmentor.utilities.args.ArgsException.ErrorCode.*;

public class IntegerArgumentMarshaler implements ArgumentMarshaler { 
  private int intValue = 0;
  
  public void set(Iterator<String> currentArgument) throws ArgsException { 
    String parameter = null;
    try {
      parameter = currentArgument.next();
      intValue = Integer.parseInt(parameter);
    } catch (NoSuchElementException e) {
      throw new ArgsException(MISSING_INTEGER);
    } catch (NumberFormatException e) {
      throw new ArgsException(INVALID_INTEGER, parameter); 
    }
  }
  
  public static int getValue(ArgumentMarshaler am) {
    if (am != null && am instanceof IntegerArgumentMarshaler)
      return ((IntegerArgumentMarshaler) am).intValue; 
    else
    return 0; 
  }
}
```

나머지 DoubleArgumentMarshaler와 StringArrayArgumentMarshaler는 다른 파생 클래스와 똑같은 패턴이므로 코드를 생략한다. 

한 가지가 눈에 거슬릴지 모르겠다. 바로 오류 코드 상수를 정의하는 부분이다. 목록 14-7을 살펴보자. 

##### 목록 14-7 ArgsException.java
```java
```

이처럼 단순한 개념을 구현하는데 코드가 너무 많이 필요해 놀랄지도 모르겠다. 우선적인 이유는 장황한 언어인 자바를 사용해서인데, 정적 타입 언어라서 타입 시스템을 만족하려면 많은 단어가 필요하다. 

하지만 이름을 붙인 방법, 함수 크기, 코드 형식에 주목을 해 본다면 전반적으로 깔끔한 구조에 잘 짜인 프로그램으로 여겨주면 좋겠다. 

예를 들어, 날짜 인수나 복소수 인수 등 새로운 인수 유형을 추가하는 방법이 명백하다. 고칠 코드도 별로 없다. 간단히 설명하자면, ArgumentMarshaler에서 새 클래스를 파생해 getXXX 함수를 추가한 후 parseSchemaElement 함수에 새 case 문만 추가하면 끝이다. 필요하다면 새 ArgsException.ErrorCode를 만들고 새 오류 메시지를 추가한다. 

<a name="1-1"></a>
#### 어떻게 짰느냐고?
일단 진정하기 바란다. 나는 위 프로그램을 처음부터 저렇게 구현하지 않았다. 더욱 중요하게는 여러분이 깨끗하고 우아한 프로그램을 한 방에 뚝딱 내놓으리라 기대하지 않는다. 지난 수십여 년 동안 쌓아온 경험에서 얻은 교훈이라면, **프로그래밍은 과학보다 공예(craft)에 가깝다는 사실이다. 깨끗한 코드를 짜려면 먼저 지저분한 코드를 짠 뒤에 정리해야 한다는 의미이다.**  

처음 듣는 이야기가 아니라고 생각한다. 초등학교 시절 선생님들도 작문을 할 때도 초안부터 쓰라고 교육을 하셨다. 깔끔한 작품을 내놓으려면 단계적으로 개선해야 한다고 가르치려 애쓰셨다.

대다수의 신참 프로그래머는 (대다수 초딩과 마찬가지로) 이 충고를 충실히 따르지 않는다. 그들은 무조건 돌아가는 프로그램을 목표로 잡는다. 일단 프로그램이 '돌아가면' 다음 업무로 넘어간다. '돌아가는' 프로그램은 그 상태가 어떻든 그대로 버려둔다. **경험이 풍부한 전문 프로그래머라면 이런 행동이 전문가로서 자살 행위라는 사실을 잘 안다.** 

<a name="2"></a>
## Args: 1차 초안
목록 14-8은 내가 맨 처음 짰던 Args 클래스다. 코드는 '돌아가지만' 엉망이다.

##### 목록 14-8 Args.java(1차 초안)
```java

```
이처럼 지저분한 코드를 보고 처음 든 생각이 "저자가 그냥 버려두지 않아서 진짜 다행이야!"이기 바란다. 만약 그렇다면 그렇다면 자신이 대충 짜서 남겨둔 코드를 남들이 어떻게 느낄지 생각하기 바란다. 

처음부터 지저분한 코드를 짜려는 생각은 없었다. 실제로도 코드를 어느 정도 손보려고 애썼다. 함수 이름이나 변수 이름을 선택한 방식, 어설프지만 나름대로 구조가 있다는 사실 등이 내 노력의 증거다. 하지만 어느 순간 프로그램은 내 손을 벗어났다. 첫 버전이던 Boolean 인수만 지원하던 초기 버전에서 String과 Integer 인수 유형을 추가하면서 부터 재앙이 시작됐다. 

주) 관련 코드들 역시 있지만 내용이 너무 많기에 내용 및 코드를 생략

<a name="2-1"></a>
#### 그래서 멈췄다
추가할 인수 유형이 적어도 두 개는 더 있었는데 그러면 코드가 훨씬 더 나빠지리라는 사실이 자명했다. 계속 밀어붙이면 프로그램은 어떻게든 완성하겠지만 그랬다가는 너무 커서 손대기 어려운 골칫거리가 생겨날 참이었다. 코드 구조를 유지보수하기 좋은 상태로 만들려면 지금이 적기라 판단했다. 

그래서 나는 기능을 더 이상 추가하지 않기로 결정하고 리팩터링을 시작했다. String, Integer 유형을 추가한 경험에서 나는 새 인수 유형을 추가하려면 주요 지점 세 곳에도 코드를 추가해야 한다는 사실을 이미 깨달았다. 

인수 유형은 다양하지만 모두가 유사한 메서드를 제공하므로 클래스 하나가 적합하다 판단했다. 그래서 ArgumentMarshaler라는 개념이 탄생했다. 

<a name="2-2"></a>
#### 점진적으로 개선하다
프로그램을 망치는 가장 좋은 방법 중 하나는 개선이라는 이름 아래 구조를 크게 뒤집는 행위다. 어떤 프로그램은 그저 그런 '개선'에서 결코 회복하지 못한다. '개선' 전과 똑같이 프로그램을 돌리기가 아주 어렵기 때문이다. 

그래서 나는 테스트 주도 개발(Test-Driven Development, TDD)라는 기법을 사용했다. TDD는 언제 어느 때라도 시스템이 돌아가야 한다는 원칙을 따른다. 다시 말해, TDD는 시스템을 망가뜨리는 변경을 허용하지 않는다. 변경을 가한 후에도 시스템이 변경 전과 똑같이 돌아가야 한다는 말이다. 

변경 전후 시스템이 똑같이 돌아간다는 사실을 확인하려면 언제든 실행이 가능한 자동화된 테스트 슈트가 필요하다. 앞서 Args 클래스를 구현하는 동안에 나는 이미 단위 테스트 슈트와 인수 테스트를 만들어 놓았다. 두 테스트 모두 언제든 실행이 가능했고, 시스템이 두 테스트를 모두 통과하면 올바로 동작한다고 봐도 좋았다. 

그래서 나는 시스템에 자잘한 변경을 가하기 시작했다. 코드를 변경할 때마다 시스템 구조는 조금씩 ArgumentMashaler 개념에 가까워졌다. 또한 변경 후에도 시스템은 여전히 잘 돌아갔다. 

<a name="3"></a>
## String 인수
String 인수를 추가하는 과정은 boolean 인수와 매우 유사했다. HashMap을 변경한 후 parse, set, get 함수를 고쳤다. 

<a name="4"></a>
## 최종 코드
최종 코드는 다음과 같다.

##### 목록 14-15 ArgsException.java
```java

```

##### 목록 14-16 Args.java
```java

```
Args 클래스에서는 주로 코드만 삭제했을 뿐이다. 상당한 코드를 Args 클래스에서 ArgsException 클래스로 옮겼다. 멋지다. 또한 모든 ArgumentMarshaler 클래스도 각자 파일로 옮겼다. 더욱 멋지다!

소프트웨어 설계는 분할만 잘해도 품질이 크게 높아진다. 적절한 장소를 만들어 코드만 분리해도 설계가 좋아진다. 관심사를 분리하면 코드를 이해하고 보수하기 훨씬 더 쉬워진다. 

특별히 눈여겨볼 코드는 ArgsException의 errorMessage 메서드다. (Args 클래스에 속했던) 이 메서드는 명백히 SRP(Single Responsibility Principle) 위반이었다. Args 클래스가 오류 메서지 형식까지 책임졌기 때문이다. 이 클래스는 인수를 처리하는 클래스지 오류 메시지 관련을 처리하는 클래스가 아니기 때문이다. 하지만 그렇다고 ArgsException 클래스가 오류 메시지 형식을 처리해야 옳을까?

솔직하게 말해, 이것은 절충안이다. ArgsException에게 맡겨서는 안 된다고 생각하는 독자라면 새로운 클래스가 필요하다. 하지만 미리 깔끔하게 만들어진 오류 메시지로 얻는 장점은 무시하기 어렵다. 

<a name="5"></a>
## 결론
그저 돌아가는 코드만으로는 부족하다. 돌아가는 코드가 심하게 망가지는 사례는 흔하다. 단순히 돌아가는 코드에 만족하는 프로그래머는 전문가 정신이 부족하다. 설계와 구조를 개선할 시간이 없다고 변경할지 모르지만 나로서는 동의하기 어렵다. 나쁜 코드보다 더 오랫동안 더 심각하게 개발 프로젝트에 악영향을 미치는 요인도 없다. 나쁜 일정은 다시 짜면 된다. 나쁜 요구사항은 다시 정의하면 된다. 나쁜 팀 역학은 복구하면 된다. **하지만 나쁜 코드는 썩어 문드러진다.** 점점 무게가 늘어나 팀의 발목을 잡는다. 속도가 점점 느려지다 못해 기어가는 팀도 많이 봤다. 너무 서두르다가 이후로 영원히 자신들의 운명을 지배할 악성 코드라는 굴레를 짊어진다. 

물론 나쁜 코드도 깨끗한 코드로 개선할 수 있다. 하지만 비용이 엄청나게 많이 든다. 코드가 썩어가며 모듈은 서로서로 얽히고설켜 뒤엉키고 숨겨진 의존성이 수도 없이 생긴다. 오래된 의존성을 찾아내 깨려면 상당한 시간과 인내심이 필요하다. 반면 처음부터 코드를 깨끗하게 유지하기란 상대적으로 쉽다. 아침에 엉망으로 만든 코드를 오후에 정리하기는 어렵지 않다. 더욱이 5분 전에 엉망으로 만든 코드는 지금 당장 정리하기 아주 쉽다. 
 
그러므로 코드는 언제나 최대한 깔끔하고 단순하게 정리하자. 절대로 썩어가게 방치하면 안 된다. 
