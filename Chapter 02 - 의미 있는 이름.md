## 목차
- 의도를 분명히 밝혀라
- 그릇된 정보를 피하라
- 의미 있게 구분하라(불용어-noise word-를 쓰지 말자)
- 발음하기 쉬운 이름을 사용하라
- 검색하기 쉬운 이름을 사용하라
- 인코딩을 피하라(변수에 부가 정보를 덧붙여 표기하는 것을 뜻함.)
 - 헝가리안 표기법
 - 맴버 변수 접두어
 - 인터페이스와 구현
- 자신의 기억력을 자랑하지 마라
- 클래스 이름
- 메서드 이름
- 기발한 이름은 피하라
- 한 개념에 한 단어를 사용하라
- 말장난을 하지 마라(위 11과 같이 보기)
- 해법 영역(Solution Domain) 용어를 사용하자
- 문제 영역(Problem Domain) 용어를 사용하자
- 의미 있는 맥락을 추가하라
- 불필요한 맥락을 없애라

----

## 의도를 분명히 밝혀라  
- 변수의 존재 이유, 기능, 사용법 등이 변수/함수/클래스명에 드러나야 한다. 따로 주석이 필요하지 않을 정도로.  
- 의미를 함축하거나 독자(코드를 읽는 사람)가 사전지식을 가지고 있다고 가정하지 말자.
- 예시 1
  - Bad
    - int d; // elapsed time in days
  - Good
    - int elapsedTimeInDays;
    - int daysSinceCreation;
    - int daysSinceModification;
    - int fileAgeInDays;
- 예시 2

```java
// Bad
public List<int[]> getThem() {
    List<int[]> list1 = new ArrayList<int[]>();
    for (int[] x : theList) {
        if (x[0] == 4) {
            list1.add(x);
        }
    }
    return list1;
}
```
 
```java
// Good
public List<int[]> getFlaggedCells() {
    List<int[]> flaggedCells = new ArrayList<int[]>();
    for (int[] cell : gameBoard) {
        if (cell[STATUS_VALUE] == FLAGGED) {
            flaggedCells.add(cell);
        }
    }
    return flaggedCells;
}
```

## 그릇된 정보를 피하라  
- 중의적으로 해석될 수 있는 이름 지양하기.  
- 개발자에게는 특수한 의미를 가지는 단어(List 등)는 실제 컨테이너가 List가 아닌 이상 accountList와 같이 변수명에 붙이지 말자. 차라리 accountGroup, bunchOfAccounts, accounts등으로 명명하자  
- 비슷해 보이는 명명에 주의하자.  

## 의미 있게 구분하라(불용어-noise word-를 쓰지 말자)  
- 말이 안되는 단어(한 글자만 바꾼다던지 한 단어), [a1, a2, …]과 같이 숫자로 구분하는 경우 주의  
- 클래스 이름에 Info, Data와 같은 불용어를 붙이지 말자. 정확한 개념 구분이 되지 않음  
- 예시  
 - `Name` VS `NameString`
 - `getActiveAccount()` VS `getActiveAccounts()` VS `getActiveAccountInfo()` (이들이 혼재할 경우 서로의 역할을 정확히 구분하기 어렵다.)
 - `money` VS `moneyAmount`
 - `message` VS `theMessage`

## 발음하기 쉬운 이름을 사용하라  

```java
// Bad
class DtaRcrd102 {
    private Date genymdhms;
    private Date modymdhms;
    private final String pszqint = "102";
    /* ... */
};
```
 
```java
// Good
class Customer {
    private Date generationTimestamp;
    private Date modificationTimestamp;
    private final String recordId = "102";
    /* ... */
};
```

## 검색하기 쉬운 이름을 사용하라  
- 상수는 static final과 같이 정의해 쓰자.  
- 변수 이름의 길이는 변수의 범위에 비례해서 길어진다.  

## 인코딩을 피하라(변수에 부가 정보를 덧붙여 표기하는 것을 뜻함.)  
- 헝가리안 표기법
  - 변수명에 해당 변수의 타입(String, Int 등)을 적지 말자
- 맴버 변수 접두어 
  - 맴버 변수 접두어를 붙이지 말자(???)  
- 인터페이스와 구현
  - 인터페이스 클래스와 구현 클래스를 나눠야 한다면 구현 클래스의 이름에 정보를 인코딩하자.  
 
| Do / Don't | Interface class | Concrete(Implementation) class |
| ---------- | --------------- | ------------------------------ |
| Don't      | IShapeFactory   | ShapeFactory                   |
| Do         | ShapeFactory    | ShapeFactoryImp                |
| Do         | ShapeFactory    | CShapeFactory                  |

## 자신의 기억력을 자랑하지 마라  
- 독자가 머리속으로 한번 더 생각해 변환해야 할만한 변수명을 쓰지 말라.(eg, URL에서 호스트와 프로토콜을 제외한 소문자 주소를 r이라는 변수로 명명하는 일 등)  
- 똑똑한 프로그래머와 전문가 프로그래머를 나누는 기준 한가지는 "Clarity(명료함)"이다.  

## 클래스 이름  
- 명사 혹은 명사구를 사용하라.(Customer, WikiPage, Account, AddressParser)  
- Manager, Processor, Data, Info와 같은 단어는 피하자  
- 동사는 사용하지 않는다.  

## 메서드 이름  
- 동사 혹은 동사구를 사용하라.(postPayment, deletePayment, deletePage, save 등)  
- 접근자, 변경자, 조건자는 get, set, is로 시작하자.  (추가: should, has 등도 가능)
- 생성자를 오버로드할 경우 정적 팩토리 메서드를 사용하고 해당 생성자를 private으로 선언한다.

```java 
// 첫번째 보다 두 번째 방법이 더 좋다.  
Complex fulcrumPoint = new Complex(23.0);  
Complex fulcrumPoint = Complex.FromRealNumber(23.0);  
```

## 기발한 이름은 피하라  
- 특정 문화에서만 사용되는 재미있는 이름보다 의도를 분명히 표현하는 이름을 사용하라  
  - HolyHandGrenade → DeleteItems  
  - whack() → kill()  

## 한 개념에 한 단어를 사용하라  
- 추상적인 개념 하나에 단어 하나를 사용하자.  
  - fetch, retrieve, get  
  - controller, manager, driver  

## 말장난을 하지 마라(위 내용에 이어)  
- 한 단어를 두 가지 목적으로 사용하지 말자. 아래와 같은 경우에는 ii를 append 혹은 insert로 바꾸는게 옳겠다.

```java
public static String add(String message, String messageToAppend)  
public List<Element> add(Element element)  
```

## 해법 영역(Solution Domain) 용어를 사용하자  
- 개발자라면 당연히 알고 있을 `JobQueue`, `AccountVisitor(Visitor pattern)`등을 사용하지 않을 이유는 없다. 전산용어, 알고리즘 이름, 패턴 이름, 수학 용어 등은 사용하자.  

## 문제 영역(Problem Domain) 용어를 사용하자  
- 적절한 프로그래머 용어(위 13)가 없거나 문제영역과 관련이 깊은 용어의 경우 문제 영역 용어를 사용하자.  

## 의미 있는 맥락을 추가하라  
- 클래스, 함수, namespace등으로 감싸서 맥락(Context)을 표현하라  
- 그래도 불분명하다면 접두어를 사용하자.  

```java
// Bad
private void printGuessStatistics(char candidate, int count) {
    String number;
    String verb;
    String pluralModifier;
    if (count == 0) {  
        number = "no";  
        verb = "are";  
        pluralModifier = "s";  
    }  else if (count == 1) {
        number = "1";  
        verb = "is";  
        pluralModifier = "";  
    }  else {
        number = Integer.toString(count);  
        verb = "are";  
        pluralModifier = "s";  
    }
    String guessMessage = String.format("There %s %s %s%s", verb, number, candidate, pluralModifier );

    print(guessMessage);
}
```

```java
// Good
public class GuessStatisticsMessage {
    private String number;
    private String verb;
    private String pluralModifier;

    public String make(char candidate, int count) {
        createPluralDependentMessageParts(count);
        return String.format("There %s %s %s%s", verb, number, candidate, pluralModifier );
    }

    private void createPluralDependentMessageParts(int count) {
        if (count == 0) {
            thereAreNoLetters();
        } else if (count == 1) {
            thereIsOneLetter();
        } else {
            thereAreManyLetters(count);
        }
    }

    private void thereAreManyLetters(int count) {
        number = Integer.toString(count);
        verb = "are";
        pluralModifier = "s";
    }

    private void thereIsOneLetter() {
        number = "1";
        verb = "is";
        pluralModifier = "";
    }

    private void thereAreNoLetters() {
        number = "no";
        verb = "are";
        pluralModifier = "s";
    }
}
```

## 불필요한 맥락을 없애라  
- `Gas Station Delux` 이라는 어플리케이션을 작성한다고 해서 클래스 이름의 앞에 GSD를 붙이지는 말자. G를 입력하고 자동완성을 누를 경우 모든 클래스가 나타나는 등 효율적이지 못하다.  
위 a처럼 접두어를 붙이는 것은 모듈의 재사용 관점에서도 좋지 못하다. 재사용하려면 이름을 바꿔야 한다.(eg, `GSDAccountAddress` 대신 `Address`라고만 해도 충분하다.)  


> 두려워하지 말고 서로의 명명을 지적하고 고치자. 그렇게 하면 이름을 외우는 것에 시간을 빼앗기지 않고 "자연스럽게 읽히는 코드"를 짜는 데에 더 집중할 수 있다.  
