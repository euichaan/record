# 필터(Filter) vs 인터셉터(Interceptor) 차이 및 용도
스프링 시큐리티를 공부하던 도중 Architecture 부분을 살펴보니  
> 스프링 시큐리티는 FilterChainProxy라는 특별한 필터를 추가하는데, 이 필터는 DelegatingFilterProxy를 통해 서블릿 필터에 등록되고 FilterChainProxy는 SecurityFilterChain을 통해 여러 필터 인스턴스에 작업을 위임한다.  
  
라는 내용이 있어 스프링 시큐리티는 인터셉터가 아닌 필터를 사용하는지에 대한 궁금증이 생겼다.  
[필터(Filter) vs 인터셉터(Interceptor) 차이 및 용도 - (1)](https://mangkyu.tistory.com/173)  
이 글을 읽고 정리한 글.  
  
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

