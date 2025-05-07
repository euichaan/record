# IntelliJ 디버깅
인텔리제이의 디버깅 기능을 알아보자.  
IntelliJ IDEA, a JetBrains IDE 공식 유튜브 채널에는 IntelliJ IDEA의 디버깅 기능을 소개하는 코스가 있다.  
## Essentials
왜 Debugging을 할까?  
- Find and fix bugs!  
- Code analysis
  - What parts of the code are executed
  - Getting familiar with the new code  
- Change behavior
  - To reproduce complicated setup (복잡한 설정(상황)을 재현하기 위해)
  - To switch app state
  - To patch the app in production   
- Add more logging on the fly  
- Analyze memory issues  
- ...
  
간단한 Java 애플리케이션을 Debug 모드로 시작한다.  
```java
public class Endless {
    public static void main(String[] args) throws IOException {
        while (true) {
            int read = System.in.read();
            System.out.println("Input: " + read);
            if (filter(read)) {
                process(read);
            }
        }
    }

    private static boolean filter(final int read) {
        return read != '\n' && read != 'a';
    }

    private static void process(final int arg) {
        if (Math.max(arg, 90) % 2 == 0) {
            System.out.println("!");
        }
    }
}
```
![image](https://github.com/user-attachments/assets/04565fef-7705-4a2e-b197-f4eb0a60dc29)  
`Pause Program`: 바로 작동하는 순간에 애플리케이션을 멈추게 한다.  
디버거 모드의 `Threads`에서 애플리케이션에서 작동하는 모든 스레드를 볼 수 있다.  
`frames` 에서는 메서드 호출의 순서를 나타내는 call stack을 볼 수 있다.  
```text
디버깅 중 코드 실행을 멈추는 두 가지 방법  
1. 코드에 breakpoint 를 설정하고, 해당 위치에 도달할 때까지 기다린다.  
2. 직접 실행을 중단(pause)시킨다. 이 경우, 현재 실행 중인 문(statement)을 완료한 후, 다음에 실행될 문에서 멈춘다.  
```
breakpoint를 일시적으로 비활성화 할 수 있다. option 키를 누르고 breakppoint를 클릭하면 된다.  
![image](https://github.com/user-attachments/assets/96f0b4c5-8894-4d62-ae4d-8320b8a55b94)  
breakpoint가 실질적인 실행 코드 라인이 아니라서 디버거가 무시하고 있는 경우도 있다.  
`Mute BreakPoints`를 사용하면 breakpoint를 무시할 수 있다.  
  
모든 줄에 breakpoint을 설정하는 애플리케이션 디버깅은 좋지 않다. 따라서 애플리케이션을 한 줄씩 실행할 방법이 필요하다.  
`Step Over`: 다음 라인으로 이동한다.   
`Step Into`: 메서드 안으로 들어가서 그 내부에서 어떤 일이 일어나는지 보여준다.  
`Step Out`: 현재 메서드에서 빠져나와, 이 메서드를 호출한 메서드로 이동한다.  
```java
if (Math.max(arg, 90) % 2 == 0) 
```
이 줄에서 Step Into를 사용하면 max 메서드 내부로 들어가는 것을 예상하지만 max 메서드는 jdk에서 가져온 것이고  
기본적으로 이러한 메서드를 필터링하기 때문에 그렇지 않다. 실제로 이러한 메서드에 step into 해야하는 경우 force를 사용할 수 있다.  
  
변수의 값을 보는 가장 쉬운 방법은 인라인 디버거이다. 소스 코드의 줄 끝에 값에 대한 정보가 나타난다.  
![image](https://github.com/user-attachments/assets/de0764ad-c44c-412e-a45f-b4158bf14abf)  
변수가 사용되는 곳과 선언된 곳에 값에 대한 정보를 표시한다.  
Variables Tab을 보면 컨텍스트에서 사용 가능한 모든 변수의 모든 값을 볼 수 있다.  
Jump to Source 을 클릭하면 해당 변수의 선언된 위치로 이동할 수 있고 Jump to Type Source를 클릭하면 해당 변수의 클래스로 이동한다.  
  
⭐️Quick Evaluate 기능을 사용하면 디버깅 중에 코드를 빠르게 평가할 수 있다.  
![image](https://github.com/user-attachments/assets/b961b20d-3ee7-433f-a999-968c94f3be04)  
option 키를 누르고 평가할 코드를 클릭하면, 결과를 빠르게 알려준다.  
`if (Math.max(arg, 90) % 2 == 0) {`   
예를 들어, 위의 코드에서 `arg`를 클릭하면 `arg`의 값을 보여주고, Math.max(arg, 90) 를 클릭하면 max 메서드의 결과를 보여준다.  
`Math.max(arg,90) % 2 == 0`을 클릭하면 해당 표현식을 평가한 결과를 보여준다.  
  
Evaluate 를 사용해서 코드를 평가할 수 있다.  
![image](https://github.com/user-attachments/assets/14765e43-3636-469c-bd4a-87916c795a1f)  
원하는 어떤 종류의 표현식이든 가능하며, private method의 호출도 가능하다.  
Code fragment를 사용하면 여러 줄의 코드를 평가할 수 있다.  
Watch를 사용해도 코드를 평가할 수 있는데, Evaluate와 Watch의 차이는 다음과 같다.  
```text
Evaluate의 경우 코드를 계속 수동 실행해야하지만, Watch의 경우 삭제하지 않는 한, break line이 실행될때마다 자동 실행된다.  
값을 변경하는 코드를 watch에 등록했다면 의도치 않게 값이 변경되서 다른 로직을 수행하게 되는 경우도 있으니 주의하자.    
Watch는 여러 디버깅 코드의 결과를 동시에 확인이 가능하고, 반복적으로 디버깅 코드를 사용할 필요가 없다.  
```
  
디버깅 중에 set value 를 통해 변수의 값을 바꿀 수 있다.  
breakpoint를 우클릭 하면 조건으로 break를 걸 수 있다.  
  







## Advanced

## Professional







출처
- https://jojoldu.tistory.com/149
- https://www.youtube.com/watch?v=59RC8gVPlvk&list=PLPZy-hmwOdEUWF85MuwrKV8YVWLmZW4ZA&index=1