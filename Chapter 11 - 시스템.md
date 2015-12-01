## 목차 ##
- 서론: 도시 만들기
- 시스템의 생성과 사용을 분리하라
 - 생성 로직을 어플리케이션의 시작이 아닌 메인으로
 - 팩토리 기법
 - 의존성 주입(Dependency Injection)
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
테스트 수행에도 문제가 발생한다. 만약 MyServiceImpl 객체가 무거운 객체라면 테스트를 위한 Test Double<sup> [1](#fn1)</sup> / Mock Object를
service필드에 대입해야 하며, 이는 기존의 runtime 로직에 관여하기 때문에 모든 가능한 경우의 수를 고려해야 하는 문제도
발생한다.  
이러한 생성/사용의 분산은 모듈성을 저해하고 코드의 중복을 가져오므로
**잘 정돈된 견고한 시스템을 만들기 위해서는 전역적이고 일관된 의존성 해결 방법을 통해 위와 같은 작은 편의 코드들이 모듈성의 저해를 가져오는 것을 막아야 한다.**

```java
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
만약 자세한 구현을 숨기고 싶다면 Abstract Factory 패턴을 사용하자.<sup> [2](#fn2)</sup>

#### 의존성 주입(Dependency Injection) ####
의존성 관리의 관점에서는 "객체는 그 자신의 의존성들을 직접 생성하지 말고 다른 'authoritative mechanism'에게 맡겨야 한다."라고 한다. 아래의 예를 보자.(JNDI가 실제로 어떤 일을 하는지는 본 챕터와 관계가 없으므로 생략한다)
```java
    /* Code 1-3 */
    MyService myService = (MyService)(jndiContext.lookup(“NameOfMyService”));
```
위 코드를 호출하는 쪽에서는 실제로 lookup 메서드가 무엇을(어떤 구현체를) 리턴하는지에 대해 관여하지 않으면서 의존성을 해결할 수 있다.

진정한 의존성 주입은 여기에서 한 단계 더 나아가 완전히 수동적인 형태를 지닌다. 의존성을 필요로 하는 객체가 직접 의존성을 해결(생성, 연결)하는 대신 생성자/setter 등을 통해 DI 컨테이너가 해당 의존성을 해결하도록 도와준다.(DI / IoC)<sup> [3](#fn3)</sup>

## 스케일링 ##
촌락은 마을로, 마을은 도시로 성장한다. 하지만 누가 마을의 성장을 고려해 미리 6차선 고속도로를 지으려 할까?  
처음부터 시스템을 제대로 제대로 만든다는 것은 미신일 뿐이다. 우리는 오늘 필요한 것을 만들 뿐이다. 내일 할 일은 테스트 기반 개발, 리펙토링, 그리고 클린코드가 이를 코드 레벨에서 도와줄 것이다.

소프트웨어 시스템 또한 마찬가지이다. 만약 우리가 **관여들(Concerns)을 적절히 분리**할 수 있다면, 소프트웨어 시스템은 물리적인 시스템(ex, 건축)과는 다르게 점진적으로 커질 수 있다. 

먼저, 스케일링을 고려하지 않은 구조에 대해 EJB1/EJB2를 예시로 알아보자.
* EJB에 대한 자세한 내용은 본 챕터와 관계가 없으므로 생략한다. (EJB에 대한 자세한 개요는 각주로 추가 바람)  
우선 entity bean이란 관계 데이터(DB 테이블의 행)의 메모리상의 표현이라는 것만 알고 가자. (An entity bean is an in-memory representation of relational data, in other words, a table row.)

```java
/* Code 2-1(Listing 11-1): An EJB2 local interface for a Bank EJB */

package com.example.banking;
import java.util.Collections;
import javax.ejb.*;

public interface BankLocal extends java.ejb.EJBLocalObject {
    String getStreetAddr1() throws EJBException;
    String getStreetAddr2() throws EJBException;
    String getCity() throws EJBException;
    String getState() throws EJBException;
    String getZipCode() throws EJBException;
    void setStreetAddr1(String street1) throws EJBException;
    void setStreetAddr2(String street2) throws EJBException;
    void setCity(String city) throws EJBException;
    void setState(String state) throws EJBException;
    void setZipCode(String zip) throws EJBException;
    Collection getAccounts() throws EJBException;
    void setAccounts(Collection accounts) throws EJBException;
    void addAccount(AccountDTO accountDTO) throws EJBException;
}
```

```java
/* Code 2-2(Listing 11-2): The corresponding EJB2 Entity Bean Implementation */

package com.example.banking;
import java.util.Collections;
import javax.ejb.*;

public abstract class Bank implements javax.ejb.EntityBean {
    // Business logic...
    public abstract String getStreetAddr1();
    public abstract String getStreetAddr2();
    public abstract String getCity();
    public abstract String getState();
    public abstract String getZipCode();
    public abstract void setStreetAddr1(String street1);
    public abstract void setStreetAddr2(String street2);
    public abstract void setCity(String city);
    public abstract void setState(String state);
    public abstract void setZipCode(String zip);
    public abstract Collection getAccounts();
    public abstract void setAccounts(Collection accounts);
    
    public void addAccount(AccountDTO accountDTO) {
        InitialContext context = new InitialContext();
        AccountHomeLocal accountHome = context.lookup("AccountHomeLocal");
        AccountLocal account = accountHome.create(accountDTO);
        Collection accounts = getAccounts();
        accounts.add(account);
    }
    
    // EJB container logic
    public abstract void setId(Integer id);
    public abstract Integer getId();
    public Integer ejbCreate(Integer id) { ... }
    public void ejbPostCreate(Integer id) { ... }
    
    // The rest had to be implemented but were usually empty:
    public void setEntityContext(EntityContext ctx) {}
    public void unsetEntityContext() {}
    public void ejbActivate() {}
    public void ejbPassivate() {}
    public void ejbLoad() {}
    public void ejbStore() {}
    public void ejbRemove() {}
}
```
1. 비지니스 로직이 EJB2 컨테이너에 타이트하게 연결되어 있다. Entity를 만들기 위해 컨테이너 타입을 subclass하고 필요한 lifecycle 메서드를 구현해야 한다.
2. 실제로 사용되지도 않을 테스트 객체의 작성을 위해 mock 객체를 만드는 데에도 무의미한 노력이 많이 든다. EJB2 구조가 아닌 다른 구조에서 재사용할 수 없는 컴포넌트를 작성해야 한다.
3. OOP 또한 등한시되고 있다. 상속도 불가능하며 쓸데없는 DTO(Data Transfer Object)를 작성하게 만든다.
======================================================

#### 참조 ####
<a name="fn1">
##### 1. Test Double #####
</a>
https://en.wikipedia.org/wiki/Test_double

<a name="fn2">
##### 2. Abstract Factory Pattern #####
</a>
A factory is the location of a concrete class in the code at which objects are constructed. The intent in employing the pattern is to insulate the creation of objects from their usage and to create families of related objects without having to depend on their concrete classes.[2]This allows for new derived types to be introduced with no change to the code that uses the base class.
Use of this pattern makes it possible to interchange concrete implementations without changing the code that uses them, even at runtime. However, employment of this pattern, as with similar design patterns, may result in unnecessary complexity and extra work in the initial writing of code. Additionally, higher levels of separation and abstraction can result in systems which are more difficult to debug and maintain.  
참조: https://en.m.wikipedia.org/wiki/Abstract_factory_pattern

<a name="fn3">
##### 3. Dependency Injection and Inversion of Control #####
</a>
http://greatkim91.tistory.com/41 
