## 목차 ##
- 주석은 나쁜 코드를 보완하지 못한다
- 코드로 의도를 표현하라   
- 좋은 주석   
    - 법적인 주석   
    - 정보를 제공하는 주석   
    - 의미를 설명하는 주석   
    - 의미를 명료하게 밝히는 주석   
    - 결과를 경고하는 주석    
    - TODO 주석    
    - 중요성을 강조하는 주석    
    - 공개 API에서 Javadocs     
- 나쁜 주석
    - 주절거리는 주석
    - 같은 이야기를 중복하는 주석
    - 오해할 여지가 있는 주석
    - 의무적으로 다는 주석
    - 이력을 기록하는 주석
    - 있으나 마나 한 주석
    - 무서운 잡음
    - 함수나 변수로 표현할 수 있다면 주석을 달지 마라
    - 위치를 표시하는 주석
    - 닫는 괄호에 다는 주석
    - 공로를 돌리거나 저자를 표시하는 주석
    - 주석으로 처리한 코드
    - HTML주석
    - 전역 정보
    - 너무 많은 정보
    - 모호한 관계
    - 함수 헤더
    - 비공개 코드에서 Javadocs

======================================================

> 나쁜 코드에 주석을 달지 마라. 새로 짜라.     
 - 브라이언 W.커니핸, P.J.플라우거

주석은 필요악이다. 코드로 의도를 표현하지 못해, 실패를 만회하기 위해 쓰는 것이다. 주석은 언제나 실패를 의미한다. 주석 없이는 자신을 표현할 방법을 찾지 못해 할 수 없이 주석을 사용한다. 그래서 주석은 반겨 맞을 손님이 아니다. 

주석을 무시하는 이유가 무엇이냐고? 주석이 오래될수록 코드에서 멀어져서 거짓말을 하게 될 가능성이 커지기 때문이다. 코드는 유지보수를 해도, 주석을 계속 유지보수하기란 현실적으로 불가능하기 때문이다.

## 주석은 나쁜 코드를 보완하지 못한다 ##
코드에 주석을 추가하는 일반적인 이유는 코드 품질이 나빠서이다. 깔끔하고 주석이 거의 없는 코드가, 복잡하고 어수선하며 주석이 많이 달린 코드보다 훨씬 좋다. 주석으로 설명하려 애쓰는 대신에 그 난장판을 깨끗이 치우는 데 시간을 보내라!

## 코드를 의도로 표현하라! ##

```java
// 직원에게 복지 혜택을 받을 자격이 있는지 검사한다. 
if ((emplotee.flags & HOURLY_FLAG) && (employee.age > 65)
```

다음 코드는 어떤가?

```java
if (employee.isEligibleForFullBenefits())
```

주석도 필요없이 함수 이름만으로 충분히 깔끔하게 표현되었다. 

## 좋은 주석 ##

#### 법적인 주석: 각 소스 파일 첫머리에 들어가는 저작권 정보와 소유권 정보 등
  `// Copyright (C) 2003, 2004, 2005 by Object Montor, Inc. All right reserved.`  
  `// GNU General Public License`

#### 정보를 제공하는 주석 

```java
// 테스트 중인 Responder 인스턴스를 반환
protected abstract Responder responderInstance();
```

  물론 이 주석도 함수 이름에 정보를 담아 responderBeingTested로 바꾸면 없앨 수 있다.    
  더 나은 예:

```java
// kk:mm:ss EEE, MMM dd, yyyy 형식이다.
Pattern timeMatcher = Pattern.compile("\\d*:\\d*\\d* \\w*, \\w*, \\d*, \\d*");
```

#### 의도를 설명하는 주석

```java
// 스레드를 대량 생성하는 방법으로 어떻게든 경쟁 조건을 만들려 시도한다. 
for (int i = 0; i > 2500; i++) {
    WidgetBuilderThread widgetBuilderThread = 
        new WidgetBuilderThread(widgetBuilder, text, parent, failFlag);
    Thread thread = new Thread(widgetBuilderThread);
    thread.start();
}
```

#### 결과를 경고하는 주석    

```java
// 여유 시간이 충분하지 않다면 실행하지 마십시오.
public void _testWithReallyBigFile() {

}
```

#### TODO 주석    

```java
// TODO-MdM 현재 필요하지 않다.
// 체크아웃 모델을 도입하면 함수가 필요 없다.
protected VersionInfo makeVersion() throws Exception {
    return null;
}
```
TODO 주석은 프로그래머가 필요하다 여기지만 당장 구현하기 어려운 업무를 기술한다. 더 이상 필요 없는 기능을 삭제하라는 알림, 누군가에게 문제를 봐달라는 요청, 더 좋은 이름을 떠올려달라는 부탁, 앞으로 발생할 이벤트에 맞춰 코드를 고치라는 주의 등에 유용하다. 요즘은 IDE를 통해 남은 TODO를 쉽게 볼 수 있으므로 편리하게 이용할 수 있다. 

#### 중요성을 강조하는 주석    

```java
String listItemContent = match.group(3).trim();
// 여기서 trim은 정말 중요하다. trim 함수는 문자열에서 시작 공백을 제거한다.
// 문자열에 시작 공백이 있으면 다른 문자열로 인식되기 때문이다. 
new ListItemWidget(this, listItemContent, this.level + 1);
return buildList(text.substring(match.end()));
```

#### 공개 API에서 Javadocs    
설명이 잘 된 공개 API는 참으로 유용하고 만족스럽다. 공개 API를 구현한다면 반드시 훌륭한 Javadocs 작성을 추천한다. 하지만 여느 주석과 마찬가지로 Javadocs 역시 독자를 오도하거나, 잘못 위치하거나, 그릇된 정보를 전달할 가능성이 존재하는 것 역시 잊으면 안 된다. 

## 나쁜 주석 ##
대다수의 주석이 이 범주에 속한다. 일반적으로 대다수 주석은 허술한 코드를 지탱하거나, 엉성한 코드를 변명하거나, 미숙한 결정을 합리화하는 등 프로그래머가 주절거리는 독백에서 크게 벗어나지 못한다. 

#### 주절거리는 주석    
특별한 이유 없이 달리는 주석이다. 

```java
public void loadProperties() {
    try {
        String propertiesPath = propertiesLocation + "/" + PROPERTIES_FILE;
        FileInputStream propertiesStream = new FileInputStream(propertiesPath);
        loadedProperties.load(propertiesStream);
    } catch (IOException e) {
        // 속성 파일이 없다면 기본값을 모두 메모리로 읽어 들였다는 의미다. 
    }
}
```

catch 블록에 있는 주석은 저자에게야 의미가 있겠지만 다른 사람들에게는 전해지지 않는다. 저 주석의 의미를 알아내려면 다른 코드를 뒤져보는 수밖에 없다. 이해가 안되어 다른 모듈까지 뒤져야 하는 주석은 제대로 된 주석이 아니다.     

- 같은 이야기를 중복하는 주석    
  코드 내용을 그대로 중복하는 주석이 있다. 전혀 필요없는 코드

```java
// this.closed가 true일 때 반환되는 유틸리티 메서드다.
// 타임아웃에 도달하면 예외를 던진다. 
public synchronized void waitForClose(final long timeoutMillis) throws Exception {
    if (!closed) {
        wait(timeoutMillis);
        if (!closed) {
            throw new Exception("MockResponseSender could not be closed");
        }
    }
}
```

#### 오해할 여지가 있는 주석    
위 코드를 다시 보자. 중복이 많으면서도 오해할 여지가 살짝 있다. this.closed가 true로 변하는 순간에 메서드는 반환되지 않는다. this.closed가 true여야 메서드는 반환된다. 아니면 무조건 타임아웃을 기다렸다 this.closed가 그래도 true가 아니면 예외를 던진다. 주석에 담긴 '살짝 잘못된 정보'로 인해 어느 프로그래머가 경솔하게 함수를 호출해 자기 코드가 아주 느려진 이유를 못찾게 되는 것이다.

#### 의무적으로 다는 주석    
모든 함수에 Javadocs를 달거나 모든 변수에 주석을 달아야 한다는 규칙은 어리석기 그지없다. 이런 주석은 코드를 복잡하게 만들며, 거짓말을 퍼뜨리고, 혼동과 무질서를 초래한다. 아래와 같은 주석은 아무 가치도 없다. 

```java
/**
 *
 * @param title CD 제목
 * @param author CD 저자
 * @param tracks CD 트랙 숫자
 * @param durationInMinutes CD 길이(단위: 분)
 */
public void addCD(String title, String author, int tracks, int durationInMinutes) {
    CD cd = new CD();
    cd.title = title;
    cd.author = author;
    cd.tracks = tracks;
    cd.duration = durationInMinutes;
    cdList.add(cd);
}
```

#### 이력을 기록하는 주석    
지금은 소스 코드 관리 시스템이 있으니 전혀 필요없다. 

````java
* 변경 이력 (11-Oct-2001부터)
* ------------------------------------------------
* 11-Oct-2001 : 클래스를 다시 정리하고 새로운 패키징
* 05-Nov-2001: getDescription() 메소드 추가
* 이하 생략
````

#### 있으나 마나 한 주석

```java
/*
 * 기본 생성자
 */
protected AnnualDateRule() {

}
```

#### 함수나 변수로 표현할 수 있다면 주석을 달지 마라    

```java
// 전역 목록 <smodule>에 속하는 모듈이 우리가 속한 하위 시스템에 의존하는가?
if (module.getDependSubsystems().contains(subSysMod.getSubSystem()))
```

주석을 제거하고 다시 표현하면 다음과 같다.

```java
ArrayList moduleDependencies = smodule.getDependSubSystems();
String ourSubSystem = subSysMod.getSubSystem();
if (moduleDependees.contains(ourSubSystem))
```

#### 위치를 표시하는 주석    
때때로 프로그래머는 소스 파일에서 특정 위치를 표시하려 주석을 사용한다. 예를 들어, 최근에 살펴보던 프로그램에서 다음 행을 발견했다. 

````java
// Actions /////////////////////////////////////////////
````

이런 주석은 가독성만 낮추므로 제거해야 마땅하다. 특히 뒷부분에 슬래시로 이어지는 잡음은 제거하는 편이 좋다. 너무 자주 사용하지 않을때만 배너는 눈에 띄며 주위를 환기한다. 그러므로 반드시 필요할 때 아주 드몰게 사용하는 편이 좋다. 

#### 닫는 괄호에 다는 주석    
중첩이 심하고 장황한 함수라면 의미가 있을지도 모르지만 작고 캡슐화면 함수에는 잡음일 뿐이다. 그러므로 닫는 괄호에 주석을 달아야겠다는 생각이 든다면 대신에 함수를 줄이려 시도하자. 

#### 공로를 돌리거나 저자를 표시하는 주석    
소스 코드 관리 시스템은 누가 언제 무엇을 추가했는지 귀신처럼 기억하기 때문에 저자 이름으로 코드를 오염시킬 필요가 없음. 

````java
/* 릭이 추가함 */
````

#### 주석으로 처리한 코드    

```java
this.bytePos = writeBytes(pngIdBytes, 0);
//hdrPos = bytePos;
writeHeader();
writeResolution();
//dataPos = bytePos;
if (writeImageData()) {
    wirteEnd();
    this.pngBytes = resizeByteArray(this.pngBytes, this.maxPos);
} else {
    this.pngBytes = null;
}
return this.pngBytes;
```

1960년대 즈음에는 주석으로 처리한 코드가 유용했었지만 우리는 우수한 소스 코드 관리 시스템을 사용하기 때문에 우리를 대신에 코드를 기억해준다. 그냥 삭제하라. 잃어버릴 염려는 없다. 약속한다. 

#### 전역 정보    
주석을 달아야 한다면 근처에 있는 코드만 기술하라. 시스템의 전반적인 정보를 기술하지 마라. 해당 시스템의 코드가 변해도 아래 주석이 변하리라는 보장은 전혀 없다. 그리고 심하게 중복된 주석도 확인하자. 

```java
/**
 * 적합성 테스트가 동작하는 포트: 기본값은 <b>8082</b>.
 *
 * @param fitnessePort
 */
public void setFitnessePort(int fitnessePort) {
    this.fitnewssePort = fitnessePort;
}
```

#### 비공개 코드에서 Javadocs    
공개 API는 Javadocs가 유용하지만 공개하지 않을 코드라면 Javadocs는 쓸모가 없다. 코드만 보기싫고 산만해질 뿐이다. 
