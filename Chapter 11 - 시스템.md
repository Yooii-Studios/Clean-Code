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
이러한 생성/사용의 분산은 
잘 정돈된 견고한 시스템을 만들기 위해서는 전역적이고 일관된 의존성 해결 방법을 통해 위와 같은 작은 편의 코드들이 모듈성의 저해를 가져오는 것을 막아야 한다.

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

======================================================

#### 참조 ####
##### 1. Adapter Pattern #####
참조: http://ko.m.wikipedia.org/wiki/어댑터_패턴
