# Atomic Operations
synchronization(동기화)은 멀티스레드 환경에서 두 개 이상의 스레드가 동시에 실행되면서 발생할 수 있는 문제를 해결하기 위한 기법이다.  
주로 공유 자원에 대한 접근을 제어하며 race condition을 방지한다.  
  
어떤 작업이 원자적이고 비원자적인지 어떻게 알 수 있을까?  
```java
public class SharedClass {
    public synchronized method1() {

    }

    public synchronized method2() {
        
    }

    public synchronized method3() {
        
    }
}
```
방어적인 접근부터 해보자. 공유 변수에 액세스할 수 있는 모든 메서드를 동기화한다.  
동시에 실행될 거라 짐작되는 스레드를 네 개 만드는 경우 코드의 대부분이 동기화된 상태이기 때문에 한 번에 스레드 한 개만 실행될 수 있다.  
사실 오히려 최악의 상황인 게 동시 실행이 전혀 안 되는 것뿐만 아니라 여러 스레드를 유지하기 위해 컨텍스트 스위칭과 메모리 오버헤드라는 손해를 감수하고 있는 상황이기 때문이다.  
  
**동기화를 최소화하기 위해 노력해야 한다.**  
  
어떤 연산이 원자적일까?  
**모든 레퍼런스 할당은 원자적 연산이다.**    
레퍼런스를 가져오거나 배열, 문자열 등 객체에 설정하는 작업을 하는 모든 getter와 setter가 그 작업을 원자적으로 수행하게 되어서 동기화시킬 필요가 없다.  
  
`long`과 `double`을 제외한 원시형에 대한 모든 할당도 원시적 작업에 해당한다.  
int, short, byte, float, char, boolean은 동기화할 필요가 없이 안전하게 읽고 쓸 수 있다.  
  
long과 double은 길이가 64비트라서 Java가 보장해주지 않기 때문인데, 64비트 컴퓨터인 경우라도 long이나 double에 쓰기 작업을 하면 실제로는 CPU가 두 개 연산을 통해 완료할 가능성이 높아진다.  
  
volatile 키워드로 long이나 double 변수를 선언하면 해당 변수에 읽고 쓰는 작업이 스레드 안전성을 지닌 원자적 연산이 된다.  
  
`java.util.concurrent.atomic` 패키지를 참고하자.  