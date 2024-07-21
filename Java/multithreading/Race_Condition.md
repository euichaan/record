# Race Condition
공유 리소스(공유 자원)에 접근하는 여러 스레드가 있거나 그 중 최소한 한 스레드가 공유 리소스를 수정하는 경우로 **스레드의 스케줄링 순서나 시점에 따라 결과가 달라지는 상황**을 말한다.    
  
문제의 핵심은 공유 리소스에 **비원자적 연산**이 실행되는 것이다.  
  
```text
공유 자원을 생성하는 상황에서도 race condition이 발생할 수 있다. 예를 들어, 두 개 이상의 스레드가 동일한 자원을 동시에 생성하려고 할 때 문제가 발생할 수 있다. 이는 보통 Singleton 패턴을 구현할 때 자주 나타난다.
```
```java
// race condition이 발생할 수 있는 Singleton 패턴
public class Singleton {
    
    private static Singleton instance;
    
    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```