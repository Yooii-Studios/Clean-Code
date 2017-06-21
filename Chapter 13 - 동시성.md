## 목차 ##
- [서론](#1)
- [동시성이 필요한 이유?](#2)
 - [미신과 오해](#2-1)
- [난관](#3)
- [동시성 방어 원칙](#4)
 - [단일 책임 원칙(Single Responsibility Principle, SRP)](#4-1)
 - [따름 정리(Corollary): 자료 범위를 제한하라](#4-2)
 - [따름 정리: 자료 사본을 사용하라](#4-3)
 - [따름 정리: 스레드는 가능한 독립적으로 구현하라](#4-4)
- [라이브러리를 이해하라](#5)
 - [스레드 환경에 안전한 컬렉션](#5-1)
- [실행 모델을 이해하라](#6)
 - [생산자-소비자(Producer-Consumer)](#6-1)
 - [읽기-쓰기(Readers-Writers)](#6-2)
 - [식사하는 철학자들(Dining Philosophers)](#6-3)
- [동기화하는 메서드 사이에 존재하는 의존성을 이해하라](#7)
- [동기화하는 부분을 작게 만들어라](#8)
- [올바른 종료 코드는 구현하기 어렵다](#9)
- [스레드 코드 테스트하기](#10)
 - [말이 안 되는 실패는 잠정적인 스레드 문제로 취급하라](#10-1)
 - [다중 스레드를 고려하지 않은 순차 코드부터 제대로 돌게 만들자](#10-2)
 - [다중 스레드를 쓰는 코드 부분을 다양한 환경에 쉽게 끼워 넣을 수 있게 스레드 코드를 구현하라](#10-3)
 - [다중 스레드를 쓰는 코드 부분을 상황에 맞게 조율할 수 있게 작성하라](#10-4)
 - [프로세서 수보다 많은 스레드를 돌려보라](#10-5)
 - [다른 플랫폼에서 돌려보라](#10-6)
 - [코드에 보조 코드instrument를 넣어 돌려라. 강제로 실패를 일으키게 해보라](#10-7)
   - [직접 구현하기](#10-7-1)
    - [자동화](#10-7-2)
- [결론](#11)

---

> “Objects are abstractions of processing. Threads are abstractions of schedule.”
<p align="right">—James O. Coplien</p>

<a name="1"></a>
## 서론 ##
단일 스레드에서 동작하는 코드는 작성하기 쉽다. 잘 동작하는 "것 처럼" 보이는 멀티 스레드 코드를 작성하기도 쉽다.

본 챕터에서는 concurrent 프로그래밍의 필요성, 어려움에 대해 논의하고 그것에 대한 해결 방안과 "clean concurrent code"를 작성하는 방법, 테스트 방법을 소개하고자 한다.

**역주: Concurrency<sup> [1](#fn1)</sup>는 원문의 뉘앙스를 해쳐 발생하는 오해를 줄이기 위해 번역하지 않음.**

<a name="2"></a>
## 동시성이 필요한 이유? ##
Concurrency는 단일 스레드에서 엮여 있던 "무엇을 할 것인가"와 "언제 끝날 것인가"간의 의존성을 해소시켜 준다. 이는 처리량과 구조 개선에 도움을 줄 수 있다.

구조 개선의 좋은 예는 Servlet 모델일 것이다. *이론적으로,* Servlet 개발자는 요청을 개별적으로 처리하는 데에만 신경을 쓰며 요청 큐를 직접 관리하는 부담을 덜 수 있다. 물론, Servlet이 제공하는 의존성의 해소는 완벽하지 않지만 Servlet이 제공하는 구조적인 이점은 그 자체로 가치가 있다.

처리량 또한 향상될 수 있다. 한 유저의 요청을 처리하는 데에 1초가 필요한 시스템을 생각해 보자. 이 시스템은 적은 유저가
사용할 경우 그럭저럭 괜찮은 퍼포먼스를 보여줄 것이다. 하지만 유저가 늘어남에 따라 모든 유저는 자신보다 먼저 도착한 요청이
끝날 때까지 기다려야만 한다. 이러한 경우 concurrency가 여러 유저를 동시에 처리함으로써 처리량을 향상시킬 수 있다.


<a name="2-1"></a>
#### 미신과 오해 ####
아래는 잘 알려진 미신과 오해에 대한 설명이다.
- Concurrency는 항상 퍼포먼스를 향상시킨다.  
  => Concurrency는 여러 스레드 혹은 여러 프로세서가 대기 시간을 공유할 수 있는 경우에만 퍼포먼스를 향상시킨다. 하지만 이러한 경우는 드물다.
- Concurrent program 작성은 시스템의 디자인을 변경시키지 않는다.  
  => "무엇"과 "언제"를 분리하는 작업은 보통 시스템의 구조에 큰 영향을 미친다.
- Web나 EJB와 같은 컨테이너를 사용한다면 Concurrency 문제들은 신경쓸 필요가 없다.  
  => 컨테이너가 어떤 일을 하는가에 대해 알아야 하며 concurrent update, 데드락을 해결하는 방법을 알아야 한다.

위에 덧붙여 아래의 사항도 숙지하자.
- Concurrency는 퍼포먼스, 코드 작성 양쪽 모두에 약간의 오버헤드를 일으킨다.
- 간단한 문제 해결을 위한 Concurrency는 간단하지 않다.
- Concurrency 관련 버그는 재현하기 어렵기 때문에 종종 one-off<sup> [2](#fn2)</sup>로 취급된다.
- Concurrency 문제에는 보통 근본적인 디자인 개편이 필요하다.

<a name="3"></a>
## 난관 ##
```java
/* Code 1-1 */
    
public class ClassWithThreadingProblem {
    private int lastIdUsed;
    
    public ClassWithThreadingProblem(int lastIdUsed) {
        this.lastIdUsed = lastIdUsed;
    }
    
    public int getNextId() {
        return ++lastIdUsed;
    }
}

public static void main(String args[]) {
    final ClassWithThreadingProblem classWithThreadingProblem = new ClassWithThreadingProblem(42);
    
    Runnable runnable = new Runnable() {
        public void run() {
            classWithThreadingProblem.getNextId();
        }
    };
    
    Thread t1 = new Thread(runnable);
    Thread t2 = new Thread(runnable);
    t1.start();
    t2.start();
}
```
위 코드가 만들 수 있는 결과는 총 3가지 이다.
- t1이 43을, t2가 44를 가져간다. lastIdUsed는 44이다.(O)
- t1이 44을, t2가 43를 가져간다. lastIdUsed는 44이다.(O)
- t1이 43을, t2가 43를 가져간다. lastIdUsed는 43이다.(X)

위의 getNextId() 메서드는 8개의 자바 byte-code로 변환되며, 이를 두 스레드에서 실행하게 되면 총 12,870개의 코드 조합을 낼 수 있다. 그 중 *얼마 안 되는* 몇몇 조합이 위의 3가지 결과 중 마지막 결과를 낳게 된다.<sup> [3](#fn3)</sup>

<a name="4"></a>
## 동시성 방어 원칙 ##

<a name="4-1"></a>
#### 단일 책임 원칙(Single Responsibility Principle, SRP)<sup> [4](#fn4)</sup> ####
Concurrency 디자인은 그 자체로 충분히 복잡하기 때문에 변경이 발생할 수 있다. 따라서 concurrency 관련 코드는 분리되어야 한다.  
하지만 concurrency 구현은 다른 코드의 변화까지 가져오는 경우가 잦다.  
아래의 사항들을 숙지하자.  

- *Concurrency 관련 코드*는 *개발*, 변경, 튜닝시 다른 코드와 분리된 생명주기를 가진다.
- *Concurrency 관련 코드*는 그 자체가 가지는 어려움(풀기 힘든 문제)이 있다.
- 잘못 작성된 concurrency 코드는 여러 문제를 발생시킬 수 있으며, 이는 추가적인 코드 없이 해결되기 힘들다.

**추천**: *Concurrency 관련 코드는 다른 코드들과 분리하라.*

<a name="4-2"></a>
#### 따름 정리(Corollary): 자료 범위를 제한하라 ####
위 "무엇이 어려운가?"에서 본 것 처럼 공유 객체를 두 스레드에서 수정하는 중 간섭이 발생할 수 있으며 이는 예기치 못한 결과를 야기할 수 있다.
이러한 *critical section*<sup> [5](#fn5)</sup>을 보호하는 한 가지 방법은 synchronized 키워드를 사용하는 것이다.
*Critical section*의 수는 가능한한 적게 만들어야 하며 이를 어길 경우 아래와 같은 문제가 발생하기 쉽게 된다.
- 한두 군데를 보호하는 것을 까먹기 쉬우며 이로 인해 해당 자원을 수정하는 모든 코드를 망가트리게 된다.
- 모든 곳이 보호되었는지 파악하기 위해 중복적인 노력이 필요하게 된다.
- 이미 찾기 어려운 문제의 근원을 더 찾기 어렵게 만들게 된다.

**추천**: *데이터 캡슐화를 가슴 깊이 새기며, 공유될 만한 자원에 접근하는 부분(코드)을 극도로 줄여라.*

<a name="4-3"></a>
#### 따름 정리: 자료 사본을 사용하라 ####
공유 자원 문제를 해결하는 좋은 방법중 하나는 애초에 공유 자원을 사용하지 않는 것이다. 읽기 전용으로 사용될 경우 자원의 복사본을 사용하게 하는 방법이 있다. 경우에 따라서는 복사본을 여러 스레드에 전달, 작업을 수행하고 결과를 단일 스레드에서 수집해 사용하는 것도 가능하다.

객체의 복사에 드는 비용을 걱정할 수도 있다. 혹은, 이 문제가 "진짜 문제가 되는지" 조사해 보는 방법도 있다. 하지만 객체의 복사본을 사용함으로써 동기화를 피할 수 있다면, 객체 생성 및 GC에 드는 비용은 공유 자원 동기화에 필요한 비용 보다 일반적으로 적은 비용으로 문제를 해결하게 해 준다.(객체 복사 cost < 공유 자원 동기화 cost)

<a name="4-4"></a>
#### 따름 정리: 스레드는 가능한 독립적으로 구현하라 ####
스레드 코드를 공유 자원을 사용하지 않는 독립된 세계로 만든다면 동기화 문제는 없어지게 된다.

HttpServlet을 생각해 보라. HttpServlet을 상속받는 클래스는 doGet, doPost와 같은 메서드에서 필요한 파라미터를 받아 처리한다.
이는 각 Servlet이 각자의 세계에 있는 것처럼 작동하게 도와주며, 지역 변수를 사용하는 한 동기화 문제는 발생하지 않게 된다.
물론 대부분의 Servlet들은 데이터베이스 연결과 같은 공유 자원이 필요하긴 하다.

**추천**: *데이터를 독립적인 스레드-더 나아가 각각의 프로세서-에서 사용될 수 있게 구분하라.*

<a name="5"></a>
## 라이브러리를 이해하라 ##
자바 5버전 이상에서 스레드 관련 코드 작성시 아래의 사항들을 숙지하자.

- 자바에서 제공하는 thread-safe 컬랙션을 사용하라.
- 연관이 없는 태스크들을 수행시 executor 프레임워크를 사용하라.
- 가능하면 nonblocking 방법을 사용하라.
- 몇몇 라이브러리 클래스들은 thread-safe하지 않다.

<a name="5-1"></a>
#### 스레드 환경에 안전한 컬렉션 ####
java.util.concurrent 패키지는 멀티 스레드 환경에서 사용할 수 있는 컬랙션들을 제공한다. ConcurrentHashMap의 경우에는 일반 HashMap보다 대부분의 상황에서 더 좋은 퍼포먼스를 제공한다. 만약 배포 환경이 자바 5버전 이상이라면 이 패키지를 활용하자.

아래와 같은 고급 concurrency 디자인 구현을 위한 컴포넌트들도 숙지하자.

| Name            | Description                                                         |
| :-------------- | :------------------------------------------------------------------ |
| ReentrantLock   | 한 메서드에서 잠그고 다른 메서드에서 해제될 수 있는 lock이다.       |
| Semaphore       | 전통적인 세마포어(갯수를 셀 수 있는 lock)의 구현체이다.             |
| CountDownLatch  | 기다리는 모든 스레드들을 해제하기 전 특정 횟수의 이벤트가 발생하는 것을 기다리게 할 수 있는 lock이다. 모든 스레드가 거의 동시에 시작될 수 있게 도와줄 수 있다. |

**추천**: *당신에게 맞는 클래스를 살펴보라. 자바의 경우 java.util.concurrent, java.util.concurrent.atomic, java.util.concurrent.locks를 살펴보라.*

<a name="6"></a>
## 실행 모델을 이해하라 ##
실행 모델에 대해 이야기하기 위해 필요한 기본적인 용어를 먼저 알아 보자.

| Name                         | Description  |
| :--------------------------- | :----------- |
| Bound Resources              | Concurrent 환경에서 사용되는 고정된 크기의 자원이다. 예시로 데이터베이스 연결, 고정된 크기의 읽기/쓰기 버퍼가 있다. |
| Mutual Exclusion             | 한 시점에 공유 자원에 접근할 수 있는 스레드는 단 하나이다. |
| Starvation                   | 한 스레드 혹은 스레드의 그룹이 긴 시간 혹은 영원히 작업을 수행할 수 없게 된다. 작업의 우선권을 가지는 수행 시간이 짧은 스레드가 끝없이 실행된다면 수행 시간이 긴 스레드는 굶게 된다. |
| Deadlock                     | 두 개 이상의 스레드들이 서로의 작업이 끝나기를 기다린다. 각 스레드는 서로가 필요로 하는 자원을 점유하고 있으며 필요한 자원을 얻지 못하는 이상 그 누구도 작업을 끝내지 못하게 된다. |
| Livelock<sup> [6](#fn6)</sup>| 스레드들이 서로 작업을 수행하려는 중 다른 스레드가 작업중인 것을 인지하고 서로 양보한다. 이러한 공명 때문에 스레드들은 작업을 계속 수행하려 하지만 장시간 혹은 영원히 작업을 수행하지 못하게 된다. |

<a name="6-1"></a>
#### 생산자-소비자(Producer-Consumer) ####
한 개 이상의 생산자가 생산한 작업물을 버퍼 혹은 큐에 넣는다. 한 개 이상의 소비자가 버퍼 혹은 큐에서 작업물을 습득, 작업을 마친다. 생산자와 소비자 사이에 있는 큐는 *bound resource*이다. 따라서 생산자는 큐에 남는 공간이 생길 때까지, 소비자는 큐에 작업물이 하나라도 생길 때까지 기다려야 한다. 큐를 통한 생산자와 소비자간의 조율에는 둘 사이의 시그널링이 필요하다. 생산자는 큐에 작업물을 넣고 소비자에게 "큐가 비어있지 않다"는 신호를 보내고 소비자는 큐에서 작업물을 꺼낸 후 "큐가 가득차 있지 않다"는 신호를 보낸다. 그 전까지 둘은 신호를 기다린다.

<a name="6-2"></a>
#### 읽기-쓰기(Readers-Writers) ####
일반적으로 독자를 위한 정보로 사용되며, 가끔 저자에 의해 업데이트되는 공유 자원의 경우 처리량이 문제가 된다. 처리량을 강조해 독자가 상대적인 우선권을 가지게 되면 저자는 기아 상태에 빠지며 공유 자원은 정체된 정보로 가득차게 된다. 반대로 저자가 우선권을 가지면 처리량이 줄어들게 된다. 저자-독자 문제는 이 둘 사이의 균형을 맞추며 concurrent 업데이트를 방지하는 것을 주안점으로 둔다.

<a name="6-3"></a>
#### 식사하는 철학자들(Dining Philosophers) ####
원탁을 둘러싼 여러 명의 철학자들이 있다. 각 철학자의 왼쪽에 포크가 놓여 있으며 테이블의 중앙에 큰 스파게티 한 그릇이 놓여 있다. 그들은 배가 고파지기 전까지 각자 생각을 하며 시간을 보낸다. 배가 고파지면 그들은 자신의 양쪽에 놓여 있는 포크 2개를 잡고 스파게티를 먹는다.  
철학자는 포크 2개가 있어야만 스파게티를 먹을 수 있다. 그렇지 않다면 옆 사람이 포크를 다 사용하기 전까지 기다려야 한다. 스파게티를 먹은 철학자는 다시 배가 고파질 때까지 포크를 놓고 있게 된다.  
위 상황에서 철학자를 스레드로, 포크를 공유 자원으로 바꾸게 되면 이는 자원을 놓고 경쟁하는 프로세스와 비슷한 상황이 된다. 잘 설계되지 않은 시스템은 deadlock, livelock, 처리량 문제, 효율성 저하 문제에 맞닥뜨리기 쉽다.

당신이 맞닥뜨릴 대부분의 concurrent관련 문제들은 이 세 가지 문제의 변형일 가능성이 높다. 이 알고리즘들을 공부하고 스스로 해법을 작성함으로써 이와 같은 문제들을 직면하더라도 의연하게 대처할 수 있도록 하자.

**추천**: *위 기본적인 알고리즘들과 그 해법을 익히자.*

<a name="7"></a>
## 동기화하는 메서드 사이에 존재하는 의존성을 이해하라 ##
동기화된 메서드 간의 의존성은 concurrent 코드에서 사소한 버그를 일으킬 수 있다. 자바는 synchronized라는 "메서드 하나를 보호하는 노테이션"을 제공한다. 하지만 한 클래스에 두 개 이상의 synchronized 메서드가 존재하면 문제를 일으킬 수도 있다.

**추천**: *공유된 객체의 두 메서드 이상을 사용하는 것을 피하라.*

만약 위 추천을 따를 수 없는 상황이라면 아래의 세 방법을 고려해 보라.

**클라이언트 기반 잠금(Client-Based Locking)**: 클라이언트가 첫 메서드를 부르기 이전부터 마지막 메서드를 부른 다음까지 서버를 잠근다. (역주: 공유 객체를 사용하는 코드에서 공유 객체를 잠그는 것이다.)  
=> *Bad: 서버를 사용하는 모든 클라이언트 코드에서 lock이 필요하게 되며 이는 유지보수 및 디버깅에 필요한 비용을 상승시킨다.*

**서버 기반 잠금(Server-Based Locking)**: 서버 내에서 서버(자신)을 잠그고 모든 동작을 수행한 후 잠금을 푸는 메서드를 제공한다. 클라이언트에게는 새로운 메서드를 제공한다. (역주: 공유 객체에 새로운 메서드를 작성하고 잠금이 필요한 동작 전체를 수행하게 하는 것이다.)  
=> *Good: Critical section에 접근하는 코드를 최소화해 위 4-2에 부합한다.*

**중계된 서버(Adapted Server)**: 잠금을 수행하는 중계자를 작성한다. 이는 기본적으로 서버 기반 잠금이지만 기존의 서버를 변경할 수 없는 상황에 사용할 수 있는 방법이다.(역주: 서드 파티 라이브러리를 사용한다고 생각하면 쉬울 것이다.)  
=> *Good: 서버 기반 잠금 방식을 사용할 수 없는 경우에 사용하자.*

아래는 위의 내용에 대한 예제이다.

```java
/* Code 2-1: 문제가 되는 상황 */

public class IntegerIterator implements Iterator<Integer> {
    private Integer nextValue = 0;
    
    public synchronized boolean hasNext() {
        return nextValue < 100000;
    }
    
    public synchronized Integer next() {
        if (nextValue == 100000)
            throw new IteratorPastEndException();
        return nextValue++;
    }
    
    public synchronized Integer getNextValue() {
        return nextValue;
    }
}

// Shared Resource
IntegerIterator iterator = new IntegerIterator();

// Threaded-Code
while(iterator.hasNext()) {
    // nextValue가 99999인 상황에서 두 스레드에서 순차적으로 while(iterator.hasNext())를 호출하게 되면
    // 두 스레드 모두 while문 안으로 진입하게 된다. 이는 예상되지 않은 결과이다.
    int nextValue = iterator.next();
    // do something with nextValue
}
```

```java
/* Code 2-2: Client-Based Locking */

// Shared Resource
IntegerIterator iterator = new IntegerIterator();

// Threaded-Code
while (true) {
    int nextValue;
    synchronized (iterator) {
        if (!iterator.hasNext())
            break;
        nextValue = iterator.next();
    }
    doSometingWith(nextValue);
}
```

```java
/* Code 2-3: Server-Based Locking */

public class IntegerIteratorServerLocked {
    private Integer nextValue = 0;
    
    public synchronized Integer getNextOrNull() {
        if (nextValue < 100000)
            return nextValue++;
        else
            return null;
    }
}

// Shared Resource
IntegerIterator iterator = new IntegerIterator();

// Threaded-Code
while (true) {
    Integer nextValue = iterator.getNextOrNull();
    if (next == null)
        break;
    // do something with nextValue
}
```

```java
/* Code 2-4: Adapted Server */

public class ThreadSafeIntegerIterator {
    private IntegerIterator iterator = new IntegerIterator();
    
    public synchronized Integer getNextOrNull() {
        if(iterator.hasNext())
            return iterator.next();
        return null;
    }
}

// Threaded-Code는 위 Code 2-3과 동일
```

<a name="8"></a>
## 동기화하는 부분을 작게 만들어라 ##
Synchronized로 수행되는 잠금은 딜레이와 오버헤드를 만들기 때문에 "비싼 수행"으로 간주되며 가능한 한 작게 만들어야 한다. 반면 critical section은 꼭 보호되어야 한다.

**추천**: *동기화된 영역은 최대한 작게 만들어라.*

<a name="9"></a>
## 올바른 종료 코드는 구현하기 어렵다 ##
"항상 살아 있어야 하는 코드"의 작성은 "잠시 동작하고 조용히 끝나는" 코드의 작성과는 다르다.

조용히 끝나는 코드는 작성하기 어렵다. 이는 보편적으로 "오지 않을 신호"를 기다리는 쓰레드의 데드락을 포함한다.

데드락에 걸린 자식 스레드의 수행이 끝나길 기다리는 부모 스레드의 경우를 생각해 보라. 자식은 데드락에 걸려 멈춰 있고 부모는 이를 끝없이 기다리게 된다.

이와 같은 코드를 작성할 경우 정상적인 종료가 이루어질 때까지 많은 시간이 소요될 것을 상정해야 한다.

**추천**: *개발 초기에 시스템 종료에 대해 고민하고 구현하라. 이 작업은 생각보다 오래 걸릴 것이다. 기존에 구현한 알고리즘을 리뷰하는 것도 필요하다.*

<a name="10"></a>
## 스레드 코드 테스트하기 ##
테스트는 정확성을 보장하지 않으며 "코드가 제대로 작성되었는가"를 증명할 수 없다.
다만 잘 작성된 테스트는 위험을 최소화할 수 있다. 이는 멀티 스레드 상황에서는 훨씬 더 복잡해 진다.

**추천**: *문제를 발생시킬 만한 테스트를 작성하고 여러 프로그램 설정과 시스템 설정, 부하 하에서 자주 수행하라. 테스트가 한번이라도 실패한다면 원인을 분석하라. 한번 더 테스트를 실행해 성공했다고 해서 이전의 실패를 무시하지 마라.*

이는 아래와 같이 분류할 수 있다.

 - 말이 안 되는 실패는 잠정적인 스레드 문제로 취급하라
 - 다중 스레드를 고려하지 않은 순차 코드부터 제대로 돌게 만들자
 - 다중 스레드를 쓰는 코드 부분을 다양한 환경에 쉽게 끼워 넣을 수 있게 스레드 코드를 구현하라
 - 다중 스레드를 쓰는 코드 부분을 상황에 맞게 조율할 수 있게 작성하라
 - 프로세서 수보다 많은 스레드를 돌려보라
 - 다른 플랫폼에서 돌려보라
 - 코드에 보조 코드instrument를 넣어 돌려라. 강제로 실패를 일으키게 해보라

<a name="10-1"></a>
#### 말이 안 되는 실패는 잠정적인 스레드 문제로 취급하라 ####
멀티 스레드 코드는 일반적으로 발생할 리 없어 보이는 문제를 발생시킨다. (저자를 포함한)대부분의 개발자는 이러한 문제를 직관적으로 파악하지 못한다. 또한 이는 매우 드물게 발생해 개발자들을 좌절하게 만든다. 그래서 개발자들은 이러한 문제들을 우주선(宇宙線), 하드웨어 버그, 혹은 이러한 류의 one-off로 치부한다. 제일 좋은 방향은 one-off는 없다고 판단하는 것이다. 이러한 one-off들이 무시될 수록 더 많은 코드들이 이미 문제가 있는 시스템에 추가되게 될 뿐이다.

**추천**: *시스템 오작동을 one-off로 판단해 무시하지 말라.*

<a name="10-2"></a>
#### 다중 스레드를 고려하지 않은 순차 코드부터 제대로 돌게 만들자 ####
당연한 말이지만 거듭 강조할 만큼 중요한 이야기이다. 스레드 밖에서 잘 동작하는 코드를 먼저 작성하라. 이는 스레드에서 사용될 POJO를 뜻한다. POJO는 스레드와 연관이 없어 스레드 밖에서도 테스트할 수 있다. 시스템은 가능한 한 POJO로 작성하는 것이 좋다.

**추천**: *스레드 관련 버그와 그렇지 않은 버그를 동시에 잡으려 하지 마라. 작성한 코드가 스레드 밖에서 잘 작동하는지 먼저 체크하라.*

<a name="10-3"></a>
#### 다중 스레드를 쓰는 코드 부분을 다양한 환경에 쉽게 끼워 넣을 수 있게 스레드 코드를 구현하라 ####
Concurrency 지원 코드를 아래와 같이 여러 설정으로 실행될 수 있게 만들어라.

- 단일 스레드, 여러 스레드 환경에서 동작하게 구현
- 실제 사용될 객체 혹은 Test Double<sup> [7](#fn7)</sup>과 상호작용할 수 있는 스레드 코드로 구현
- 수행 속도를 조절할 수 있는 Test Double을 구현
- 지정된 횟수만큼 반복 수행할 수 있게 구현

**추천**: *스레드 기반 코드를 여러 환경에서 실행할 수 있게 하라.*

<a name="10-4"></a>
#### 다중 스레드를 쓰는 코드 부분을 상황에 맞게 조율할 수 있게 작성하라 ####
스레드 관련 코드의 적절한 균형을 맞추는 작업은 보통 시행착오를 필요로 한다. 여러 환경에서 시스템의 퍼포먼스를 테스트할 수 있는 방법을 개발 초기에 강구하라. 실행할 스레드 갯수를 쉽게 변경할 수 있게 작성하라. 이를 시스템이 동작하는 도중에 변경할 수 있게 하는 것을 고려해 보라. 처리량과 시스템 활용도를 기준으로 스스로를 조정할 수 있게 하는 것을 고려해 보라.

<a name="10-5"></a>
#### 프로세서 수보다 많은 스레드를 돌려보라 ####
시스템이 작업을 전환할 때에도 문제는 발생한다. 작업 전환을 빈번히 발생하게 하기 위해 프로세서 수보다 많은 스레드를 실행해 보라. 작업 전환이 잦을수록 빠뜨린 critical section이나 dead lock을 찾을 확률이 높아지게 된다.

<a name="10-6"></a>
#### 다른 플랫폼에서 돌려보라 ####
우리(저자)는 2007년 중순 concurrent 프로그래밍 강좌를 개발했다. 강좌의 개발은 OSX에서 진행되었으며 시연은 VM상의 Windows XP에서 진행되었다. 하지만 실패를 시연하기 위해 작성된 테스트는 OSX에서는 자주 발생했지만 Windows XP에서는 OSX에서만큼 자주 발생하지 않았다.

우리는 이로 인해 서로 다른 운영체제는 상이한 스레딩 정책을 가지며 코드의 실행에 영향을 미친다는 것을 다시 한번 깨닫게 되었다. 멀티 스레드 코드는 실행 환경에 따라 다르게 동작한다. 따라서 당신은 모든 잠재적 배포 환경에 대해 테스트를 수행해야 한다.

**추천**: 스레드 관련 코드를 이른 시기에, 빈번한 주기로 모든 타겟 플랫폼에서 수행하라.

<a name="10-7"></a>
#### 코드에 보조 코드instrument를 넣어 돌려라. 강제로 실패를 일으키게 해보라 ####
스레드 관련 문제는 수많은 실행 경로중 얼마 안되는 확률로 발생하기 때문에 드물게 발생하며 재현하기 어렵다. 이 실행 경로를 조작해 스레드 문제가 발생할 확률을 높이는 code instrumentation에는 두 가지 방법이 있다.

- 직접 구현하기
- 자동화

<a name="10-7-1"></a>
###### 직접 구현하기 ######
이는 Object.wait(), Object.sleep(), Object.yield(), Object.priority()등의 메서드를 사용해 실행 경로를 변경함으로써 코드의 문제를 발견하는 방법이다.

```java
/* Code 3-1 */

public synchronized String nextUrlOrNull() {
    if(hasNext()) {
        String url = urlGenerator.next();
        Thread.yield();
        // inserted for testing.
        updateHasNext();
        return url;
    }
    return null;
}
```

yield() 메서드를 호출함으로써 코드의 실행 경로를 변경할 수 있다. 만약 위 코드에서 문제가 발생한다면 이는 yield()를 추가해 생긴 문제가 아니라 이미 존재하던 문제를 명백히 만든것 뿐이다.

하지만 이 방법에는 몇 가지 문제가 있다.
- 테스트할 부분을 직접 찾아야 한다.
- 어디에 어느 메서드를 호출해야 할지 알기 어렵다.
- 이와 같은 코드를 제품에 포함해 배포하는 것은 불필요하게 퍼포먼스를 저하시킬 뿐이다.
- Shotgun approach<sup> [8](#fn8)</sup>이기 때문에 반드시 문제가 발생한다는 보장을 얻을 수 없다.

우리는 실제 제품에 포함되지 않으며 여러 조합으로 실행해 에러를 찾기 쉽게 만들 방법이 필요하다.

이를 위해서는 시스템을 최대한 POJO 단위로 나눠 instrument code를 삽입할 부분을 찾기 쉽게 하고 여러 정책에 따라 sleep, yield등을 삽입할 수 있게 해야 한다.

<a name="10-7-2"></a>
###### 자동화 ######
위와 다르게 Aspect-oriented Framework, CGLib, ASM등을 통해 프로그램적으로 코드를 조작할 수도 있다. 아래의 예를 보자.

```java
/* Code 4-1 */

public class ThreadJigglePoint {
    public static void jiggle() { }
}

public synchronized String nextUrlOrNull() {
    if(hasNext()) {
        ThreadJiglePoint.jiggle();
        String url = urlGenerator.next();
        ThreadJiglePoint.jiggle();
        updateHasNext();
        ThreadJiglePoint.jiggle();
        return url;
    }
    return null;
}
```

위와 같이 구현한 후 간단한 Aspect<sup> [9](#fn9)</sup>를 이용해 '아무 것도 안하기', 'sleep', 'yield'등을 무작위로 선택하게 할 수 있다.

혹은 ThreadJigglePoint가 두 가지 구현을 가지게 할 수도 있다. 첫 번째 구현은 배포용 코드를 위한 '아무 것도 안하기'를 수행하며 두 번째 구현은 'sleep, yield, 아무 것도 안하기' 중의 하나를 무작위로 선택하는 것이다. 다소 간단하긴 하지만 좀 더 정교한 툴을 사용하는 대신 이 정도로 구현하는 것도 적절한 선택일 것이다.

혹은 이와 비슷한 작업을 수행해 주는 IBM에서 개발한 ConTest라는 툴도 있다. 이는 수행시마다 다른 순서로 스레드를 실행하게 만들어 줌으로써 문제를 발견할 확률을 극적으로 높여준다.

<a name="11"></a>
## 결론 ##
Concurrent 코드는 제대로 작성하기 어렵다. 이해하기 쉬운 코드는 여러 스레드와 공유 자원이 엮이게 되면 끔찍한 결말을 낳게 된다. 당신이 concurrent code를 작성하게 된다면 엄격한 기준으로 clean하게 작성하라. 그렇지 않으면 찾기 어렵고 빈번하지 않은 오류를 만나게 될 것이다.

최우선적으로 SRP를 숙지하라. 시스템을 최대한 POJO단위로 잘라 스레드 관련 코드와 非 스레드 관련 코드를 나누어라. 스레드 관련 코드를 테스트할 때에는 그 이외의 것들은 제외하고 스레드 관련 문제만 테스트하라. 이는 스레드 관련 문제가 최대한 작은 부분에 집중되게 한다.

한 공유 자원에 대한 멀티 스레드 수행, 공유되는 자원 풀 등 concurrency 문제를 일으킬 수 있는 부분에 대해 인지하라. 깔끔하게 종료되게 하는 문제나 반복문 탈출과 같은 문제는 특히 성가실 수 있다.

라이브러리를 이해하고 기본적인 알고리즘을 이해하라. 라이브러리가 제공하는 기능이 어떻게 문제를 해결하는지 이해하라. 

잠가야 할 필요가 있는 부분을 찾는 방법을 배우고 잠가라. 쓸데 없는 구간을 잠그지 마라. 잠긴 구간에서 또 다른 잠긴 구간을 부르는 것을 기피하라. 이는 "무엇이 공유되고 안되고"에 대한 깊은 이해를 요구한다. 공유 객체의 갯수와 공유 영역을 최소한으로 줄여라. 클라이언트가 공유 객체의 상태(잠금 등)를 관리하는 대신 공유 객체의 디자인을 변경하라.

문제는 돌연 발생할 것이다. 그렇지 않은 문제들은 보통 "한번만 발생하는" 문제로 치부된다. 이러한 one-off들은 보통 시스템에 부하가 걸린 경우, 혹은 무작위로 발생한다. 그러므로 스레드 관련 코드는 여러 설정, 환경에서 반복적이고 지속적으로 수행해 보라.

당신의 코드를 시간을 들여 instrument하게 되면 문제점을 찾을 확률은 높아질 것이다. 직접 코드를 작성할 수도 있고 자동화 툴을 사용할 수도 있다. 출시하기 전까지 최대한 오래 테스트 해야할 것이다.

Clean한 접근 방식을 사용한다면, 제대로 된 코드를 만들어낼 가능성은 급격히 올라갈 것이다.

---

#### 참조
##### <a name="fn1">1. Concurrency</a>
https://en.wikipedia.org/wiki/Concurrency_(computer_science)

##### <a name="fn2">2. One-off</a>
사전적 의미는 "한 번만 일어나는"이며, 여기에서는 "고칠 수 없는"이라는 의미도 포함하고 있다.

##### <a name="fn3">3. 더 자세한 내용은 원문의 [부록 A: Concurrency II]를 참고하길 바란다.</a>

##### <a name="fn4">4. SRP(Single Responsibility Principle)</a>
참조: https://en.wikipedia.org/wiki/Single_responsibility_principle

##### <a name="fn5">5. Critical Section: 둘 이상의 스레드가 동시에 접근해서는 안되는 공유 자원(자료 구조 또는 장치)을 접근하는 코드의 일부</a>
출처: https://ko.m.wikipedia.org/wiki/%EC%9E%84%EA%B3%84_%EA%B5%AC%EC%97%AD

##### <a name="fn6">6. Livelock</a>
실생활에서 발생할 수 있는 상황을 예로 들면 아래와 같다.  
두 사람이 좁은 길목에서 만나 서로 비켜가기 위해 한 쪽으로 걷는다. 하지만 우연히도 두 사람은 계속 같은 방향으로 피하게 된다. 따라서 두 사람 모두 앞으로 진행하지 못하게 된다.  
출처: http://stackoverflow.com/a/6155978/2279149

##### <a name="fn7">7. Test Double: 테스트용으로 만들어진 비교적 단순한 구조를 가지는 객체</a>
https://en.wikipedia.org/wiki/Test_double

##### <a name="fn8">8. Shotgun approach: 산탄총으로 목표를 향해 발사하는 것처럼 되는 대로 시도해 보는 방법</a>
출처: http://dictionary.reference.com/browse/shotgun-approach

##### <a name="fn9">9. Aspect: Chapter 11의 AOP 참조</a>
참조: https://github.com/Yooii-Studios/Clean-Code/blob/master/Chapter%2011%20-%20시스템.md#4-2
