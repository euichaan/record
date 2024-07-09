# 스레드 종료와 Daemon 스레드
## 스레드를 멈춰야 하는 이유
스레드는 리소스를 사용한다.  
- 메모리와 커널 리소스  
- 스레드가 실행 중이라면 CPU cycle과 cache memory  
  
따라서 생성한 스레드가 이미 작업을 완료했는데 애플리케이션은 작동 중이라면 사용하지 않는 스레드가 사용하는 리소스를 정리해야 한다.  
스레드가 오작동하면, 중지해야 한다.  
스레드를 정지하는 마지막 이유는 애플리케이션을 종료하기 위해서이다.  
  
### Thread.interrupt()
```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(new BlockingTask());

        thread.start();

        thread.interrupt();
    }

    private static class BlockingTask implements Runnable {

        @Override
        public void run() {
            try {
                Thread.sleep(500000);
            } catch (InterruptedException e) {
                System.out.println("Exiting blocking thread");
            }
        }
    }
}
```
메인 스레드가 생성된 스레드를 인터럽트하게 된다.  
Thread.sleep() 이 호출된 상태에서 interrupt를 걸어 InterruptedException이 발생했다.  
  
```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(new LongComputationTask(new BigInteger("2"), new BigInteger("10000000000")));

        thread.start();
        thread.interrupt();
    }

    private static class LongComputationTask implements Runnable {
        private final BigInteger base;
        private final BigInteger power;

        public LongComputationTask(BigInteger base, BigInteger power) {
            this.base = base;
            this.power = power;
        }

        @Override
        public void run() {
            System.out.println(base + "^" + power + " = " + pow(base, power));
        }

        private BigInteger pow(BigInteger base, BigInteger power) {
            BigInteger result = BigInteger.ONE;

            for (BigInteger i = BigInteger.ZERO; i.compareTo(power) != 0; i = i.add(BigInteger.ONE)) {
                result = result.multiply(base);
            }

            return result;
        }
    }

    private static class BlockingTask implements Runnable {
        @Override
        public void run() {
            try {
                Thread.sleep(500000);
            } catch (InterruptedException e) {
                System.out.println("Exiting blocking thread");
            }
        }
    }
}
```
인터럽트는 보내졌지만 이를 처리할 로직이 없다. 이 문제를 해결하려면 코드 내에서 시간이 오래 걸리는 핫스팟을 찾아야 한다.  
```java
        private BigInteger pow(BigInteger base, BigInteger power) {
            BigInteger result = BigInteger.ONE;

            for (BigInteger i = BigInteger.ZERO; i.compareTo(power) != 0; i = i.add(BigInteger.ONE)) {
                if (Thread.currentThread().isInterrupted()) {
                    System.out.println("Prematurely interrupted computation");
                    return BigInteger.ZERO;
                }
                result = result.multiply(base);
            }

            return result;
        }
```
### Daemon 스레드
background 스레드로 메인 스레드가 종료되어도 애플리케이션 종료를 막지 않는다.  
JVM은 데몬 스레드의 종료를 기다리지 않는다. JVM이 종료되는 과정에서 실행중인 데몬 스레드가 있다면 죽이고 셧다운 작업을 진행한다.  