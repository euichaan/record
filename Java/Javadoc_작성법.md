# Javadoc 작성법
@see vs @link

기능적인 차이점은 다음과 같다.
- {@link} 는 인라인 링크로, 설명 중 어디에나 삽입할 수 있다.
- @see는 별도의 섹션으로 생성된다.

from stackoverflow
> {@link}는 설명 안에서 클래스, 필드, 생성자, 메서드 이름을 직접 언급할 때 사용하는 것이 가장 좋다고 생각한다. 이 경우 독자는 링크를 클릭해 해당 항목의 Javadoc으로 바로 이동할 수 있다.
> 
> 반면에, 저는 @see를 다음 두 가지 경우에 사용한다.
> 1. 설명에서 직접 언급하지 않았지만 매우 관련성이 높은 항목을 추가적으로 참고할 때 
> 2. 설명 중 동일한 항목을 여러 번 참조해야 하는 경우, 여러 번의 {@link} 대신 @see로 한 번에 처리할 때.

---
아래 내용은 다음 글을 참고했습니다.
https://johngrib.github.io/wiki/java/javadoc/  
https://www.oracle.com/technical-resources/articles/java/javadoc-tool.html

### 메서드 Javadoc
- 메서드가 무엇을 입력받아서 무엇을 리턴하는지를 반드시 설명한다.
- 메서드가 어떤 경우에 어떤 예외를 던지는지를 케이스별로 설명한다.
- **구현에 대해서는 설명하지 않는다.** (구현이 바뀌면 주석도 바뀌게 된다)
### 클래스 Javadoc
- 이 클래스의 책임 또는 목표가 무엇인지를 설명한다.

> 이해하는데 주석이 필요없을 정도로 코드를 명확하게 작성할 수 있다고 자기 자신을 속이지 마십시오.

# main description
### 리턴값을 설명하는 형식의 main description
```java
// 싫음: 리턴값이 무엇인지를 설명하지 않는다.
/**
 * 문자열이 문자들의 시퀀스 s를 포함하는지 확인합니다.
 */
public boolean contains(CharSequence s) {
```

```java
// 좋음: 무엇을 리턴하는지 명확히 표현한다.
/**
 * 문자열이 문자들의 시퀀스 s를 포함한다면 true를 리턴하고, 그렇지 않다면 false를 리턴합니다.
 */
public boolean contains(CharSequence s) {
```
### 예외 클래스라면 어떤 경우에 던지는지 설명한다.
### 구현에 의존하지 않는다.
**Unless otherwise noted, the Java API Specification assertions need to be implementation-independent. Exceptions must be set apart and prominently marked as such.**

달리 명시되지 않는 한, Java API 사양 선언은 구현에 독립적이어야 한다.
예외는 별도로 설정하고 이를 두드러지게 표시해야 한다.