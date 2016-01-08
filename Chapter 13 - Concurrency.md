## 목차 ##
- [서론](#1)
- [왜 Concurrency가 중요한가?](#2)
 - [미신과 오해](#2-1)
- [무엇이 어려운가?](#3)
- [Concurrency 문제 해결을 위한 원칙들](#4)
 - [단일 책임 원칙](#4-1)
 - [Corollary: 데이터를 접근할 수 있는 범위를 제한하라](#4-2)
 - [Corollary: 데이터의 복사본을 사용하라](#4-3)
 - [Corollary: 스레드는 가능한 한 독립적이어야 한다](#4-4)
- [라이브러리를 이해하라](#5)
 - [Thread-Safe한 컬랙션들](#5-1)
- [실행 모델에 대한 이해](#6)
 - [생산자-소비자](#6-1)
 - [저자-독자](#6-2)
 - [철학자들의 저녁식사](#6-3)
- [동기화된 메서드 간의 의존성을 주의하라](#7)
- [제대로 "Shut-Down"하는 코드는 작성하기 어렵다](#8)
- [스레드를 포함하는 코드 테스트하기](#9)
 - [일시적이라고 판단되는 문제를 스레드 관련 이슈의 후보로 생각하라](#9-1)
 - [스레드와 관련없는 코드를 작동하게 하라](#9-2)
 - [스레드와 연관될 코드를 pluggable하게 만들어라](#9-3)
 - [스레드와 연관될 코드를 tunable하게 만들어라](#9-4)
 - [프로세서의 수 보다 많은 수의 스레드를 실행해 보라](#9-5)
 - [다른 플렛폼에서 실행해 보라](#9-6)
 - [문제가 발생하도록 조작해 보라](#9-7)
   - [직접 조작](#9-7-1)
    - [자동화된 조작](#9-7-2)
- [결론](#10)

======================================================

> “Objects are abstractions of processing. Threads are abstractions of schedule.”
<p align="right">—James O. Coplien</p>

<a name="1"></a>
## 서론 ##
단일 스레드에서 동작하는 코드는 작성하기 쉽다. 잘 동작하는 "것 처럼" 보이는 멀티 스레드 코드를 작성하기도 쉽다.

본 챕터에서는 병행성 프로그래밍의 필요성, 어려움에 대해 논의하고 그것에 대한 해결 방안과 "clean concurrent code"를 작성하는 방법, 테스트 방법을 소개하고자 한다.

**역주: Concurrency<sup> [1](#fn1)</sup>는 원문의 뉘앙스를 해쳐 발생하는 오해를 줄이기 위해 번역하지 않음.**

<a name="2"></a>
## 왜 Concurrency가 중요한가? ##
Concurrency는 단일 스레드에서 엮여 있던 "무엇을 할 것인가"와 "언제 끝날 것인가"간의 의존성을 해소시켜 준다.  
이는 처리량과 구조 개선에 도움을 줄 수 있다.

구조 개선의 좋은 예는 Servlet 모델일 것이다. *이론적으로,* Servlet 개발자는 요청을 개별적으로 처리하는 데에만 신경을 쓰며 요청 큐를 직접 관리하는 부담을 덜 수 있다. 물론, Servlet이 제공하는 의존성의 해소는 완벽하지 않지만 Servlet이 제공하는 구조적인 이점은 그 자체로 가치가 있다.

처리량 또한 향상될 수 있다. 한 유저의 요청을 처리하는 데에 1초가 필요한 시스템을 생각해 보자. 이 시스템은 적은 유저가
사용할 경우 그럭저럭 괜찮은 퍼포먼스를 보여줄 것이다. 하지만 유저가 늘어남에 따라 모든 유저는 자신보다 먼저 도착한 요청이
끝날 때까지 기다려야만 한다. 이러한 경우 concurrency가 여러 유저를 동시에 처리함으로써 처리량을 향상시킬 수 있다.


<a name="2-1"></a>
#### 미신과 오해 ####
아래는 잘 알려진 미신과 오해에 대한 설명이다.
- Concurrency는 항상 퍼포먼스를 향상시킨다.  
  Concurrency는 여러 스레드 혹은 여러 프로세서가 wait time을 공유할 수 있는 경우에만 퍼포먼스를 향상시킨다. 하지만 이러한 경우는 드물다.
- Concurrent program 작성은 시스템의 디자인을 변경시키지 않는다.  
  "무엇"과 "언제"를 분리하는 작업은 보통 시스템의 구조에 큰 영향을 미친다.
- Web나 EJB와 같은 컨테이너를 사용한다면 Concurrency 문제들은 신경쓸 필요가 없다.  
  컨테이너가 어떤 일을 하는가에 대해 알아야 하며 concurrent update, 데드락을 해결하는 방법을 알아야 한다.

위에 덧붙여 아래의 사항도 숙지하자.
- Concurrency는 퍼포먼스, 코드 작성 양쪽 모두에 약간의 오버헤드를 일으킨다.
- 간단한 문제 해결을 위한 Concurrency는 간단하지 않다.
- Concurrency 관련 버그는 재현하기 어렵기 때문에 종종 one-off<sup> [2](#fn2)</sup>로 취급된다.
- Concurrency 문제는 보통 근본적인 디자인 개편이 필요로 한다.

<a name="3"></a>
## 무엇이 어려운가? ##
```java
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
- t1이 43을, t2가 44를 가져간다. lastIdUsed는 44이다.
- t1이 44을, t2가 43를 가져간다. lastIdUsed는 44이다.
- t1이 43을, t2가 43를 가져간다. lastIdUsed는 43이다.

위의 getNextId() 메서드는 8개의 자바 byte-code로 변환되며, 이를 두 스레드에서 실행하게 되면 총 12,870개의 코드 조합을 낼 수 있다. 그 중 *얼마 안 되는* 몇몇 조합이 위의 3가지 결과 중 마지막 결과를 낳게 된다.<sup> [3](#fn3)</sup>

<a name="4"></a>
## Concurrency 문제 해결을 위한 원칙들 ##

<a name="4-1"></a>
#### 단일 책임 원칙<sup> [4](#fn4)</sup> ####
Concurrency 디자인은 그 자체로 충분히 복잡하기 때문에 변경이 발생할 수 있다. 따라서 concurrency 관련 코드는 분리되어야 한다.
하지만 concurrency 구현은 다른 코드의 변화까지 가져오는 경우가 잦다.  
아래의 사항들을 숙지하자

- *Concurrency 관련 코드*는 *개발*, 변경, 튜닝시 다른 코드와 분리된 생명주기를 가진다.
- *Concurrency 관련 코드*는 그 자체가 가지는 어려움(풀기 힘든 문제)이 있다.
- 잘못 작성된 concurrency 코드는 여러 문제를 발생시킬 수 있으며, 이는 추가적인 코드 없이 해결되기 힘들다.

**추천**: *Concurrency 관련 코드는 다른 코드들과 분리하라*

<a name="4-2"></a>
#### Corollary: 데이터를 접근할 수 있는 범위를 제한하라 ####
위 3에서 본 것 처럼 공유 객체를 두 스레드에서 수정하는 중 간섭이 발생할 수 있으며 이는 예기치 못한 결과를 야기할 수 있다.
이러한 *critical section*을 보호하는 한 가지 방법은 synchronized 키워드를 사용하는 것이다.
*Critical section*의 수는 가능한한 적게 만들어야 하며 이를 어길 경우 아래와 같은 문제가 발생하기 쉽게 된다.
- 한두 군데를 보호하는 것을 까먹기 쉬우며 이로 인해 해당 자원을 수정하는 모든 코드를 망가트리게 된다.
- 모든 곳이 보호되었는지 파악하기 위해 중복적인 노력이 필요하게 된다.
- 이미 찾기 어려운 문제의 근원을 더 찾기 어렵게 만들게 된다.

**추천**: *데이터 캡슐화를 가슴 깊이 새기며, 공유될 만한 자원에 접근하는 부분(코드)을 극도로 줄여라*

<a name="4-3"></a>
#### Corollary: 데이터의 복사본을 사용하라 ####
공유 자원 문제를 해결하는 좋은 방법중 하나는 애초에 공유 자원을 사용하지 않는 것이다. 읽기 전용으로 사용될 경우 자원의 복사본을 사용하게 하는 방법이 있다. 경우에 따라서는 복사본을 여러 스레드에 전달, 작업을 수행하고 결과를 단일 스레드에서 수집해 사용하는 것도 가능하다.

객체의 복사에 드는 비용을 걱정할 수도 있다. 혹은, 이 문제가 "진짜 문제가 되는지" 조사해 보는 방법도 있다. 하지만 객체의 복사본을 사용함으로써 동기화를 피할 수 있다면, 객체 생성 및 GC에 드는 비용은 공유 자원 동기화에 필요한 비용 보다 일반적으로 적은 비용으로 문제를 해결하게 해 준다.(객체 복사 cost < 공유 자원 동기화 cost)

<a name="4-4"></a>
#### Corollary: 스레드는 가능한 한 독립적이어야 한다 ####
스레드 코드를 공유 자원을 사용하지 않는 독립된 세계로 만든다면 동기화 문제는 없어지게 된다.

HttpServlet을 생각해 보라. HttpServlet을 상속받는 클래스는 doGet, doPost와 같은 메서드에서 필요한 파라미터를 받아 처리한다.
이는 각 Servlet이 각자의 세계에 있는 것처럼 작동하게 도와주며, 지역 변수를 사용하는 한 동기화 문제는 발생하지 않게 된다.
물론 대부분의 Servlet들은 데이터베이스 연결과 같은 공유 자원이 필요하긴 하다.

**추천**: *데이터를 독립적인 스레드-더 나아가 각각의 프로세서-에서 사용될 수 있게 구분하라*

<a name="5"></a>
## 라이브러리를 이해하라 ##
자바 5버전 이상에서 스레드 관련 코드 작성시 아래의 사항들을 숙지하자.

- 자바에서 제공하는 thread-safe 컬랙션을 사용하라.
- 연관이 없는 태스크들을 수행시 executor 프레임워크를 사용하라.
- 가능하면 nonblocking 방법을 사용하라.
- 몇몇 라이브러리 클래스들은 thread-safe하지 않다.

<a name="5-1"></a>
#### Thread-Safe한 컬랙션들 ####
java.util.concurrent 패키지는 멀티 스레드 환경에서 사용할 수 있는 컬랙션들을 제공한다. ConcurrentHashMap의 경우에는 일반 HashMap보다 대부분의 상황에서 더 좋은 퍼포먼스를 제공한다. 만약 배포 환경이 자바 5버전 이상이라면 이 패키지를 활용하자.

아래와 같은 고급 concurrency 디자인 구현을 위한 컴포넌트들도 숙지하자.

| Name            | Description                                                         |
| :-------------- | :------------------------------------------------------------------ |
| ReentrantLock   | A lock that can be acquired in one method and released in another.  |
| Semaphore       | An implementation of the classic semaphore, a lock with a count.    |
| CountDownLatch  | A lock that waits for a number of events before releasing all threads waiting on it. This allows all threads to have a fair chance of starting at about the same time. |

<a name="6"></a>
## 실행 모델에 대한 이해 ##

<a name="6-1"></a>
#### 생산자-소비자 ####

<a name="6-2"></a>
#### 저자-독자 ####

<a name="6-3"></a>
#### 철학자들의 저녁식사 ####

<a name="7"></a>
## 동기화된 메서드 간의 의존성을 주의하라 ##

<a name="8"></a>
## 제대로 "Shut-Down"하는 코드는 작성하기 어렵다 ##

<a name="9"></a>
## 스레드를 포함하는 코드 테스트하기 ##

<a name="9-1"></a>
#### 일시적이라고 판단되는 문제를 스레드 관련 이슈의 후보로 생각하라 ####

<a name="9-2"></a>
#### 스레드와 관련없는 코드를 작동하게 하라 ####

<a name="9-3"></a>
#### 스레드와 연관될 코드를 pluggable하게 만들어라 ####

<a name="9-4"></a>
#### 스레드와 연관될 코드를 tunable하게 만들어라 ####

<a name="9-5"></a>
#### 프로세서의 수 보다 많은 수의 스레드를 실행해 보라 ####

<a name="9-6"></a>
#### 다른 플렛폼에서 실행해 보라 ####

<a name="9-7"></a>
#### 문제가 발생하도록 조작해 보라 ####

<a name="9-7-1"></a>
###### 직접 조작 ######

<a name="9-7-2"></a>
###### 자동화된 조작 ######

<a name="10"></a>
## 결론 ##

======================================================

#### 참조 ####
<a name="fn1">
##### 1. Concurrency #####
</a>
https://en.wikipedia.org/wiki/Concurrency_(computer_science)

<a name="fn2">
##### 2. One-off #####
</a>
사전적 의미는 "한 번만 일어나는"이며, 여기에서는 "고칠 수 없는"이라는 의미도 포함하고 있다.

<a name="fn3">
##### 3. 더 자세한 내용은 [부록 A: Concurrency II]를 참고하길 바란다. #####
</a>

<a name="fn4">
##### 4. SRP(Single Responsibility Principle) #####
</a>
https://en.wikipedia.org/wiki/Single_responsibility_principle
