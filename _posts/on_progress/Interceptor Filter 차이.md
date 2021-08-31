# Interceptor Filter 차이

HandlerInterceptor는 handler 스스로의 실행을 금지하는 옵션과 함께 pre & post processing이 가능하게 함

Filter는 거 강력함, request와 response 자체를 바꿀수 있음

filter는 `web.xml`에서, HandlerInterceptor는 `ApplicationContext`에서 설정 정보를 가져옴

권한 체크, 세션 같은 전처리 작업에는 interceptor,
요청, 응답 값 조작에는 Filter

`Interceptor.postHandle()` 과 `Filter.doFilter()`의 차이

`postHandle()`은 뷰가 랜더되기 전에 실행됌, 뷰에 오브젝트들을 추가 할 수는 있지만
`HttpServletResponse`는 이미 committed 됐기에 바꿀 수 없음

Filter
HTTP request와 response에 대해 Servlet Container로 인해 실행됌
자원에 접근하기전에 HTTP request 관리가 가능함
자원 실행이 끝난 후에 response 관리 또한 가능 

Interceptor
Spring Context에서 작동, HTTP Request 와 Response에 대한 관리가 가능함
Spring context에 접근이 가능하기 때문

구조 이미지
https://miro.medium.com/max/3716/1*TOMg7ZPmHaklt4RW4JJ0Ew.png
Filter는 web Container안에
web container안에는 spring context도 있음
이 spring context안에 인터셉터가 있음 디스패처 서블릿 다음 순서 

Filter 구현
Filter interface를 구현
`@Order(1)`를 통해 Filter chain의 순서를 정할 수 있음
`chain.doFilter`를 실행 다음 필터로 넘어감
Fileter에서는 ip, request params, payload(request body)에 접근이 가능함
request body는 read only `HttpServletRequestWrapper`를 통해 read only를 방지

Filter와 Interceptor의 예외처리
Filter에서 예외가 발생하면 Web Application에서 처리해야 함. 
에러페이지를 잘 설정하던지, 예외를 잡아서 디스패쳐한테 떠넘겨서 처리해야함
Interceptor에서 예외가 발생하면, Spring의 servlet dispatcher 내부이기 때문에
`@ControllerAdvice`에서 `@ExceptionHandler`를 통해 처리 가능

Filter에서만 가능한 것
`ServletRequest`와 `ServletResponse`의 교체
`HttpServletRequest`의 `ServletInputStream`(body) 내용을 한번 read 가능, `HttpServletRequestWrapper`를 통해 inputStream을 여러번 읽을 수 있게 커스터마이징