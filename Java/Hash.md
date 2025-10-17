# Hash
- 해시 함수는 임의의 길이의 데이터를 입력으로 받아, 고정된 길이의 해시값(해시 코드)을 출력하는 함수이다.  
    - 고정된 길이는 저장 공간의 크기를 뜻한다. 예를 들어 int형 1,100은 둘다 4byte를 차지하는 고정된 길이를 뜻한다.  
- 같은 데이터를 입력하면 항상 같은 해시 코드가 출력된다.  
- 다른 데이터를 입력해도 같은 해시 코드가 출력될 수 있다. 이것을 해시 충돌이라 한다.  
  
해시 코드(Hash Code)  
- 해시 코드는 데이터를 대표하는 값을 뜻한다. 보통 해시 함수를 통해 만들어진다.  
  
해시 인덱스(Hash Index)  
- 해시 인덱스는 데이터의 저장 위치를 결정하는데, 주로 해시 코드를 사용해서 만든다.  
- 보통 해시 코드의 결과에 배열의 크기를 나누어 구한다.  
  
## Object.hashCode()
자바는 모든 객체가 자신만의 해시 코드를 표현할 수 있는 기능을 제공한다.  
```java
public class Object {
    public int hashCode();
}
```
- 이 메서드의 기본 구현은 객체의 참조값을 기반으로 해시 코드를 생성한다.  
- 객체의 인스턴스가 다르면 해시 코드도 다르다.  
  
## equals와 hashCode
해시 자료 구조를 사용하려면 hashCode()도 중요하지만 해시 인덱스가 중요할 경우를 대비해서 equals()도 반드시 재정의해야 한다.  
해시 인덱스가 충돌할 경우 같은 해시 인덱스에 있는 데이터들을 하나하나 비교해서 찾아야한다. 이때 equals()를 사용해서 비교한다.  
  
Object는 기본적으로 hashCode(), equals()를 다음과 같이 정의한다.  
`hashCode()`: 객체의 참조값을 기반으로 해시 코드를 반환한다.  
`equals()`: == 동일성 비교를 한다. 따라서 객체의 참조값이 같아야 true를 반환한다.  
  
해시 자료 구조를 사용하는 경우 hashCode()와 equals()를 반드시 함께 재정의해야 한다.  
해시 함수는 같은 입력에 대해서 항상 동일한 해시 코드를 반환해야 한다.  
## equals를 재정의하려거든 hashCode도 재정의하라 (Effective Java Item 11)
hash 값을 사용하는 Collection(HashMap, HashSet, HashTable)은 객체가 논리적으로 같은지 비교할 때 아래 그림과 같은 과정을 거친다.  
<img width="1766" height="489" alt="Image" src="https://github.com/user-attachments/assets/eb14b897-bed4-4b6d-8c64-1914fc54871e" />  
  
hashCode 메서드의 리턴 값이 우선 일치하고 equals 메서드의 리턴 값이 true여야 논리적으로 같은 객체라고 판단한다.  
  
**equals를 재정의한 클래스 모두에서 hashCode도 재정의해야 한다.** 그렇지 않으면 hashCode 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 HashMap이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킬 것이다.  
```
Object 명세의 규약  
- equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다.
- equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.  
- equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.  
```
hashCode 재정의를 잘못했을 때 크게 문제가 되는 조항은 두 번째다. 즉, 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.  
```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "제니");
```
이 코드 다음에 `m.get(new PhoneNumber(707, 867, 5309));`를 실행하면 제니가 나와야 할 것 같지만, 실제로는 null을 반환한다. PhoneNumber 클래스는 hashCode를 재정의하지 않았기 때문에 논리적 동치인 두 객체가 서로 다른 해시코드를 반환하여 두 번째 규약을 지키지 못한다. 그 결과 get 메서드는 엉뚱한 해시 버킷에 가서 객체를 찾으려 한 것이다.  
설사 두 인스턴스를 같은 버킷에 담았더라도 get 메서드는 여전히 null을 반환하는데, HashMap은 해시코드가 다른 엔트리끼리는 동치성 비교를 시도조차 하지 않도록 최적화되어 있기 때문이다.  
  
이 문제는 PhoneNumber에 적절한 hashCode 메서드만 작성해주면 해결된다. 올바른 hashCode 메서드는 어떤 모습이어야 할까?  
  
```java
// 최악의 (하지만 적법한) hashCode 구현 - 사용 금지!
@Override
public int hashCode() { return 42; }
```
이 경우 동등한 모든 객체가 해시테이블의 버킷 하나에 담겨 마치 연결 리스트(linked list)처럼 동작한다.  
그 결과 평균 수행 시간이 O(1)인 해시테이블이 O(n)으로 느려져서, 객체가 많아지면 도저히 쓸 수 없게 된다.  
  
좋은 해시 함수라면 서로 다른 인스턴스에 다른 해시코드를 반환한다.  
**이상적인 해시 함수는 주어진 (서로 다른) 인스턴스들을 32비트 정수 범위에 균일하게 분배해야 한다.**  
  
Objects 클래스는 임의의 개수만큼 객체를 받아 해시코드를 계산해주는 정적 메서드인 hash를 제공한다.  
```java
@Override
public int hashCode() { 
    return Objects.hash(lineNum, prefix, areaCode);
}
```
하지만 아쉽게도 속도는 더 느리다. 입력 인수를 받기 위한 배열이 만들어지고, 입력 중 기본 타입이 있다면 박싱과 언박싱도 거쳐야 하기 떄문이다. 그러니 hash 메서드는 성능에 민감하지 않은 상황에서만 사용하자.  
  
**성능을 높인답시고 해시코드를 계산할 때 핵심 필드를 생략해서는 안 된다.**  
  
다음은 좋은 hashCode를 작성하는 간단한 요령이다.  
[Guide to hashCode() in Java](https://www.baeldung.com/java-hashcode)  