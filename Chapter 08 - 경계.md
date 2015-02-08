## 목차 ##
- 서드파티 코드 사용하기
- 경계 탐험하고 공부하기
- log4j 공부하기
- "공부를 위한 테스트"는 값어치를 한다
- 아직 존재하지 않는 코드 사용하기
- Clean한 경계(주: 이 책에서 나오는 clean의 뉘앙스를 살리기 위해 일부러 번역하지 않음)
- 결론

======================================================

> 우리는 가끔 서드파티 패키지나 오픈소스를 사용해야될 상황에 직면한다. 혹은 우리 회사 내부 팀이 만든 컴포넌트를 사용해야할 상황도 있다.
어느 상황이던, 우리는 이 코드들을 우리 내부 코드와 "깨끗하게" 통합시켜야 한다.

## 서드파티 코드 사용하기 ##
- 인터페이스를 "제공하는" 입장과 "사용하는" 입장 사이에는 필연적인 긴장감이 존재한다.
 - "제공하는" 입장에서는 좀 더 다양한 환경에서 좀 더 많은 사용자가 사용할 수 있도록 다양한 사용성을 지향한다.
 - "사용하는" 입장에서는 그들의 사용성에 맞는 specific한 인터페이스를 원한다.
 - 이것을 "경계에서의 긴장"이라 부른다.  
 
| Figure 8-1. The methods of Map              |
| ------------------------------------------- |
| clear() void – Map                          |
| containsKey(Object key) boolean – Map       |
| containsValue(Object value) boolean – Map   |
| clear() void – Map                          |
| containsKey(Object key) boolean – Map       |
| containsValue(Object value) boolean – Map   |
| entrySet() Set – Map                        |
| equals(Object o) boolean – Map              |
| get(Object key) Object – Map                |
| getClass() Class<? extends Object> – Object |
| hashCode() int – Map                        |
| isEmpty() boolean – Map                     |
| keySet() Set – Map                          |
| notify() void – Object                      |
| notifyAll() void – Object                   |
| put(Object key, Object value) Object – Map  |
| putAll(Map t) void – Map                    |
| remove(Object key) Object – Map             |
| size() int – Map                            |
| toString() String – Object                  |
| values() Collection – Map                   |
| wait() void – Object                        |
| wait(long timeout) void – Object            |
| wait(long timeout, int nanos) void – Object |

## 경계 탐험하고 공부하기 ##
## log4j 공부하기 ##
## "공부를 위한 테스트"는 값어치를 한다 ##
## 아직 존재하지 않는 코드 사용하기 ##
## Clean한 경계(주: 이 책에서 나오는 clean의 뉘앙스를 살리기 위해 일부러 번역하지 않음) ##
## 결론 ##


- 내용
```java
// 코드
```
======================================================

#### 참조 ####
##### 1. 참조 #####
내용  
출처: url
참조 1: url
