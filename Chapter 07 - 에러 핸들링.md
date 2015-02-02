## 목차 ##
- 리턴코드 대신 Exceptions를 사용하라
- Try-Catch-Finally문을 먼저 써라
- Unchecked Exceptions를 사용하라
- Exceptions로 문맥을 재공하라
- 사용에 맞게 Exceptions 클래스를 선언하라
- 정상적인 상황을 정의하라(Default값을 설정하라)
- Null을 리턴하지 마라
- Null을 넘기지 마라
- 결론

======================================================

> 오류 처리는 중요하다. 하지만 로직을 헷갈리게 만드는 오류처리는 나쁘다.
(원문 : Error handling is important, but if it obscures logic, it's wrong.)

## 리턴코드 대신 Exceptions를 사용하라 ##
- 예전 프로그래밍 언어들은 exceptions를 제공하지 않았다.
- 그 경우, 개발자들은 에러 flag를 set하거나 에러 코드를 리턴, 호출하는 측에서 예외처리를 해줘야 했다.
- 하지만 이러한 방식은 예외처리를 잊어버리기 쉽고 로직을 헷갈리게 하기 쉽다.
- 그러므로 exceptions를 사용하자. 겉보기에만 아름다운 코드가 되는게 아니라 "실제 로직"과 "예외처리" 부분이 나뉘어져 필요한 부분에 집중할 수 있게 된다.
```java
// Bad
public class DeviceController {
  ...
  public void sendShutDown() {
    DeviceHandle handle = getHandle(DEV1);
    // Check the state of the device
    if (handle != DeviceHandle.INVALID) {
      // Save the device status to the record field
      retrieveDeviceRecord(handle);
      // If not suspended, shut down
      if (record.getStatus() != DEVICE_SUSPENDED) {
        pauseDevice(handle);
        clearDeviceWorkQueue(handle);
        closeDevice(handle);
      } else {
        logger.log("Device suspended. Unable to shut down");
      }
    } else {
      logger.log("Invalid handle for: " + DEV1.toString());
    }
  }
  ...
}
```
```java
// Good
public class DeviceController {
  ...
  public void sendShutDown() {
    try {
      tryToShutDown();
    } catch (DeviceShutDownError e) {
      logger.log(e);
    }
  }
    
  private void tryToShutDown() throws DeviceShutDownError {
    DeviceHandle handle = getHandle(DEV1);
    DeviceRecord record = retrieveDeviceRecord(handle);
    pauseDevice(handle); clearDeviceWorkQueue(handle); closeDevice(handle);
  }
  
  private DeviceHandle getHandle(DeviceID id) {
    ...
    throw new DeviceShutDownError("Invalid handle for: " + id.toString());
    ...
  }
  ...
}
```

## Try-Catch-Finally문을 먼저 써라 ##
- try문은 transaction처럼 동작하는 실행코드로, catch문은 try문에 관계없이 프로그램을 일관적인 상태로 유지하도록 한다.
- 이렇게 함으로써 코드의 "Scope 정의"가 가능해진다.
- 예시 : 잘못된 input을 넣을 경우 StorageException을 제대로 던지는지 확인하는 테스트 코드를 작성해보자
```java
  // Step 1: StorageExceptio을 던지지 않으므로 이 테스트는 실패한다.
  
  @Test(expected = StorageException.class)
  public void retrieveSectionShouldThrowOnInvalidFileName() {
    sectionStore.retrieveSection("invalid - file");
  }
  
  public List<RecordedGrip> retrieveSection(String sectionName) {
    // dummy return until we have a real implementation
    return new ArrayList<RecordedGrip>();
  }
```
```java
  // Step 2: 이제 테스트는 통과한다.
  public List<RecordedGrip> retrieveSection(String sectionName) {
    try {
      FileInputStream stream = new FileInputStream(sectionName)
    } catch (Exception e) {
      throw new StorageException("retrieval error", e);
    }
  return new ArrayList<RecordedGrip>();
}
```
```java
  // Step 3: Exception의 범위를 FileNotFoundException으로 줄여 정확히 어떤 Exception이 발생한지 체크하자.
  public List<RecordedGrip> retrieveSection(String sectionName) {
    try {
      FileInputStream stream = new FileInputStream(sectionName);
      stream.close();
    } catch (FileNotFoundException e) {
      throw new StorageException("retrieval error”, e);
    }
    return new ArrayList<RecordedGrip>();
  }
```

## Unchecked Exceptions를 사용하라 ##
- Checked exception VS Unchecked Exception(참조 1)
- 예외처리에 드는 비용 대비 이득을 생각해봐야 한다.
- 예시
 - 1. 특정 메소드에서 checked exception을 throw하고
 - 2. 3단계(메소드 콜) 위의 메소드에서 그 exception을 catch한다면
 - 3. 모든 중간단계 메소드에 exception을 정의해야 한다.(자바의 경우 메소드 선언에 throws 구문을 붙이는 등)
 - Open/Closed Principle violation(참조 2)
- 상위 레벨 메소드에서 하위 레벨 메소드의 디테일에 대해 알아야 하기 때문에 캡슐화 또한 깨진다.
- 필요한 경우 checked exceptions를 사용해야 되지만 일반적인 경우 득보다 실이 많다.

## Exceptions로 문맥을 재공하라 ##
## 사용에 맞게 Exceptions 클래스를 선언하라 ##
## 정상적인 상황을 정의하라(Default값을 설정하라) ##
## Null을 리턴하지 마라 ##
## Null을 넘기지 마라 ##
## 결론 ##


#### 참조 ####
##### 1. Checked exception VS Unchecked Exception #####
Checked Exception과 Unchecked Exception의 가장 명확한 구분 기준은 ‘꼭 처리를 해야 하느냐’이다. Checked Exception이 발생할 가능성이 있는 메소드라면 반드시 로직을 try/catch로 감싸거나 throw로 던져서 처리해야 한다. 반면에 Unchecked Exception은 명시적인 예외처리를 하지 않아도 된다. 이 예외는 피할 수 있지만 개발자가 부주의해서 발생하는 경우가 대부분이고, 미리 예측하지 못했던 상황에서 발생하는 예외가 아니기 때문에 굳이 로직으로 처리를 할 필요가 없도록 만들어져 있다.
출처: http://www.nextree.co.kr/p3239/
##### 2. Open/Closed Principle #####
The Open Close Principle states that the design and writing of the code should be done in a way that new functionality should be added with minimum changes in the existing code. The design should be done in a way to allow the adding of new functionality as new classes, keeping as much as possible existing code unchanged.
출처: http://www.oodesign.com/open-close-principle.html

## 제목 ##
#### 부제목 ####
내용
```java
// comment
if (someCondition) {
  statement;
}
```
