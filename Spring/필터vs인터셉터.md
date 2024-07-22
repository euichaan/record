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
  
