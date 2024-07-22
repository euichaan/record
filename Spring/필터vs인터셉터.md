# 필터(Filter) vs 인터셉터(Interceptor) 차이 및 용도
스프링 시큐리티를 공부하던 도중 Architecture 부분을 살펴보니  
> 스프링 시큐리티는 FilterChainProxy라는 특별한 필터를 추가하는데, 이 필터는 DelegatingFilterProxy를 통해 서블릿 필터에 등록되고 FilterChainProxy는 SecurityFilterChain을 통해 여러 필터 인스턴스에 작업을 위임한다.  
  
라는 내용이 있어 왜 스프링 시큐리티는 인터셉터가 아닌 필터를 사용하는지?에 대한 궁금증이 생겼다.  
[필터(Filter) vs 인터셉터(Interceptor) 차이 및 용도 - (1)](https://mangkyu.tistory.com/173)  
아래 내용은 이 글을 읽고 정리한 글이다.  
  
## 필터(Filter)
**필터는 디스패처 서블릿에 요청이 전달되기 전/후에 url 패턴에 맞는 모든 요청에 대해 부가작업을 처리할 수 있는 기능을 제공한다.** 즉, 스프링 컨테이너가 아닌 톰캣과 같은 웹 컨테이너(서블릿 컨테이너)에 의해 관리가 되는 것이고(스프링 빈으로 등록은 된다), 디스패처 서블릿 전/후에 처리하는 것이다.  
참고로, 스프링 빈으로 등록은 `DelegatingFilterProxy`를 통해서 가능하다.  
![R1280x0](https://github.com/user-attachments/assets/c7daa5ea-4ae2-44e6-915d-e858647152bd)  
  
필터를 추가하기 위해서는 jakarta.servlet의 Filter 인터페이스를 구현해야 한다.  
```java
public interface Filter {
    default void init(FilterConfig filterConfig) throws ServletException {
    }

    void doFilter(ServletRequest var1, ServletResponse var2, FilterChain var3) throws IOException, ServletException;

    default void destroy() {
    }
}
```
- init: 필터 객체를 초기화하고 서비스에 추가하기 위한 메서드이다. 서블릿 컨테이너가 1회 init 메서드를 호출하여 필터 객체를 초기화하면 이후의 요청들은 doFilter를 통해 처리된다.  
- doFilter: url-pattern에 맞는 모든 HTTP 요청이 디스패처 서블릿으로 전달되기 전에 웹 컨테이너에 의해 실행되는 메서드이다. doFilter의 파라미터로 FilterChain이 있는데, FilterChain의 doFilter를 통해 다음 대상으로 요청을 전달하게 된다. chain.doFilter() 전/후에 필요한 처리 과정을 넣어줌으로써 원하는 처리를 진행할 수 있다.  
- destroy: 필터 객체를 서비스에서 제거하고 사용하는 자원을 반환하기 위한 메서드이다. 이는 서블릿 컨테이너에 의해 1번 호출되며 이후에는 이제 doFilter에 의해 처리되지 않는다.  
## 인터셉터(Interceptor)
**Spring이 제공하는 기술로써, 디스패처 서블릿이 컨트롤러를 호출하기 전과 후에 요청과 응답을 참조하거나 가공할 수 있는 기능을 제공한다.** 즉, 서블릿 컨테이너에서 동작하는 필터와 달리 인터셉터는 스프링 컨텍스트에서 동작을 하는 것이다.  
  
디스패처 서블릿은 핸들러 매핑을 통해 적절한 컨트롤러를 찾도록 요청하는데, 그 결과로 HandlerExecutionChain을 돌려준다. HandlerExecutionChain은 1개 이상의 인터셉터가 등록되어 있다면 순차적으로 인터셉터들을 거쳐 컨트롤러가 실행되도록 하고, 인터셉터가 없다면 바로 컨트롤러를 실행한다.
![R1280x0-2](https://github.com/user-attachments/assets/81fe09b3-fc09-44e5-aec4-d05025ebf0c1)
  
실제로는 interceptor가 Controller로 요청을 위임하지는 않는다. 처리 순서를 도식화한 것으로만 이해하자.  
  
```java
public interface HandlerInterceptor {
    default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        return true;
    }

    default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable ModelAndView modelAndView) throws Exception {
    }

    default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable Exception ex) throws Exception {
    }
}
```
인터셉터를 추가하기 위해서는 org.springframework.web.servlet 의 HandlerInterceptor 인터페이스를 구현해야 한다.  
  
- preHandle: 컨트롤러가 호출되기 전에 실행된다. 컨트롤러 이전에 처리해야 하는 전처리 작업이나 요청 정보를 가공하거나 추가하는 경우에 사용할 수 있다. 반환값이 true이면 다음 단계로 진행이 되지만, false라면 작업을 중단하여 이후의 작업은 진행되지 않는다.   
- postHandle: 컨트롤러를 호출된 후에 실행된다. 컨트롤러 하위 계층에서 작업을 진행하다가 중간에 예외가 발생하면 postHandle은 호출되지 않는다.  
- afterCompletion: 모든 작업이 완료된 후에 실행된다. postHandle과 달리 컨트롤러 하위 계층에서 작업을 진행하다가 중간에 예외가 발생하더라도 afterCompletion은 반드시 호출된다.  
  
## 필터(Filter) vs 인터셉터(Interceptor) 차이 및 용도
![R1280x0-3](https://github.com/user-attachments/assets/a5146b78-f98b-42cd-984d-444064296417)
  
필터는 스프링 이전의 서블릿 영역에서 관리되지만, 인터셉터는 스프링 영역에서 관리되는 영역이기 때문에 필터는 스프링이 처리해주는 내용들을 적용 받을 수 없다. 이로 인한 차이로 발생하는 대표적인 예시가 **스프링에 대한 예외처리가 되지 않는다**는 것이다.  
  
필터는 Request와 Response를 조작할 수 있지만 인터셉터는 조작할 수 없다. 여기서 조작한다는 것은 내부 상태를 변경한다는 것이 아니라 다른 객체로 바꿔친다는 의미이다.  
```java
public MyFilter implements Filter {

    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
        // 개발자가 다른 request와 response를 넣어줄 수 있음
        chain.doFilter(new MockHttpServletRequest(), new MockHttpServletResponse());       
    }
    
}
```
인터셉터는 true를 반환하면 다음 인터셉터가 실행되거나 컨트롤러로 요청이 전달되며, false가 반환되면 요청이 중단된다. 그러므로 우리가 다른 Request/Response 객체를 넘겨줄 수 없다.  
```java
public class MyInterceptor implements HandlerInterceptor {

    default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        // Request/Response를 교체할 수 없고 boolean 값만 반환할 수 있다.
        return true;
    }

}
```
필터의 용도 및 예시
- 공통된 보안 및 인증/인가 작업  
- 모든 요청에 대한 로깅  
- 이미지/데이터 압축 및 문자열 인코딩  
- Spring과 분리되어야 하는 기능  
  
필터에서는 기본적으로 **스프링과 무관하게 전역적으로 처리해야 하는 작업**들을 처리할 수 있다.  
대표적으로 **보안 공통 작업**이 있다. 필터는 인터셉터보다 앞단에서 동작하므로 전역적으로 해야하는 보안 검사(XSS 방어 등)를 하여 올바른 요청이 아닐 경우 차단을 할 수 있다. 그러면 스프링 컨테이너까지 요청이 전달되지 못하고 차단되므로 안정성을 더욱 높일 수 있다.  
또한 필터는 이미지나 데이터의 압축이나 문자열 인코딩과 같이 웹 애플리케이션에 전반적으로 사용되는 기능을 구현하기에 적당하다. Filter는 다음 체인으로 넘기는 ServletRequest/ServletResponse 객체를 조작할 수 있다는 점에서 Interceptor보다 훨씬 강력한 기술이다.  
  
인터셉터의 용도 및 예시  
- 세부적인 보안 및 인증/인가 공통 작업  
- API 호출에 대한 로깅  
- Controller로 넘겨주는 정보의 가공  
  
인터셉터는 클라이언트의 요청과 관련하여 전역적으로 처리해야 하는 작업들을 처리할 수 있다.  
대표적으로 세부적으로 적용해야 하는 인증이나 인가와 같이 클라이언트 요청과 관련된 작업 등이 있다.  
HttpServletRequest나 HttpServletResponse등과 같은 객체를 제공받으므로 객체 자체를 조작할 수는 없다. 대신 해당 객체가 내부적으로 갖는 값은 조작할 수 있으므로 컨트롤러로 넘겨주기 위한 정보를 가공하기에 용이하다. 그 외에도 다양한 목적으로 API 호출에 대한 정보들을 기록해야 할 수 있다. 이러한 경우에 HttpServletRequest나 HttpServletResponse를 제공해주는 인터셉터는 클라이언트 IP나 요청 정보를 포함해 기록하기에 용이하다.  
  
스프링 시큐리티가 필터를 사용하는 주된 이유는 **필터가 HTTP 요청의 초기 단계에서 전역적으로 동작하여 일관된 보안 처리를 제공하기 때문이다.** 이는 서블릿 컨테이너 수준에서 표준화된 메커니즘으로 동작하며, 다양한 보안 시나리오를 효율적으로 처리할 수 있는 장점을 제공한다.  
반면, 인터셉터는 주로 스프링 MVC의 컨트롤러 호출 전후로 동작하기 때문에, 전역적인 보안 처리를 위해서는 필터가 더 적합하다.
