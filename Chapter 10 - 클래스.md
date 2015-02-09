## 목차  
- 클래스 체계
  - 캡슐화
- 클래스는 작아야 한다!
	- 단일 책임 원칙
	- 응집도 Cohesion
	- 응집도를 유지하면 작은 클래스 여럿이 나온다
- 변경하기 쉬운 클래스
  - 변경으로부터 거리

## Intro
이전 챕터에서 코드, 코드 블록, 함수 구현 방법과 함수간의 관련 맺는 방식을 공부 했지만  
하지만 좀 더 차원 높은 단계까지 신경 쓰지 않으면 코드를 얻기는 어렵다.
이 장에서는 클래스를 다룬다.

## 클래스 체계
JAVA Convention에 따르면 가장 먼저 변수 목록이 나온다.
**static public > static private > private 인스턴스 > public...은 필요한 경우가 거의 없다.**  
변수목록 다음에는 공개 함수가 나온다. 비공개 함수는 자신을 호출 하는 공개 함수 직후에 나온다.  
즉, 추상화 단계가 순차적으로 내려간다.

#### 캡슐화
변수와 유틸리티 함수는 가능한 공개하지 않는 편이 낫지만 반드시 숨겨야 하는 것은 아니다.  
우리에게 테스트는 중요하므로 테스트를 위해 protected로 선언해서 접근을 허용하기도 한다.  
**하지만 비공개 상태를 유지할 온갖 방법을 강구하고, 캡슐화를 풀어주는 결정은 언제나 최후의 수단이다.**

## 클래스는 작아야 한다!
클래스는 첫째! 작아야한다. 둘째! 작아야한다. 더 작아야 한다. 단 함수와는 다르게(함수는 물리적인 행 수로 측정)  
**클래스는 맡은 책임을 측정한다.**

#### 개념은 빈 행으로 분리하라
코드의 각 줄은 수식이나 절을 나타내고, 여러 줄의 묶음은 완결된 생각 하나를 표현한다.  
생각 사이에는 빈 행을 넣어 분리해야한다. 그렇지 않다면 단지 줄바꿈만 다를 뿐인데도 코드 가독성이 현저히 떨어진다.

```java
//어마어마하게 큰 슈퍼 만능 클래스
```

```java
//메소드를 5개로 줄인다고 하더라도 여전히 책임이 많다..
```

클래스 이름은 해당 클래스 책임을 기술해야된다. 작명은 클래스 크기를 줄이는 첫번째 관문임.  
간결한 이름이 떠오르지 않는다면 클래스 책임이 너무 많아서이다.  
(e.g. Chapter 2장에 언급한 것 처럼 Manager, Processor, Super 등)

또한 클래스 설명은 "if", "and", "or", "but"을 사용하지 않고 25 단어 내외로 가능해야된다.
한글의 경우 만약, 그리고, ~하며, 하지만 이 들어가면 안된다.

#### 단일 책임의 원칙
단일 책임의 원칙 (이하 SRP)은 클래스나 모듈을 변경할 이유가 단 하나뿐이어야 한다는 원칙이다.
책임, 즉 변경할 이유를 파악하려고 애쓰다 보면 코드를 추상화 하기도 쉬워진다.  

```java
//이 코드는 작아보이지만, 변경할 이유가 2가지이다.
```

```java
//위 코드에서 버전 정보를 다루는 메서드 3개를 따로 빼서
//Version이라는 독자적인 클래스를 만들어 다른 곳에서 재사용하기 쉬워졌다.
```

SRP는 객체지향설계에서 더욱 중요한 개념이고, 지키기 수월한 개념인데, 개발자가 가장 무시하는 규칙 중 하나이다.  
대부분의 프로그래머들이 **돌아가는 소프트웨어**에 초점을 맞춘다. 전적으로 올바른 태도이기는 하지만,  
돌아가는 소프트웨어가 작성되면 **깨끗하고 체계적인 소프트웨어**라는 다음 관심사로 전환을 해야한다.

작은 클래스가 많은 시스템이든, 큰 클래스가 몇 개뿐인 시스템이든 돌아가는 부품은 그 수가 비슷하다.
> "도구 상자를 어떻게 관리하고 싶은가?  
   작은 서랍을 많이 두고 기능과 이름이 명확한 컴포넌트를 나눠 넣고 싶은가?  
   아니면 큰 서랍 몇개를 두고 모두 떤져 넣고 싶은가?"  



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

#### 세로 밀집도
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

#### 수직 거리 
서로 밀접한 개념은 세로로 가까이 둬야 한다.  
두 개념이 서로 다른 파일에 속한다면 규칙이 통하지 않지만,  
타당한 근거가 없다면 서로 밀접한 개념은 한 파일에 속해야 마땅하다(protected 변수를 피해야 하는 이유)  
같은 파일에 속할 정도로 밀접한 두 개념은 세로 거리로 연관성_- 한 개념을 이해하는 데 다른 개념이 중요한 정도 -_을 표현한다.  

###### 변수선언
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

###### 인스턴스 변수
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

###### 종속 함수
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

###### 개념의 유사성
개념적인 친화도가 높을 수록 코드를 서로 가까이 배치한다.  
앞서 살펴보았듯이 한 함수가 다른 함수를 호출하는 종속성, 변수와 그 변수를 사용하는 함수가 그 예다.  
그 외에도 비슷한 동작을 수행하는 함수 무리 또한 개념의 친화도가 높다.

```java
//같은 assert 관련된 동작들을 수행하며, 명명법이 똑같고 기본 기능이 유사한 함수들로써 개념적 친화도가 높다.
//이런 경우에는 종속성은 오히려 부차적 요인이므로, 종속적인 관계가 없더라도 가까이 배치하면 좋다.

public class Assert {
	static public void assertTrue(String message, boolean condition) {
		if (!condition) 
			fail(message);
	}

	static public void assertTrue(boolean condition) { 
		assertTrue(null, condition);
	}

	static public void assertFalse(String message, boolean condition) { 
		assertTrue(message, !condition);
	}
	
	static public void assertFalse(boolean condition) { 
		assertFalse(null, condition);
	} 
...
```

#### 세로 순서
일반적으로 함수 호출 종속성은 아래방향으로 유지하므로, 호출되는 함수를 호출하는 함수보다 뒤에 배치한다.  
그러면 소스코드가 자연스럽게 고차원-->저차원으로 내려간다.  
가장 중요한 개념을 가장 먼저 표현하고, 세세한 사항은 마지막에 표현한다.  
그렇게 하면 첫 함수 몇개만 읽어도 개념을 파악하기 쉬워질 것이다.


## 가로 형식 맞추기
대다수의 프로그래머들은 명백히 짧은 행을 선호하므로 짧은 행이 바람직하다.  
Hollerith가 제안한 80자 제한은 다소 인위적이므로 조금 더 늘여도 좋다. 하지만 120자 이상을 넘어간다면 주의 부족이다.  
필자 개인적으로는 120자 정도로 길이를 제한한다.

#### 가로 공백과 밀집도
가로로는 공백을 사용해 밀접/느슨한 개념을 표현한다

```java
private void measureLine(String line) { 
	lineCount++;
	
	//흔히 볼 수 있는 코드인데, 할당 연산자 좌우로 공백을 주어 왼쪽,오른쪽 요소가 확실하게 구분된다.
	int lineSize = line.length();
	totalChars += lineSize; 
	
	//반면 함수이름과 괄호 사이에는 공백을 없앰으로써 함수와 인수의 밀접함을 보여준다
	//괄호 안의 인수끼리는 쉼표 뒤의 공백을 통해 인수가 별개라는 사실을 보여준다.
	lineWidthHistogram.addLine(lineSize, lineCount);
	recordWidestLine(lineSize);
}
```

추가로 연산자의 우선순위를 강조하기 위해서도 공백을 사용한다
`return b*b - 4*a*c;`  
하지만 Code Formatter등의 도구가 연산자 우선순위까지 고려하지 못하므로 공백을 임의로 넣어주더라도 사라지는 경우가 대부분.
<br/>
Q.그렇다면 괄호로 묶어주는 것이 더 바람직하지 않을까?

#### 가로 정렬
```java
public class FitNesseExpediter implements ResponseSender {
	private		Socket		socket;
	private 	InputStream 	input;
	private 	OutputStream 	output;
	private 	Reques		request; 		
	private 	Response 	response;	
	private 	FitNesseContex	context; 
	protected 	long		requestParsingTimeLimit;
	private 	long		requestProgress;
	private 	long		requestParsingDeadline;
	private 	boolean		hasError;
	
	... 
```
보기엔 깔끔해 보일지 모르나, 코드가 엉뚱한 부분을 강조해 변수 유형을 자연스레 무시하고 이름부터 읽게 된다.  
게다가 Code Formatter 대부분들은 이렇게 해놔봤자 무시하고 원래대로 돌려놓는다(내가 이 문제 때문에 씨름한 경험이 있음)  
그러므로 선언문과 할당문을 별도로 정렬할 필요가 없다.  
정렬이 필요할 정도로 목록이 길다면(Q. 여기서 말하는 목록은 인스턴스 변수 개수를 말하는 것인가?),  
목록의 길이가 문제이지 정렬이 부족해서가 아니다. 선언부가 길다는 것은 클래스를 쪼개야 한다는 것을 의미한다.

#### 들여쓰기  
들여쓰기를 잘 해놓으면 _- 물론 그러고 있겠지만! -_ 구조가 한 눈에 들어온다.

###### 들여쓰기 무시하기
간단한 if문, while문, 짧은 함수에서 들여쓰기를 무시하고픈 유혹이 생긴다.(정말?)   
하지만 들여쓰기로 제대로 범위를 표현한 코드가 가독성이 더 높다. 유혹을 뿌리치자

```java
//이렇게 한행에 다 넣을 수 있다고 다 때려 박는 것이 멋있는 코드가 아니란 것! 알아두삼

public class CommentWidget extends TextWidget {
	public static final String REGEXP = "^#[^\r\n]*(?:(?:\r\n)|\n|\r)?";
	
	public CommentWidget(ParentWidget parent, String text){super(parent, text);}
	public String render() throws Exception {return ""; } 
}
```

```java
//한줄이라도 정성스럽게 들여쓰기로 감싸주자. 가독성을 위해

public class CommentWidget extends TextWidget {
	public static final String REGEXP = "^#[^\r\n]*(?:(?:\r\n)|\n|\r)?";
	
	public CommentWidget(ParentWidget parent, String text){
		super(parent, text);
	}
	
	public String render() throws Exception {
		return ""; 
	} 
}
```

#### 가짜 범위
빈 while문이나 for문을 접할 때가 있다. 가능한한 피해야 되지만, 피하지 못 할 경우엔  
빈 블록을 올바로 들여쓰고 괄호로 감싸라. **그렇지 않으면 찾을 수 없는 버그가 발생할지도...**


## 팀 규칙
당연한 이야기들이 적혀있는 것 같지만.. 팀에 속해있다면,  
가장 우선시 되어야 하고 선호해야 할 규칙은 팀 규칙이다.  
처음 팀이 이루어졌다면 우성이형과 동현이처럼 코딩을 시작하기 전,  
코딩 스타일을 의논하여(괄호를 어디에 넣을지, 네이밍은 어떻게 할지 등) IDE Formatter로 지정하여
구현하는 것이 옳은 방식이다.  
**좋은 소프트웨어 시스템은 읽기 쉬운 문서로 이뤄지고, 읽기 쉬운 문서는 스타일이 일관적이고 매끄러워야 한다.**  

## 밥 아저씨의 형식 규칙
끝으로 이 책의 저자가 사용하는 규칙이 여실히 드러나는 코드를 첨부하며 턴을 종료한다.
```java
public class CodeAnalyzer implements JavaFileAnalysis { 
	private int lineCount;
	private int maxLineWidth;
	private int widestLineNumber;
	private LineWidthHistogram lineWidthHistogram; 
	private int totalChars;
	
	public CodeAnalyzer() {
		lineWidthHistogram = new LineWidthHistogram();
	}
	
	public static List<File> findJavaFiles(File parentDirectory) { 
		List<File> files = new ArrayList<File>(); 
		findJavaFiles(parentDirectory, files);
		return files;
	}
	
	private static void findJavaFiles(File parentDirectory, List<File> files) {
		for (File file : parentDirectory.listFiles()) {
			if (file.getName().endsWith(".java")) 
				files.add(file);
			else if (file.isDirectory()) 
				findJavaFiles(file, files);
		} 
	}
	
	public void analyzeFile(File javaFile) throws Exception { 
		BufferedReader br = new BufferedReader(new FileReader(javaFile)); 
		String line;
		while ((line = br.readLine()) != null)
			measureLine(line); 
	}
	
	private void measureLine(String line) { 
		lineCount++;
		int lineSize = line.length();
		totalChars += lineSize; 
		lineWidthHistogram.addLine(lineSize, lineCount);
		recordWidestLine(lineSize);
	}
	
	private void recordWidestLine(int lineSize) { 
		if (lineSize > maxLineWidth) {
			maxLineWidth = lineSize;
			widestLineNumber = lineCount; 
		}
	}

	public int getLineCount() { 
		return lineCount;
	}

	public int getMaxLineWidth() { 
		return maxLineWidth;
	}

	public int getWidestLineNumber() { 
		return widestLineNumber;
	}

	public LineWidthHistogram getLineWidthHistogram() {
		return lineWidthHistogram;
	}
	
	public double getMeanLineWidth() { 
		return (double)totalChars/lineCount;
	}

	public int getMedianLineWidth() {
		Integer[] sortedWidths = getSortedWidths(); 
		int cumulativeLineCount = 0;
		for (int width : sortedWidths) {
			cumulativeLineCount += lineCountForWidth(width); 
			if (cumulativeLineCount > lineCount/2)
				return width;
		}
		throw new Error("Cannot get here"); 
	}
	
	private int lineCountForWidth(int width) {
		return lineWidthHistogram.getLinesforWidth(width).size();
	}
	
	private Integer[] getSortedWidths() {
		Set<Integer> widths = lineWidthHistogram.getWidths(); 
		Integer[] sortedWidths = (widths.toArray(new Integer[0])); 
		Arrays.sort(sortedWidths);
		return sortedWidths;
	} 
}
```
