# Wildcard Capture and Raw type use
```java
public static void main(String[] args) {
    List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
    reverse(list);
    System.out.println(list);
}
```
다음과 같이 리스트의 원소를 거꾸로 만드는 함수를 정의한다고 가정하자.  
reverse 메서드는 제네릭으로 정의할 수 있다.  
```java
public static <T> void reverse(List<T> list) {
    List<T> tmp = new ArrayList<>(list);
    for (int i = 0; i < list.size(); i++) {
        tmp.set(i, list.get(list.size() - i - 1));
    }
}
``` 
그러나, 자바 설계자들의 말에 따르면 메서드 내부에서 타입 파라미터를 사용하지 않는 경우 와일드카드를 사용하는 것이 좋다고 한다. 따라서 다음과 같이 변경한다.  
```java
public static void reverse(List<?> list) {
    List<?> tmp = new ArrayList<>(list);
    for (int i = 0; i < list.size(); i++) {
        tmp.set(i, list.get(list.size() - i - 1));
    }
}
```
그러나 이 경우, `list.get(list.size() -i -1)` 에서 컴파일 에러가 발생한다. `capture of ?` 라는 메시지를 볼 수 있는데, set 을 할 때 ? 로 인해 타입이 불확실하기 때문이다.  
참고로, 와일드카드로 선언된 제네릭 변수에는 null만 넣을 수 있다.  
이 문제를 해결하는 방법은 두 가지 방법이 있다.  
  
## 1. Helper Methods
첫 번째 방법은 Helper Method를 사용하는 방법이다.
```java
public static void reverse(List<?> list) {
    reverseHelper(list);
}

private static <T> void reverseHelper(final List<T> list) {
    List<T> temp = new ArrayList<>(list);
    for (int i = 0; i < list.size(); i++) {
        list.set(i, temp.get(list.size() - i - 1));
    }
}
```
다음과 같이 공개 api는 와일드카드로 구현하고, 비공개 헬퍼 메서드를 제네릭 메서드로 만드는 방법이다.  
제네릭은 불공변이므로 List<T> 를 파라미터로 받는 메서드를 List<?>의 argument로 어떻게 호출이 가능한지 궁금할 것이다.  
  
컴파일 시 List<?> list를 제네릭 메서드 타입으로 capture 하기 때문에 가능하다.  
> Wildcard capture
와일드카드가 포함된 타입을 다룰 때, 컴파일러가 ?를 구체적인 타입으로 변환(캡처)해야 특정 메서드를 호출할 수 있는 경우가 있다.  
메서드 내에서 와일드카드 타입을 사용할 때, 이를 명시적인 타입으로 캡처해서 처리하는 메커니즘.  
  
필요에 따라서 ?(와일드카드) 타입을 추론해야 할 때가 있다.  
추론을 하지 못하면 컴파일 에러가 발생한다.  
## 2. Raw type use
두 번째 방법은 Raw type을 사용하는 방법이다. Raw type이란 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않을 때를 말한다.  
이펙티브 자바 아이템 26에 '로 타입은 사용하지 말라' 로 자세히 설명되어 있다.  
```java
public static void reverse(List<?> list) {
    List temp = new ArrayList<>(list);
    List list2 = list;
    for (int i = 0; i < list.size(); i++) {
        list2.set(i, temp.get(list.size() - i - 1));
    }
}
```
컴파일 하기도 전에 IDE에서 경고가 뜨며, 당연히 1번을 사용해야 한다고 생각할 것이다.  
그런데 자바 API 설계자들은 1번 방식의 메서드 오버헤드 문제 때문인지 2번 방식을 많이 사용한다.  
Collections의 swap 메서드를 보면 다음과 같다.  
```java
public static void swap(List<?> list, int i, int j) {
        // instead of using a raw type here, it's possible to capture
        // the wildcard but it will require a call to a supplementary
        // private method
    final List l = list;
    l.set(i, l.set(j, l.get(i)));
}
```
1번 방식의 wildcard 캡처를 사용할 수 있지만 추가적인 private method를 사용해야 한다고 언급하고 있다.