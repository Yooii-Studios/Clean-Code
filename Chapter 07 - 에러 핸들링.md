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
## Unchecked Exceptions를 사용하라 ##
## Exceptions로 문맥을 재공하라 ##
## 사용에 맞게 Exceptions 클래스를 선언하라 ##
## 정상적인 상황을 정의하라(Default값을 설정하라) ##
## Null을 리턴하지 마라 ##
## Null을 넘기지 마라 ##
## 결론 ##


## 제목 ##
#### 부제목 ####
내용
```java
// comment
if (someCondition) {
  statement;
}
```
