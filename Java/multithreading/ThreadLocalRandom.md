# ThreadLocalRandom
```text
A random number generator isolated to the current thread. Like the global Random generator used by the Math class, a ThreadLocalRandom is initialized with an internally generated seed that may not otherwise be modified. When applicable, use of ThreadLocalRandom rather than shared Random objects in concurrent programs will typically encounter much less overhead and contention. Use of ThreadLocalRandom is particularly appropriate when multiple tasks (for example, each a ForkJoinTask) use random numbers in parallel in thread pools.
Usages of this class should typically be of the form: ThreadLocalRandom.current().nextX(...) (where X is Int, Long, etc). When all usages are of this form, it is never possible to accidently share a ThreadLocalRandom across multiple threads.

This class also provides additional commonly used bounded random generation methods.

Instances of ThreadLocalRandom are not cryptographically secure. Consider instead using SecureRandom in security-sensitive applications. Additionally, default-constructed instances do not use a cryptographically random seed unless the system property java.util.secureRandomSeed is set to true.
```
중요하게 봐야할 내용은 다음과 같다.  
A random number generator isolated to the current thread.  
When applicable, use of ThreadLocalRandom rather than shared Random objects in concurrent programs will typically encounter much less overhead and contention.  
Use of ThreadLocalRandom is particularly appropriate when multiple tasks (for example, each a ForkJoinTask) use random numbers in parallel in thread pools.  
  
`ThreadLocalRandom.current().next(...)`의 형태로 사용한다.  
  
`Random` 클래스는 하나의 인스턴스를 여러 스레드에서 공유하여 사용하면 성능에 문제가 발생할 수 있다. 여러 스레드가 동시에 접근하게 되면 동기화가 필요하게 되고, 이로 인해 경합 상태가 발생할 수 있다.  
`ThreadLocalRandom` 클래스는 ThreadLocal 변수를 사용하여 각 스레드마다 독립적인 난수 생성기를 제공한다. 이는 스레드 간의 충돌을 방지하고, 동기화 오버헤드를 없애준다.
