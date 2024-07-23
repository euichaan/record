# 필터가 스프링 빈 등록과 주입이 가능한 이유 - DelegatingFilterProxy
필터는 서블릿이 제공하는 기술이므로 서블릿 컨테이너에 의해 생성되며 서블릿 컨테이너에 등록이 된다. 그렇기 때문에 스프링의 빈으로 등록도 불가능했으며, 빈을 주입받을 수 없었다.  
하지만 필터에서도 DI와 같은 스프링 기술을 필요로 하는 상황이 발생하면서, 필터도 빈으로 주입받을 수 있도록 대안을 마련하였는데, 그것이 바로 `DelegatingFilterProxy`이다.  
  
DelegatingFilterProxy는 서블릿 컨테이너에서 관리되는 프록시용 필터로써 우리가 만든 필터를 가지고 있다. 우리가 만든 필터는 스프링 컨테이너의 빈으로 등록되는데, **요청이 오면 DelegatingFilterProxy가 요청을 받아서 우리가 만든 필터(스프링 빈)에게 요청을 위임한다.**  
  
1. Filter 구현체가 스프링 빈으로 등록됨  
2. ServletContext가 Filter 구현체를 갖는 DelegatingFilterProxy를 생성함  
3. ServletContext가 DelegatingFilterProxy를 서블릿 컨테이너에 필터로 등록함  
4. 요청이 오면 DelegatingFilterProxy가 필터 구현체에게 요청을 위임하여 필터 처리를 진행함  
  
Spring Boot가 아니라면 서블릿 컨텍스트에 addFilter로 추가해주어야 하지만, Spring Boot라면 그럴 필요가 없다.  
Spring Boot가 내장 웹서버를 지원하면서 톰캣과 같은 서블릿 컨테이너까지 Spring Boot가 제어 가능하기 때문이다. **Spring Boot가 서블릿 필터의 구현체 빈을 찾으면 DelegatingFilterProxy 없이 바로 필터 체인에 필터를 등록해준다.**    
  
요약하자면, DelegatingFilterProxy의 등장 이후 DelegatingFilterProxy를 통해 스프링 빈으로 등록 및 다른 빈 주입이 가능해졌다. Spring Boot의 등장 이후 서블릿 컨테이너를 직접 제어해서 DelegatingFilterProxy를 사용하지 않고도 필터를 쉽게 등록할 수 있다.  