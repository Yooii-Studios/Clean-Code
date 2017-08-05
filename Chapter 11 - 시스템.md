## 목차 ##
- [서론: 도시 만들기](#1)
- [시스템의 생성과 사용을 분리하라](#2)
 - [생성 로직을 어플리케이션의 시작이 아닌 메인으로](#2-1)
 - [팩토리 기법](#2-2)
 - [의존성 주입(Dependency Injection)](#2-3)
- [스케일링](#3)
 - [Cross-Cutting Concerns(관심 분야)](#3-1)
- [Cross-Cutting Concerns 해결을 위한 세 가지 방법](#4)
 - [자바 프록시](#4-1)
 - [순수 자바 AOP 프레임워크](#4-2)
 - [AspectJ](#4-3)
- [시스템 아키텍쳐를 테스트 주도하라(Test Drive the System Architecture)](#5)
- [의사 결정의 최적화하라](#6)
- [표준은 확실한 이득을 가져올 경우 추가하라](#7)
- [시스템에는 DSL(도메인 영역 언어)이 필요하다](#8)
- [결론](#9)

---

> Complexity kills. It sucks the life out of developers, it makes products difficult to plan, build, and test.

<a name="1"></a>
## 서론: 도시 만들기 ##
 도시를 건설하고 관리하는 데에는 한 사람 만으로는 충분하지 않다. 그래도 도시는 돌아간다. 그것은 도시라는 거대한 덩어리를
수도, 전원, 교통 등의 모듈로 모듈화하고 관리되기 때문이다. 일정 수준의 추상화를 통해 큰 그림에 대한 이해 없이도 도시는
돌아간다.

 소프트웨어 또한 이와 비슷한 방식으로 구성되기는 하나 도시의 모듈화 만큼의 추상화를 이루지 못하는 경우가 많다.
 
 **클린 코드는 이 것을 낮은 단계의 추상화를 통해 이루는 것을 도와준다.**

<a name="2"></a>
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

이러한 생성/사용의 분산은 모듈성을 저해하고 코드의 중복을 가져오므로 **잘 정돈된 견고한 시스템을 만들기 위해서는 전역적이고 일관된 의존성 해결 방법을 통해 위와 같은 작은 편의 코드들이 모듈성의 저해를 가져오는 것을 막아야 한다.**

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

<a name="2-1"></a>
#### 생성 로직을 어플리케이션의 시작이 아닌 메인으로 ####
<p align="center"><img src="/images/figure 11-1.png" width="500" /></p>
생성과 사용을 분리하는 가장 간단한 방법은 모든 생성과 관련된 로직을 main으로 옮기는 것이다.  
어플리케이션에서는 사용할 모등 객체들이 main에서 잘 생성되었을 것이라 여기고 나머지 디자인에 집중할 수 있다.

<a name="2-2"></a>
#### 팩토리 기법 ####
<p align="center"><img src="/images/figure 11-2.png" width="500" /></p>
객체의 생성 시기를 직접 결정하려면 main에서 완성된 객체를 던져주기 보다 factory 객체를 만들어서 던져주자.

만약 자세한 구현을 숨기고 싶다면 Abstract Factory 패턴을 사용하자.<sup> [2](#fn2)</sup>

<a name="2-3"></a>
#### 의존성 주입(Dependency Injection) ####
의존성 관리의 관점에서는 "객체는 그 자신의 의존성들을 직접 생성하지 말고 다른 'authoritative mechanism'에게 맡겨야 한다."라고 한다. 아래의 예를 보자.(JNDI가 실제로 어떤 일을 하는지는 본 챕터와 관계가 없으므로 생략한다)
```java
    /* Code 1-3 */
    MyService myService = (MyService)(jndiContext.lookup(“NameOfMyService”));
```
위 코드를 호출하는 쪽에서는 실제로 lookup 메서드가 무엇을(어떤 구현체를) 리턴하는지에 대해 관여하지 않으면서 의존성을 해결할 수 있다.

진정한 의존성 주입은 여기에서 한 단계 더 나아가 완전히 수동적인 형태를 지닌다. 의존성을 필요로 하는 객체가 직접 의존성을 해결(생성, 연결)하는 대신 생성자/setter 등을 통해 DI 컨테이너가 해당 의존성을 해결하도록 도와준다.(DI / IoC)<sup> [3](#fn3)</sup>

<a name="3"></a>
## 스케일링 ##
촌락은 마을로, 마을은 도시로 성장한다. 하지만 누가 마을의 성장을 고려해 미리 6차선 고속도로를 지으려 할까?  
처음부터 시스템을 제대로 제대로 만든다는 것은 미신일 뿐이다. 우리는 오늘 필요한 것을 만들 뿐이다. 내일 할 일은 테스트 기반 개발, 리펙토링, 그리고 클린코드가 이를 코드 레벨에서 도와줄 것이다.

소프트웨어 시스템 또한 마찬가지이다. 만약 우리가 **Concern들을 적절히 분리**할 수 있다면, 소프트웨어 시스템은 물리적인 시스템(ex, 건축)과는 다르게 점진적으로 커질 수 있다. 

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

위 코드와 같은 전형적인 EJB2 객체 구조는 아래와 같은 문제점을 가지고 있다.  
1. 비지니스 로직이 EJB2 컨테이너에 타이트하게 연결되어 있다. Entity를 만들기 위해 컨테이너 타입을 subclass하고 필요한 lifecycle 메서드를 구현해야 한다.  
2. 실제로 사용되지 않을 테스트 객체의 작성을 위해 mock 객체를 만드는 데에도 무의미한 노력이 많이 든다. EJB2 구조가 아닌 다른 구조에서 재사용할 수 없는 컴포넌트를 작성해야 한다.  
3. OOP 또한 등한시되고 있다. 상속도 불가능하며 쓸데없는 DTO(Data Transfer Object)를 작성하게 만든다.  

<a name="3-1"></a>
#### Cross-Cutting Concerns(관심 분야) ####
* Cross-Cuttin Concerns란?  
 - 이론적으로는 독립된 형태로 구분될 수 있지만 실제로는 코드에 산재하기 쉬운 부분들을 뜻한다.(transaction, authorization, logging등)  

반면, 어떤 측면에서는 EJB2 아키텍쳐는 시스템의 스케일링을 위한 concern의 분리를 잘 이행하고 있다. 이들은 AOP(aspect-oriented programming)<sup> [4](#fn4)</sup>를 통해 transaction, logging과 같은 cross-cutting concerns의 모듈성을 되살리고 있다.
AOP에서는 "코드의 어느 부분에 어떤 추가적인 기능을 삽입할까"에 대한 정의를 aspect라는 형태로 제공한다.  

이제 자바를 사용한 aspect-like mechanism에 대해 알아보자.

<a name="4"></a>
## Cross-Cutting Concerns 해결을 위한 세 가지 방법<sup> [5](#fn5)</sup> ##
<a name="4-1"></a>
#### 자바 프록시<sup> [6](#fn6)</sup> ####
간단한 경우라면 자바 프록시가 적절한 솔루션일 것이다. 아래는 자바 프록시를 사용해 객체의 변경이 자동으로 persistant framework에 저장되는 구조에 대한 예시이다.

```java
/* Code 3-1(Listing 11-3): JDK Proxy Example */

// Bank.java (suppressing package names...)
import java.utils.*;

// The abstraction of a bank.
public interface Bank {
    Collection<Account> getAccounts();
    void setAccounts(Collection<Account> accounts);
}

// BankImpl.java
import java.utils.*;

// The “Plain Old Java Object” (POJO) implementing the abstraction.
public class BankImpl implements Bank {
    private List<Account> accounts;

    public Collection<Account> getAccounts() {
        return accounts;
    }
    
    public void setAccounts(Collection<Account> accounts) {
        this.accounts = new ArrayList<Account>();
        for (Account account: accounts) {
            this.accounts.add(account);
        }
    }
}
// BankProxyHandler.java
import java.lang.reflect.*;
import java.util.*;

// “InvocationHandler” required by the proxy API.
public class BankProxyHandler implements InvocationHandler {
    private Bank bank;
    
    public BankHandler (Bank bank) {
        this.bank = bank;
    }
    
    // Method defined in InvocationHandler
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        String methodName = method.getName();
        if (methodName.equals("getAccounts")) {
            bank.setAccounts(getAccountsFromDatabase());
            
            return bank.getAccounts();
        } else if (methodName.equals("setAccounts")) {
            bank.setAccounts((Collection<Account>) args[0]);
            setAccountsToDatabase(bank.getAccounts());
            
            return null;
        } else {
            ...
        }
    }
    
    // Lots of details here:
    protected Collection<Account> getAccountsFromDatabase() { ... }
    protected void setAccountsToDatabase(Collection<Account> accounts) { ... }
}

// Somewhere else...
Bank bank = (Bank) Proxy.newProxyInstance(
    Bank.class.getClassLoader(),
    new Class[] { Bank.class },
    new BankProxyHandler(new BankImpl())
);
```
위 코드에 대한 간략한 설명은 아래와 같다.  
1. Java Proxy API를 위한 Bank 인터페이스를 작성한다.  
2. 위에서 작성한 Bank 인터페이스를 사용한 BankImpl(POJO aka Plane Old Java Object)를 구현한다. 여기에는 순수한 데이터만 들어가며 비지니스 로직은 포함되지 않는다.(모델과 로직의 분리)  
3. InvocationHandler를 구현하는 BankProxyHandler를 작성한다. 이 핸들러는 Java Reflection API를 이용해 Bank 인터페이스를 구현하는 객체들의 메서드콜을 가로챌 수 있으며 추가적인 로직을 삽입할 수 있다. 본 예제에서 비지니스 로직(persistant stack logic)은 이 곳에 들어간다.  
4. 마지막으로 코드의 마지막 블럭과 같이 BankImpl 객체를 BankProxyHandler에 할당, Bank 인터페이스를 사용해 프록시된 인터페이스를 사용해 모델과 로직이 분리된 코드를 작성할 수 있다. 이로써 모델과 로직의 분리를 이뤄낸 코드를 작성할 수 있게 되었다.  

하지만 위와 같은 상대적으로 간단한 경우임에도 불구하고 결과적으로 추가적인 복잡한 코드가 생겼다. 이는 클린코드를 작성하는 데에 걸림돌이 되며 또한 시스템 전반적인 advice를 삽입하는 데에도 부적절하다.

<a name="4-2"></a>
#### 순수 자바 AOP 프레임워크<sup> [7](#fn7)</sup> ####

위 Java Proxy API의 단점들은 Spring, JBoss와 같은 순수 자바 AOP 프레임워크를 통해 해결할 수 있다. 예를 들어 Spring에서는 비지니스 로직을 POJO로 작성해 자신이 속한 도메인에 집중하게 한다. 결과적으로 의존성은 줄어들고 테스트 작성에 필요한 고민도 줄어든다. 이러한 심플함은 user story의 구현과 유지보수, 확장 또한 간편하게 만들어 준다.  

예시를 통해 Spring 프레임워크의 동작 방식에 대해 확인해 보자.  

```java
/* Code 3-2(Listing 11-4): Spring 2.X configuration file */

<beans>
    ...
    <bean id="appDataSource"
        class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close"
        p:driverClassName="com.mysql.jdbc.Driver"
        p:url="jdbc:mysql://localhost:3306/mydb"
        p:username="me"/>
    
    <bean id="bankDataAccessObject"
        class="com.example.banking.persistence.BankDataAccessObject"
        p:dataSource-ref="appDataSource"/>
    
    <bean id="bank"
        class="com.example.banking.model.Bank"
        p:dataAccessObject-ref="bankDataAccessObject"/>
    ...
</beans>
```

Bank객체는 BankDataAccessObject가, BankDataAccessObject는 BankDataSource가 감싸 프록시하는 구조로 되어 각각의 bean들이 <a href="/images/additional references/Chapter 11 - Russian nesting dolls.jpg?raw=true">"러시안 인형"</a>의 한 부분처럼 구성되었다. 클라이언트는 Bank에 접근하고 있다고 생각하지만 사실은 가장 바깥의 BankDataSource에 접근하고 있는 것이다.
<p align="center"><img src="/images/figure 11-3.png" width="500" /></p>

이 프록시된 Bank객체를 생성하는 방법은 아래와 같다.  

```java
/* Code 3-3: Code 3-2의 활용법 */

XmlBeanFactory bf = new XmlBeanFactory(new ClassPathResource("app.xml", getClass()));
Bank bank = (Bank) bf.getBean("bank");
```
위와 같이 최소한의 Spring-specific한 코드만 작성하면 되므로 ***프레임워크와 "거의" decouple된*** 어플리케이션을 작성할 수 있다.  

구조 정의를 위한 xml은 다소 장황하고 읽기 힘들 수는 있지만 Java Proxy보다는 훨씬 간결하다. 이 개념은 아래에 설명할 EJB3의 구조 개편에 큰 영향을 미쳤다. EJB3은 xml와 Java annotation을 사용해 cross-cutting concerns를 정의하고 서포트하게 되었다.  
```java
/* Code 3-4(Listing 11-5): An EBJ3 Bank EJB */

package com.example.banking.model;

import javax.persistence.*;
import java.util.ArrayList;
import java.util.Collection;

@Entity
@Table(name = "BANKS")
public class Bank implements java.io.Serializable {
    @Id @GeneratedValue(strategy=GenerationType.AUTO)
    private int id;
    
    @Embeddable // An object “inlined” in Bank’s DB row
    public class Address {
        protected String streetAddr1;
        protected String streetAddr2;
        protected String city;
        protected String state;
        protected String zipCode;
    }
    
    @Embedded
    private Address address;
    @OneToMany(cascade = CascadeType.ALL, fetch = FetchType.EAGER, mappedBy="bank")
    private Collection<Account> accounts = new ArrayList<Account>();
    public int getId() {
        return id;
    }
    
    public void setId(int id) {
        this.id = id;
    }
    
    public void addAccount(Account account) {
        account.setBank(this);
        accounts.add(account);
    }
    
    public Collection<Account> getAccounts() {
        return accounts;
    }
    
    public void setAccounts(Collection<Account> accounts) {
        this.accounts = accounts;
    }
}
```
위와 같이 EJB3은 EJB2 보다 훨씬 간결한 코드로 작성할 수 있게 되었다. 몇몇 세부 속성들은 annotation으로 클래스 내에 정의되어 있지만 annotation을 벗어나진 않기 때문에 이전보다 더 깨끗하고 명료한 코드를 산출하며 그로 인해 유지보수, 테스트하기 편한 장점을 갖게 되었다.  

<a name="4-3"></a>
#### AspectJ ####
AspectJ는 AOP를 실현하기 위한 full-featured tool이라 일컬어진다. 8~90%의 경우에는 Spring AOP와 JBoss AOP로도 충분하지만 AspectJ는 훨씬 강력한 수준의 AOP를 지원한다. 다만 이를 사용하기 위해 새로운 툴, 언어 구조, 관습적인 코드를 익혀야 한다는 단점도 존재한다.(최근 소개된 "annotation-form AspectJ"로 인해 적용에 필요한 노력은 많이 줄어들었다고 한다.)  
AOP에 대한 더 자세한 내용은 [AspectJ], [Colyer], [Spring]를 참조하기 바란다.  

<a name="5"></a>
## 시스템 아키텍쳐를 테스트 주도하라(Test Drive the System Architecture) ##
코드 레벨에서부터 아키텍쳐와 분리된(decouple된) 프로그램 작성은 당신의 아키텍쳐를 test drive하기 쉽게 만들어 준다. 처음에는 작고 간단한 구조에서 시작하지만 필요에 따라 새로운 기술을 추가해 정교한 아키텍쳐로 진화할 수 있다. 또한 decouple된 코드는 user story, 규모 변화와 같은 변경사항에 더 빠르게 대처할 수 있도록 우리를 도와 준다.
도리어 BDUF(Big Design Up First)와 같은 방식은 변경이 생길 경우 기존의 구조를 버려야 한다는 심리적 저항, 아키텍쳐에 따른 디자인에 대한 고민 등 변화에 유연하지 못한 단점들을 가져오게 된다.  
초기 EJB와 같이 너무 많은 엔지니어링이 가미되어 많은 concern들을 묶어버리지 않으며 오히려 많은 부분들을 숨기는 것이 아름다운 구조를 가져올 것이다.  

> 이상적인 시스템 아키텍쳐는 각각 POJO로 만들어진 모듈화된 관심 분야 영역(modularized domains of concern)으로 이루어져야 한다. 다른 영역끼리는 Aspect의 개념을 사용해 최소한의 간섭으로 통합되어야 한다. 이러한 아키텍쳐는 코드와 마찬가지로 test-driven될 수 있다.

<a name="6"></a>
## 의사 결정의 최적화하라 ##
충분히 큰 시스템에서는(그것이 도시이건 소프트웨어이건) 한 사람이 모든 결정을 내릴 수는 없다. 결정은 최대한 많은 정보가 모일 때까지 미루고 시기가 되었을 경우 해당 파트의 책임자(여기서는 사람이 아닌 모듈화된 컴포넌트를 뜻한다)에게 맡기는 것이 불필요한 고객 피드백과 고통을 덜어줄 것이다.  
> 모듈화된 관심 분야로 이루어진 POJO 시스템의 (변화에 대한)민첩함은 가장 최신의 정보를 가지고 적시에 최적의 선택을 할 수 있게 도와준다. 결정에 필요한 복잡도 또한 경감된다.  

<a name="7"></a>
## 표준은 확실한 이득을 가져올 경우 추가하라 ##
많은 소프트웨어 팀들은 훨씬 가볍고 직관적인 디자인이 가능했음에도 불구하고 그저 표준이라는 이유만으로 EJB2 구조를 사용했다. **표준에 심취해 "고객을 위한 가치 창출"이라는 목표를 잃어 버렸기 때문이다.**  
> 표준은 아이디어와 컴포넌트의 재사용, 관련 전문가 채용, 좋은 아이디어의 캡슐화, 컴포넌트들의 연결을 쉽게 도와 준다. 하지만 종종 표준을 만드는 데에 드는 시간은 납품 기한을 맞추기 어렵게 만들고, 혹은 최초에 제공하려던 기능과 동떨어지게 되기도 한다.

<a name="8"></a>
## 시스템에는 DSL(도메인 영역 언어)이 필요하다 ##
좋은 DSL은 도메인 영역의 개념과 실제 구현될 코드 사이의 "소통의 간극"을 줄여 도메인 영역을 코드 구현으로 번역하는 데에 오역을 줄여준다. DSL을 효율적으로 사용하면 코드 덩어리와 디자인 패턴의 추상도를 높여 주며 그에 따라 코드의 의도를 적절한 추상화 레벨에서 표현할 수 있게 해준다.  
> DSL은 "모든 단계에서의 추상화"와 "모든 도메인의 POJO화"를 고차원적 규칙과 저차원적 디테일 전반에 걸쳐 도와 준다.

<a name="9"></a>
## 결론 ##
코드뿐만이 아니라 시스템 또한 깨끗해야 한다. 침략적인(invasive) 아키텍쳐는 도메인 로직에 피해를 주고 신속성에도 영향을 준다. 도메인 로직이 모호해지면 버그는 숨기 쉬워지고 기능 구현은 어려워 진다. 신속성이 침해되면 생산성이 저해되고 TDD로 인한 이득 또한 얻을 수 없다.  
의도는 모든 레벨의 추상화에서 명확해야 한다. 이는 각각의 concern들을 POJO로 작성된 코드와 aspect-like 메커니즘을 통해 구성할 때 비로소 실현될 수 있다.  
당신이 시스템을 디자인하든 독자적인 모듈을 디자인하든, *동작하는 범위에서 가장 간단한 것*을 사용하는 것을 잊어서는 안된다.

---

#### 참조 ####  
##### <a name="fn1">1. Test Double</a>  
https://en.wikipedia.org/wiki/Test_double    

##### <a name="fn2">2. Abstract Factory Pattern</a>
A factory is the location of a concrete class in the code at which objects are constructed. The intent in employing the pattern is to insulate the creation of objects from their usage and to create families of related objects without having to depend on their concrete classes.[2]This allows for new derived types to be introduced with no change to the code that uses the base class.
Use of this pattern makes it possible to interchange concrete implementations without changing the code that uses them, even at runtime. However, employment of this pattern, as with similar design patterns, may result in unnecessary complexity and extra work in the initial writing of code. Additionally, higher levels of separation and abstraction can result in systems which are more difficult to debug and maintain.<br />
참조: https://en.m.wikipedia.org/wiki/Abstract_factory_pattern  

##### <a name="fn3">3. Dependency Injection and Inversion of Control</a>  
http://greatkim91.tistory.com/41   

##### <a name="fn4">4. AOP</a>  
읽기 좋은 정리: http://isstory83.tistory.com/90<br/>  
그림: http://addio3305.tistory.com/86<br/>    
사전적 설명(개요): http://seulkom.tistory.com/18

##### <a name="fn5">5. 해당 섹션은 독자의 이해를 돕기 위해 역자 임의로 추가된 섹션</a>  

##### <a name="fn6">6. Java Proxy API sample</a>
https://github.com/crowjdh/DatabaseProxySample  

##### <a name="fn7">7. Spring Framework example</a>
https://github.com/crowjdh/DatabaseProxyUsingAOPSample   
