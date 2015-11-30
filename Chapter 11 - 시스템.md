## 목차 ##
- 서론: 도시 만들기
- 시스템의 생성과 사용을 분리하라
 - 생성 로직을 어플리케이션의 시작이 아닌 메인으로
 - 팩토리 기법
 - 의존성 주입
- 스케일링
 - Cross-Cutting Concerns(관여)
- Cross-Cutting Concerns 해결을 위한 세 가지 방법
 - 자바 프록시
 - 순수 자바 AOP 프레임워크
 - AspectJ
- 시스템 아키텍쳐를 테스트 주도하라
- 의사 결정의 최적화
- 표준은 확실한 이득을 가져올 경우 추가하라
- 시스템에는 DSL(도메인 영역 언어)이 필요하다
- 결론

======================================================

> Complexity kills. It sucks the life out of developers, it makes products difficult to plan, build, and test.

## 서론: 도시 만들기 ##
 도시를 건설하고 관리하는 데에는 한 사람 만으로는 충분하지 않다. 그래도 도시는 돌아간다. 그것은 도시라는 거대한 덩어리를
수도, 전원, 교통 등의 모듈로 모듈화하고 관리되기 때문이다. 일정 수준의 추상화를 통해 큰 그림에 대한 이해 없이도 도시는
돌아간다.  
 소프트웨어 또한 이와 비슷한 방식으로 구성되기는 하나 도시의 모듈화 만큼의 추상화를 이루지 못하는 경우가 많다.  
 **클린 코드는 이 것을 낮은 단계의 추상화를 통해 이루는 것을 도와준다.**

## 시스템의 생성과 사용을 분리하라 ##
```
  /* Code 1-1 */
  
  public Service getService() {
      if (service == null)
          service = new MyServiceImpl(...); // Good enough default for most cases?
      return service;
  }
```

위 Code 1-1은 "Lazy Initialization/Evaluation(게으른 초기화)"의 일반적인 형태이다. 이는 불필요한 초기화 코스트의 최적화,
null 반환 방지 등의 이점을 가지는 코드이다.
하지만 이 코드로 인해 우리의 시스템은 MyServiceImpl 객체에 대한 의존성을 가지게 되었고 MyServiceImpl의 사용 여부와 관계 없이
무조건 이 의존성을 만족해야 하게 되었다.
테스트 수행에도 문제가 발생한다. 만약 MyServiceImpl 객체가 무거운 객체라면 테스트를 위한 Test Double / Mock Object를
service필드에 대입해야 하며, 이는 기존의 runtime 로직에 관여하기 때문에 모든 가능한 경우의 수를 고려해야 하는 문제도
발생한다.  
이러한 생성/사용의 분산은 모듈성을 저해하고 코드의 중복을 가져오므로
**잘 정돈된 견고한 시스템을 만들기 위해서는 전역적이고 일관된 의존성 해결 방법을 통해 위와 같은 작은 편의 코드들이 모듈성의 저해를 가져오는 것을 막아야 한다.**

```
  /* Code 1-2: Android Example */

  @Override
  public Object getSystemService(@ServiceName @NonNull String name) {
      /* Pre-contidion checks here... */

      if (WINDOW_SERVICE.equals(name)) {
          return mWindowManager;
      } else if (SEARCH_SERVICE.equals(name)) {
          ensureSearchManager();
          return mSearchManager;
      }
      return super.getSystemService(name);
  }
  
  // Usage
  ConnectivityManager cm =
      (ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE);
```

#### 생성 로직을 어플리케이션의 시작이 아닌 메인으로 ####
<p align="center"><img src="/images/figure 11-1.png" width="500" /></p>
생성과 사용을 분리하는 가장 간단한 방법은 모든 생성과 관련된 로직을 main으로 옮기는 것이다.  
어플리케이션에서는 사용할 모등 객체들이 main에서 잘 생성되었을 것이라 여기고 나머지 디자인에 집중할 수 있다.

#### 팩토리 기법 ####
<p align="center"><img src="/images/figure 11-2.png" width="500" /></p>
객체의 생성 시기를 직접 결정하려면 main에서 완성된 객체를 던져주기 보다 factory 객체를 만들어서 던져주자.
만약 자세한 구현을 숨기고 싶다면 Abstract Factory 패턴을 사용하자.<sup>[1](#myfootnote1)</sup>

#### 의존성 주입 ####

======================================================

#### 참조 ####
<a name="myfootnote1">
##### 1. Abstract Factory Pattern #####
</a>:
A factory is the location of a concrete class in the code at which objects are constructed. The intent in employing the pattern is to insulate the creation of objects from their usage and to create families of related objects without having to depend on their concrete classes.[2]This allows for new derived types to be introduced with no change to the code that uses the base class.
Use of this pattern makes it possible to interchange concrete implementations without changing the code that uses them, even at runtime. However, employment of this pattern, as with similar design patterns, may result in unnecessary complexity and extra work in the initial writing of code. Additionally, higher levels of separation and abstraction can result in systems which are more difficult to debug and maintain.  
참조: https://en.m.wikipedia.org/wiki/Abstract_factory_pattern
