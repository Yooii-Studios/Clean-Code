## 목차 ##
- 자료 추상화  
- 자료/객체 비대칭  
- 디미터 법칙  
  - 기차 충돌  
  - 잡종 구조  
  - 구조체 감추기  
- 자료 전달 객체  
  - 활성 레코드  
- 결론  
- 참고 문헌  

## Intro ##

변수를 비공개private로 정의하는 이유가 있다. 남들이 변수에 의존하지 않게 만들고 싶어서다. 충동이든 변덕이든, 변수 타입이나 구현을 맘대로 바꾸고 싶어서다. 그렇다면 어째서 수많은 프로그래머가 조회get 함수와 설정set 함수를 당연하게 공개public해 비공개 변수를 외부에 노출할까?

## 자료 추상화 ##

###### 목록 6-1 구체적인 Point 클래스
```java
public class Point { 
  public double x; 
  public double y;
}
```

###### 목록 6-2 추상적인 Point 클래스
```java
public interface Point {
  double getX();
  double getY();
  void setCartesian(double x, double y); 
  double getR();
  double getTheta();
  void setPolar(double r, double theta); 
}
```

목록 6-2는 구현을 완전히 숨긴다. 그에 반해 목록 6-1은 내부 구조를 노출하고, 개별적으로 좌표값을 읽고 설정하게 강제한다. 일반적으로 변수를 private으로 많이 선언을 하는데, 각 값마다 get과 set 함수를 제공한다면 이는 결과적으로 내부 구조를 노출하는 구조가 된다.  
  변수 사이에 함수라는 계층을 계층을 넣는다고 구현이 저절로 감춰지지는 않는다. 구현을 감추려면 **추상화**가 필요하다! set, get 메서드로 변수를 다룬다고 클래스가 되는 것이 아니라, 추상 인터페이스를 제공해 사용자가 구현을 모른 채 자료의 핵심을 조작할 수 있어야 진정한 의미의 클래스다. 
  
## 자료/객체 비대칭
1. 객체는 추상화 뒤로 자료를 숨긴 채 자료를 다루는 함수만 공개한다. 
2. 자료 구조는 자료를 그대로 공개하며 별다른 함수는 제공하지 않는다. 

두 정의는 본질적으로 상반된다. 두 개념은 사실상 정반대다. 사소한 차이로 보일지 모르지만 그 차이가 미치는 영향은 굉장히다. 

###### 목록 6-5 절차적인 도형 (Procedural Shape)
```java
public class Square { 
  public Point topLeft; 
  public double side;
}

public class Rectangle { 
  public Point topLeft; 
  public double height; 
  public double width;
}

public class Circle { 
  public Point center; 
  public double radius;
}

public class Geometry {
  public final double PI = 3.141592653589793;
  
  public double area(Object shape) throws NoSuchShapeException {
    if (shape instanceof Square) { 
      Square s = (Square)shape; 
      return s.side * s.side;
    } else if (shape instanceof Rectangle) { 
      Rectangle r = (Rectangle)shape; 
      return r.height * r.width;
    } else if (shape instanceof Circle) {
      Circle c = (Circle)shape;
      return PI * c.radius * c.radius; 
    }
    throw new NoSuchShapeException(); 
  }
}
```
  
객체 지향 프로그래머가 위 코드를 본다면 코웃음을 칠지도 모르겠다. 클래스가 절차적이라 비판한다면 맞는 말이다. 하지만 그런 비웃음이 100% 옳다고 말하기는 어렵다. **만약 Geometry 클래스에 둘레 길이를 구하면 `perimeter()` 함수를 추가하고 싶다면? 도형 클래스는 아무 영향도 받지 않는다!** 도형 클래스에 의존하는 다른 클래스도 마찬가지다! **반대로 새 도형을 추가하고 싶다면 Geometry 클래스에 속한 함수를 모두 고쳐야 한다.** 그래서 두 조건은 완전히 정반대라고 할 수 있다. 
  
이번에는 목록 6-6을 살펴보자. 객체 지향적인 도형 클래스다. 새 도형을 추가해서 기존 함수에 아무런 영향을 미치지 않는다. 반면 새 함수를 추가하고 싶다면 도형 클래스 전부를 고쳐야 한다.  

###### 목록 6-6 다형적인 도형 (Polymorphic Shape)
```java
public class Square implements Shape { 
  private Point topLeft;
  private double side;
  
  public double area() { 
    return side * side;
  } 
}

public class Rectangle implements Shape { 
  private Point topLeft;
  private double height;
  private double width;

  public double area() { 
    return height * width;
  } 
}

public class Circle implements Shape { 
  private Point center;
  private double radius;
  public final double PI = 3.141592653589793;

  public double area() {
    return PI * radius * radius;
  } 
}
```

앞서도 말했듯이, 두 방식은 사실상 반대다! 그래서 객체와 자료 구조는 근본적으로 양분된다. 

> (자료 구조를 사용하는) 절차적인 코드는 기존 자료 구조를 변경하지 않으면서 새 함수를 추가하기 쉽다. 반면, 객체 지향 코드는 기존 함수를 변경하지 않으면서 새 클래스를 추가하기 쉽다. 

반대쪽도 참이다.

> 절차적인 코드는 새로운 자료 구조를 추가하기 어렵다. 그러려면 모든 함수를 고쳐야 한다. 객체 지향 코드는 새로운 함수를 추가하기 어렵다. 그러려면 모든 클래스를 고쳐야 한다. 

다시 말해, **객체 지향 코드에서 어려운 변경은 절차적인 코드에서 쉬우며, 절차적인 코드에서 어려운 변경은 객체 지향 코드에서 쉽다!**  

복잡한 시스템을 짜다 보면 새로운 함수가 필요할 경우거나, 새로운 자료 타입이 필요한 경우가 생긴다. 이때 상황에 맞게 **클래스 & 객체 지향 기법**을 사용하거나, **절차적인 코드와 자료 구조**를 적절하게 사용하는 것이 좋다.   

  분별 있는 프로그래머는 모든 것이 객체라는 생각이 **미신**임을 잘 안다. 때로는 단순한 자료 구조와 절차적인 코드가 가장 적합한 상황도 있다. 

## 디미터 법칙 ##
디미터 법칙은 잘 알려진 휴리스틱heuristic(경험에 기반하여 문제를 해결하거나 학습하거나 발견해 내는 방법)으로, 모듈은 자신이 조작하는 객체의 속사정을 몰라야 한다는 법칙이다. 

좀 더 정확히 표현하자면, 디미터 법칙은 "클래스 C의 메서드 f는 다음과 같은 객체의 메서드만 호출해야 한다"고 주장한다. 

* 클래스 C
* f가 생성한 객체
* f 인수로 넘어온 객체
* C 인스턴스 변수에 저장된 객체

하지만 위 객체에서 허용된 메서드가 반환하는 객체의 메서드는 호출하면 안 된다. 다시 말해, **낯선 사람은 경계하고 친구랑만 놀라는 의미**이다. 

#### 기차 충돌 ####
다음과 같은 코드를 기차 충돌train wreck이라 부른다. 
```java
final String outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();
```
여러 객차가 한 줄로 이어진 기차처럼 보이기 때문이다. 일반적으로 조잡하다 여겨지는 방식이므로 피하는 편이 좋다. 위 코드는 다음과 같이 나누는 편이 좋다.

```java
Options opts = ctxt.getOptions();
File scratchDir = opts.getScratchDir();
final String outputDir = scratchDir.getAbsolutePath();
```

위 예제가 디미터 법칙을 위반하는지 여부는 위의 변수들이 객체인지 자료 구조인지에 달렸다. 객체라면 내부 구조를 숨겨야 하므로 확실히 디미터 법칙을 위반한다. 반면, 자료 구조라면 당연히 내부 구조를 노출하므로 문제되지 않는다. 

#### 잡종 구조
이런 혼란으로 말미암아 때때로 절반은 객체, 절반은 자료 구조인 잡종 구조가 나온다. 잡종 구조는 중요한 기능을 수행하는 함수도 있고, 공개 변수나 공개 get/set 함수도 있다. 이런 구조는 새로운 함수는 물론이고 새로운 자료 구조도 추가하기 어렵다. 양쪽 세상에서 단점만 모아놓은 구조다. 그러므로 되도록 이런 구조는 피하도록 하자. **프로그래머가 함수나 타입을 보호할지 공개할지 확신하지 못해 (더 나쁘게는 무지해) 어중간하게 내놓은 설계에 불과하다.**

#### 구조체 감추기
위의 `outputDir` 예제의 경우 좋은 방식이 아니다. 이 경로를 왜 필요할지 같은 모듈에서 찾아 보았더니 (한참 아래로 내려가서) 이런 코드가 있다. 

```java
String outFile = outputDir + "/" + className.replace('.', '/') + ".class"; 
FileOutputStream fout = new FileOutputStream(outFile); 
BufferedOutputStream bos = new BufferedOutputStream(fout);
```

추상화 수준을 뒤섞어 놓아 다소 불편하다. 점, 슬래시, 파일 확장자, File 객체를 부주의하게 마구 뒤섞으면 안 된다. 어찌 되었거나, 위 코드에 따르면 경로를 얻으려는 이유가 임시 파일을 생성하기 위함을 알 수 있다.  

그렇다면 ctxt 객체에 임시 파일을 생성하라고 시키면 어떨까?

```java
BufferedOutputStream bos = ctxt.createScratchFileStream(classFileName);
```

객체에게 맡기기에 적당한 임무로 보인다! ctxt는 내부 구조를 드러내지 않으며, 모듈은 자신이 몰라야 하는 여러 객체를 탐색할 필요가 없다. 따라서 디미터 법칙을 위반하지 않는다. 

## 자료 전달 객체
자료 구조체의 전형적인 형태는 공개 변수만 있고 함수가 없는 클래스다. 이를 때로는 자료 전달 객체Data Transfer Object, DTO라 한다. 

```java
public class Address { 
  public String street; 
  public String streetExtra; 
  public String city; 
  public String state; 
  public String zip;
}
```
#### 활성 레코드
DTO의 특수한 형태다. 공개 변수가 있거나 비공개 변수에 getter/setter가 있는 자료 구조지만, 대게 save나 find와 같은 탐색 함수도 제공한다. 활성 레코드는 데이터베이스 테이블이나 다른 소스에서 자료를 직접 변환한 결과다.  
  불행히도 활성 레코드에 비즈니스 규칙 메서드를 추가해 이런 자료 구조를 객체로 취급하는 개발자가 흔하다. 하지만 이렇게 하게 되면 잡종 구조가 나오게 된다.  
  
  해결책은 당연하다. **활성 레코드는 자료 구조로 취급한다.** 비즈니스 규칙을 담으면서 내부 자료를 숨기는 객체는 따로 생성한다. (여기서 내부 자료는 활성 레코드의 인스턴스일 가능성이 높다.)
  
## 결론 ##
객체는 동작을 공개하고 자료를 숨긴다. 그래서 기존 동작을 변경하지 않으면서 새 객체 타입을 추가하기는 쉬운 반면, 기존 객체에 새 동작을 추가하기는 어렵다.   

자료 구조는 별다른 동작 없이 자료를 노출한다. 그래서 기존 자료 구조에 새 동작을 추가하기는 쉬우나, 기존 함수에 새 자료 구조를 추가하기는 어렵다.  

(어떤) 시스템을 구현할 때, 새로운 자료 타입을 추가하는 유연성이 필요하면 객체가 더 적합하다. 다른 경우로 새로운 동작을 추가하는 유연성이 필요하면 자료 구조와 절차적인 코드가 더 적합하다. 우수한 소프트웨어 개발자는 편견 없이 이 사실을 이해해 직면한 문제에 최적인 해결책을 선택한다. 
