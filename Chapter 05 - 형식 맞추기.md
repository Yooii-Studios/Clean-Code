## 목차  
- 형식을 맞추는 목적  
- 적절한 행 길이를 유지하라  
	- 신문 기사처럼 작성하라
	- 개념은 빈 행으로 분리하라
	- 세로 밀집도
	- 수직 거리
	- 세로 순서
- 가로 형식 맞추기
	- 가로 공백과 밀집도
	- 가로 정렬
	- 들여쓰기
	- 가짜 범위
- 팀 규칙
- 밥 아저씨의 형식 규칙

## Intro
질서정연하고 깔끔하며, 일관적인 코드를 본다면 사람들에게 전문가가 짰다는 인상을 심어줄 수 있다.  
반대로, 코드가 어수선해 보인다면 프로젝트 전반적으로 무성의한 태도로 작성했다고 생각할 것이다.

프로그래머라면 형식을 깔끔하게 맞춰 코드를 짜야한다.  
코드 형식을 맞추기 위한 간단한 규칙을 정하고, 그 규칙을 착실히 따라야 하며,  
팀으로 일한다면 팀이 합의해 규칙을 정하고 모두가 그 규칙을 따라야 한다.  
필요하다면 규칙을 자동으로 적용하는 도구를 활용한다 (**e.g. AndroidStudios의 Code Formatter**)  


## 형식을 맞추는 목적
코드 형식은 중요하다! 너무 중요해서 무시하기 어렵다.  
코드 형식은 의사소통의 일환이며, **의사소통은 전문 개발자의 일차적인 의무다.**  
오늘 구현한 기능이 다음 버전에서 바뀔 확률은 높고, 시간이 지나면 원래 코드의 흔적을 찾아볼 수 없는 경우도 많지만,  
오늘 구현한 코드의 스타일과 가독성 수준은 유지보수의 용이성과 확정성에 지속적인 영향을 미친다.  
  
**코드는 사라져도 스타일과 규율은 사라지지 않는다!**
  

## 적절한 행 길이를 유지하라 (코드의 세로 길이)
소스코드는 얼마나 길어야 적당할까?  
500줄을 넘지 않고 대부분 200줄 정도인 파일로도 커다란 시스템을 구축할 수 있다.  
(실제로 전문 자바 프로젝트들(JUnit, FitNesse, Time and Money 등)이 이렇게 구현되어있다)  
<br/>
코드 길이를 200줄 정도로 제한하는 것은 반드시 지킬 엄격한 규칙은 아니지만,  
일반적으로 큰 파일보다는 작은 파일이 이해하기 쉽다.


## 신문 기사처럼 작성하라
좋은 신문 기사는 최상단에 표제(기사를 몇마디로 요약하는 문구),  
첫 문단에는 전체 기사 내용을 요약하며, 기사를 읽으며 내려갈 수록 세세한 사실이 조금씩 드러나며 세부사항이 나오게 된다.  
소스파일 이름(표제)는 간단하면서도 설명이 가능하게 지어,  
이름만 보고도 올바른 모듈을 살펴보고 있는지를 판단 할 수 있도록 한다.  
소스파일의 첫 부분(요약 내용)은 고차원 개념과 알고리즘을 설명한다.  
아래로 내려갈수록 의도를 세세하게 묘사하며, 마지막에는 가장 저차원 함수(아마 Getter/Setter?)와 세부 내역이 나온다.  
신문이 사실, 날짜, 이름 등을 무작위로 뒤섞은 긴 기사 하나만 싣는다면 아무도 신문을 읽지 않을 것이다.  

## 개념은 빈 행으로 분리하라
코드의 각 줄은 수식이나 절을 나타내고, 여러 줄의 묶음은 완결된 생각 하나를 표현한다.  
생각 사이에는 빈 행을 넣어 분리해야한다. 그렇지 않다면 단지 줄바꿈만 다를 뿐인데도 코드 가독성이 현저히 떨어진다.

```java
//빈 행을 넣지 않을 경우
package fitnesse.wikitext.widgets;
import java.util.regex.*;
public class BoldWidget extends ParentWidget {
	public static final String REGEXP = "'''.+?'''";
	private static final Pattern pattern = Pattern.compile("'''(.+?)'''",
		Pattern.MULTILINE + Pattern.DOTALL);
	public BoldWidget(ParentWidget parent, String text) throws Exception {
		super(parent);
		Matcher match = pattern.matcher(text); match.find(); 
		addChildWidgets(match.group(1));}
	public String render() throws Exception { 
		StringBuffer html = new StringBuffer("<b>"); 		
		html.append(childHtml()).append("</b>"); 
		return html.toString();
	} 
}
```

```java
//빈 행을 넣을 경우
package fitnesse.wikitext.widgets;

import java.util.regex.*;

public class BoldWidget extends ParentWidget {
	public static final String REGEXP = "'''.+?'''";
	private static final Pattern pattern = Pattern.compile("'''(.+?)'''", 
		Pattern.MULTILINE + Pattern.DOTALL
	);
	
	public BoldWidget(ParentWidget parent, String text) throws Exception { 
		super(parent);
		Matcher match = pattern.matcher(text);
		match.find();
		addChildWidgets(match.group(1)); 
	}
	
	public String render() throws Exception { 
		StringBuffer html = new StringBuffer("<b>"); 
		html.append(childHtml()).append("</b>"); 
		return html.toString();
	} 
}
```

## 세로 밀집도
줄바꿈이 개념을 분리한다면, 반대로 세로 밀집도는 연관성을 의미한다.  
즉, 서로 밀집한 코드 행은 세로로 가까이 놓여야 한다.

```java
//의미없는 주석으로 변수를 떨어뜨려 놓아서 한눈에 파악이 잘 안된다.

public class ReporterConfig {
	/**
	* The class name of the reporter listener 
	*/
	private String m_className;
	
	/**
	* The properties of the reporter listener 
	*/
	private List<Property> m_properties = new ArrayList<Property>();
	public void addProperty(Property property) { 
		m_properties.add(property);
	}
```

```java
//의미 없는 주석을 제거함으로써 코드가 한눈에 들어온다.
//변수 2개에 메소드가 1개인 클래스라는 사실이 드러난다.

public class ReporterConfig {
	private String m_className;
	private List<Property> m_properties = new ArrayList<Property>();
	
	public void addProperty(Property property) { 
		m_properties.add(property);
	}
```

## 수직 거리 
서로 밀접한 개념은 세로로 가까이 둬야 한다.  
두 개념이 서로 다른 파일에 속한다면 규칙이 통하지 않지만,  
타당한 근거가 없다면 서로 밀접한 개념은 한 파일에 속해야 마땅하다(protected 변수를 피해야 하는 이유)  
같은 파일에 속할 정도로 밀접한 두 개념은 세로 거리로 연관성_- 한 개념을 이해하는 데 다른 개념이 중요한 정도 -_을 표현한다.  

#### 변수선언
변수는 사용하는 위치에서 최대한 가까이 선언한다.  
우리가 만든 함수는 매우 짧으므로 (Chapter3 - 함수를 공부했다면 말이지) 

```java
//InputStream이 함수 맨 처음에 선언 되어있다.

private static void readPreferences() {
	InputStream is= null;
	try {
		is= new FileInputStream(getPreferencesFile()); 
		setPreferences(new Properties(getPreferences())); 
		getPreferences().load(is);
	} catch (IOException e) { 
		try {
			if (is != null) 
				is.close();
		} catch (IOException e1) {
		} 
	}
}
```

```java
//모두들 알다시피 루프 제어 변수는 Test each처럼 루프 문 내부에 선언

public int countTestCases() { 
	int count= 0;
	for (Test each : tests)
		count += each.countTestCases(); 
	return count;
}
```

```java
//드물지만, 긴 함수에서는 블록 상단 또는 루프 직전에 변수를 선언 할 수도 있다.
...
for (XmlTest test : m_suite.getTests()) {
	TestRunner tr = m_runnerFactory.newTestRunner(this, test);
	tr.addListener(m_textReporter); 
	m_testRunners.add(tr);

	invoker = tr.getInvoker();
	
	for (ITestNGMethod m : tr.getBeforeSuiteMethods()) { 
		beforeSuiteMethods.put(m.getMethod(), m);
	}

	for (ITestNGMethod m : tr.getAfterSuiteMethods()) { 
		afterSuiteMethods.put(m.getMethod(), m);
	} 
}
...
```

#### 인스턴스 변수
인스턴스 변수는 클래스 맨 처음에 선언한다(자바의 경우).  
변수 간 세로로 거리를 두지 않는다 - 잘 설계한 클래스는 대다수 클래스 메서드가 인스턴스 변수를 사용하기 때문.  
C++의 경우에는 마지막에 선언하는 것이 일반적이다. 어느 곳이든 잘 알려진 위치에 인스턴스 변수를 모으는 것이 중요하다.

```java
//도중에 선언된 변수는 꽁꽁 숨겨놓은 보물 찾기와 같다. 십중 팔구 코드를 읽다가 우연히 발견한다. 발견해보시길.
//요즘은 IDE가 잘 되어있어서 찾기야 어렵지 않겠지만, 더러운건 마찬가지

public class TestSuite implements Test {
	static public Test createTest(Class<? extends TestCase> theClass,
									String name) {
		... 
	}

	public static Constructor<? extends TestCase> 
	getTestConstructor(Class<? extends TestCase> theClass) 
	throws NoSuchMethodException {
		... 
	}

	public static Test warning(final String message) { 
		...
	}
	
	private static String exceptionToString(Throwable t) { 
		...
	}
	
	private String fName;

	private Vector<Test> fTests= new Vector<Test>(10);

	public TestSuite() { }
	
	public TestSuite(final Class<? extends TestCase> theClass) { 
		...
	}

	public TestSuite(Class<? extends TestCase> theClass, String name) { 
		...
	}
	
	... ... ... ... ...
}
```

#### 종속 함수
한 함수가 다른 함수를 호출한다면(종속 함수) 두 함수는 세로로 가까이 배치한다.
가능하면 호출되는 함수를 호출하는 함수보다 뒤에 배치한다. (프로그램이 자연스럽게 읽힐 수 있도록)
이러한 규칙을 일관되게 적용한다면 독자는 방금 함수에서 호출한 함수가 잠시 후에 정의될 것이라고 자연스레 예측한다.

```java
/*첫째 함수에서 가장 먼저 호출하는 함수가 바로 아래 정의된다.
다음으로 호출하는 함수는 그 아래에 정의된다. 그러므로 호출되는 함수를 찾기가 쉬워지며
전체 가독성도 높아진다.*/
	
/*makeResponse 함수에서 호출하는 getPageNameOrDefault함수 안에서 "FrontPage" 상수를 사용하지 않고,
상수를 알아야 의미 전달이 쉬워지는 함수 위치에서 실제 사용하는 함수로 상수를 넘겨주는 방법이
가독성 관점에서 훨씬 더 좋다*/

public class WikiPageResponder implements SecureResponder { 
	protected WikiPage page;
	protected PageData pageData;
	protected String pageTitle;
	protected Request request; 
	protected PageCrawler crawler;
	
	public Response makeResponse(FitNesseContext context, Request request) throws Exception {
		String pageName = getPageNameOrDefault(request, "FrontPage");
		loadPage(pageName, context); 
		if (page == null)
			return notFoundResponse(context, request); 
		else
			return makePageResponse(context); 
		}

	private String getPageNameOrDefault(Request request, String defaultPageName) {
		String pageName = request.getResource(); 
		if (StringUtil.isBlank(pageName))
			pageName = defaultPageName;

		return pageName; 
	}
	
	protected void loadPage(String resource, FitNesseContext context)
		throws Exception {
		WikiPagePath path = PathParser.parse(resource);
		crawler = context.root.getPageCrawler();
		crawler.setDeadEndStrategy(new VirtualEnabledPageCrawler()); 
		page = crawler.getPage(context.root, path);
		if (page != null)
			pageData = page.getData();
	}
	
	private Response notFoundResponse(FitNesseContext context, Request request)
		throws Exception {
		return new NotFoundResponder().makeResponse(context, request);
	}
	
	private SimpleResponse makePageResponse(FitNesseContext context)
		throws Exception {
		pageTitle = PathParser.render(crawler.getFullPath(page)); 
		String html = makeHtml(context);
		SimpleResponse response = new SimpleResponse(); 
		response.setMaxAge(0); 
		response.setContent(html);
		return response;
	} 
...
```


## 서술적인 이름을 사용하라!  
> “코드를 읽으면서 짐작했던 기능을 각 루틴이 그대로 수행한다면 깨끗한 코드라 불러도 되겠다” - 워드

작은 함수는 그 기능이 명확하므로 이름을 붙이기가 더 쉬우며, 일관성 있는 서술형 이름을 사용한다면 코드를 순차적으로 이해하기도 쉬워진다.

## 함수 인수  
함수에서 이상적인 인수 개수는 0개(무항).
인수는 코드 이해에 방해가 되는 요소이므로 최선은 0개이고, 차선은 1개뿐인 경우이다.
출력인수(함수의 반환 값이 아닌 입력 인수로 결과를 받는 경우)는 이해하기 어려우므로 왠만하면 쓰지 않는 것이 좋겠다.

#### 많이 쓰는 단항 형식  
  - 인수에 질문을 던지는 경우  
	`boolean fileExists(“MyFile”);`  
  - 인수를 뭔가로 변환해 결과를 변환하는 경우  
	`InputStream fileOpen(“MyFile”);`  
  - 이벤트 함수일 경우 (이 경우에는 이벤트라는 사실이 코드에 명확하게 드러나야 한다.)

위의 3가지가 아니라면 단항 함수는 가급적 피하는 것이 좋다.

#### 플래그 인수  
플래그 인수는 추하다. 쓰지마라. bool 값을 넘기는 것 자체가 그 함수는 한꺼번에 여러가지 일을 처리한다고 공표하는 것과 마찬가지다.

#### 이항 함수  
단항 함수보다 이해하기가 어렵다.
Point 클래스의 경우에는 이항 함수가 적절하다.
2개의 인수간의 자연적인 순서가 있어야함 
`Point p = new Point(x,y);`
무조건 나쁜 것은 아니지만, 인수가 2개이니 만큼 이해가 어렵고 위험이 따르므로 가능하면 단항으로 바꾸도록

#### 삼항 함수  
이항 함수보다 이해하기가 훨씬 어려우므로, 위험도 2배 이상 늘어난다.
삼항 함수를 만들 때는 신중히 고려하라.

#### 인수 객체  
인수가 많이 필요할 경우, 일부 인수를 독자적인 클래스 변수로 선언할 가능성을 살펴보자
x,y를 인자로 넘기는 것보다 Point를 넘기는 것이 더 낫다.

#### 인수 목록  
때로는 String.format같은 함수들처럼 인수 개수가 가변적인 함수도 필요하다. 
String.format의 인수는 List형 인수이기 때문에 이항함수라고 할 수 있다.

#### 동사와 키워드
단항 함수는 함수와 인수가 동사/명사 쌍을 이뤄야한다.  
`writeField(name);`  
함수이름에 키워드(인수 이름)을 추가하면 인수 순서를 기억할 필요가 없어진다.  
`assertExpectedEqualsActual(expected, actual);`  

## 부수 효과를 일으키지 마라!  
부수효과는 거짓말이다. 함수에서 한가지를 하겠다고 약속하고는 남몰래 다른 짓을 하는 것이므로, 한 함수에서는 딱 한가지만 수행할 것!

아래 코드에서 `Session.initialize();`는 함수명과는 맞지 않는 부수효과이다.
```java
public class UserValidator {
	private Cryptographer cryptographer;
	public boolean checkPassword(String userName, String password) { 
		User user = UserGateway.findByName(userName);
		if (user != User.NULL) {
			String codedPhrase = user.getPhraseEncodedByPassword(); 
			String phrase = cryptographer.decrypt(codedPhrase, password); 
			if ("Valid Password".equals(phrase)) {
				Session.initialize();
				return true; 
			}
		}
		return false; 
	}
}
```

#### 출력인수  
   일반적으로 출력 인수는 피해야 한다.   
   함수에서 상태를 변경해야 한다면 함수가 속한 객체 상태를 변경하는 방식을 택하라.

## 명령과 조회를 분리하라
함수는 뭔가 객체 상태를 변경하거나, 객체 정보를 반환하거나 둘중 하나다. 둘다 수행해서는 안된다.  
`public boolean set(String attribute, String value);`같은 경우에는 속성 값 설정 성공 시 true를 반환하므로 괴상한 코드가 작성된다.  
`if(set(“username”, “unclebob”))...` 그러므로 명령과 조회를 분리해 혼란을 주지 않도록 한다.  

## 오류코드보다 예외를 사용하라!

try/catch를 사용하면 오류 처리 코드가 원래 코드에서 분리되므로 코드가 깔끔해 진다.

#### Try/Catch 블록 뽑아내기  
```java
if (deletePage(page) == E_OK) {
	if (registry.deleteReference(page.name) == E_OK) {
		if (configKeys.deleteKey(page.name.makeKey()) == E_OK) {
			logger.log("page deleted");
		} else {
			logger.log("configKey not deleted");
		}
	} else {
		logger.log("deleteReference from registry failed"); 
	} 
} else {
	logger.log("delete failed"); return E_ERROR;
}
```
정상 작동과 오류 처리 동작을 뒤섞는 추한 구조이므로 if/else와 마찬가지로 블록을 별도 함수로 뽑아내는 편이 좋다.
````java
public void delete(Page page) {
	try {
    		deletePageAndAllReferences(page);
  	} catch (Exception e) {
    		logError(e);
  	}
}

private void deletePageAndAllReferences(Page page) throws Exception { 
	deletePage(page);
	registry.deleteReference(page.name); 
	configKeys.deleteKey(page.name.makeKey());
}

private void logError(Exception e) { 
	logger.log(e.getMessage());
}
```
오류 처리도 한가지 작업이다.

Error.java 의존성 자석
```java
public enum Error { 
	OK,
	INVALID,
	NO_SUCH,
	LOCKED,
	OUT_OF_RESOURCES, 	
	WAITING_FOR_EVENT;
}
```

오류를 처리하는 곳곳에서 오류코드를 사용한다면 enum class를 쓰게 되는데 이런 클래스는 의존성 자석이므로, 새 오류코드를 추가하거나 변경할 때 코스트가 많이 필요하다.
그러므로 예외를 사용하는 것이 더 안전하다.

## 반복하지 마라!  
중복은 모든 소프트웨어에서 모든 악의 근원이므로 늘 중복을 없애도록 노력해야한다.

## 구조적 프로그래밍  
구조적 프로그래밍의 원칙을 따르자면 모든 함수와 함수 내 모든 블록에 입구와 출구가 하나여야 된다. 즉, 함수는 return문이 하나여야 되며, 루프 안에서 break나 continue를 사용해선 안된다고 한다. 함수가 클 경우에만 상당 이익을 제공하므로, 함수를 작게 만든다면 오히려 여러차례 사용하는 것이 함수의 의도를 표현하기 쉬워진다.

## 함수를 어떻게 짜죠?  
처음에는 길고 복잡하고, 들여쓰기 단계나 중복된 루프도 많다. 인수목록도 길지만, 이 코드들을 빠짐없이 테스트하는 단위 테스트 케이스도 만들고, 
코드를 다듬고, 함수를 만들고, 이름을 바꾸고, 중복을 제거한다. 처음부터 탁 짜지지는 않는다.
