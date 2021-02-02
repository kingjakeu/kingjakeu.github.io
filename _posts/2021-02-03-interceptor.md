---
title: "내가 보려고 쓰는 Spring Interceptor"
date: 2021-02-03 00:50:00 +0900
category: springboot
lastmod : 2021-02-03 00:50:00 +0900
sitemap :
  changefreq : weekly
  priority : 1.0
---

<br>

## Spring Interceptor란

---

Spring Interceptor란 **요청이 처리 되기 전(Before handling), 요청이 처리된 후(after handling), 요청이 완료된 후(after completion)** 세가지 시점에 요청을 intercept(가로채) 처리하는 모듈이다.

**HandlerMapping**을 통해 Controller 메소드에 URL을 매핑하면,  
**DispatcherServlet**은 해당 리퀘스트를 처리할 때, **HandlerAdapter**를 통해 매핑된 **실제 메소드를 실행**한다.  

또한 Interceptor를 이용, cross-cutting 이슈, 공통 로깅, 전역 적용 파라미터 처리등의 코드 중복을 막을 수 있다.  

> 실행 순서  
> Filter -> DispatcherServlet -> Interceptor -> Controller

### Servlet이란

- Host 애플리케이션이 request-response 방법의 프로그래밍 모델로 접근할 수 있도록, 서버의 확장성을 높히는데 사용되는 자바 클래스
- 서블렛이 모든 타입의 request에 응답 할 수 있지만, 보통 웹 서버를 호스팅하는 애플리케이션에서 확장되어 사용. ex. HTTP
- `javax.servlet`와 `javax.servlet.http` package가 servlet에 사용되는 인터페이스를 제공
- 모든 서블렛은 서블렛의 라이프 사이클을 정의 해둔 Servlet 인터페이스를 구현
- HttpServlet 클래스를 통해 doGet/doPost와 같은 HTTP 용 서비스들을 다룰 수 있다

---

<br>

## HandlerInterceptor

---

Interceptor는 HandlerMapping에 따라 작동하기 때문에 `HandlerInterceptor` 인터페이스를 implement 해야한다.

### HandlerInterceptor의 세 개의 메소드

| 메소드 명 | 설명 |
|:--------|:------|
| preHandle() | 핸들러가 실행되기 전 실행 |
| postHandle() | 핸들러가 실행된 후 실행 |
| afterCompletion() | 리퀘스트가 끝나고, 뷰가 생성 됐을 때 실행 |

- **preHandle**
    - `return true`는 다음 프로세스(리퀘스트 처리)를 진행하는 것
    - `false`는 더이상 리퀘스트를 진행하지 않고 끝냄

- **postHandle**
    - 리퀘스트가 HandlerAdapter에 의해 처리된 후 바로 실행되며, View는 아직 생성 되기 전

- **afterCompletion**
    - 뷰가 생성되고, 리퀘스테에 관련된 추가적인 hook을 걸어 줄 때 사용(ex. load time)

<br>

### HandlerInterceptor 적용 방법

`HandlerInterceptetor`는 `DefaultAnnotationHandlerMapping` bean에 등록된다.  
`DefaultAnnotationHandlerMapping` Bean은 `@Controller`이 달려있는 클래스에 인터셉터들을 적용한다.  

<br>

### HandlderInterceptor와 HanlerInterceptorAdapter의 차이

`HandlerInterceptor`는 `preHandle(), postHandle(), afterCompletion()` 세 개의 매소드를 모두 오버라이드 해야한다.  
하지만 `HanlerInterceptorAdapter`는 세 개의 메소드 중 필요한 것만 오버라이드한다.  

---

<br>

## HandlerAdaptor

---

`HandlerAdaptor`는 HTTP Request들을 Spring MVC에서 유연한 방법으로 처리하게 해주는 기본 인터페이스이다.

DispatcherServlet이 `HandlerAdapter`의 HandlerMapping 메소드를 실행하는 것으로 HandlerMapping과 결합되어 사용된다.

Servlet은 메소드를 직접 실행하지 않고, Loosely Coupling Design으로 Servlet과 Handler사이의 브릿지만 제공하기 때문에 HandlerAdaptor가 Controller에 매핑된 메소드를 실행한다.

### HandlerAdaptor 구성

- support API
    - 특정 Handler 인스턴스의 support 여부를 체크  
    - support()는 handle()을 부르기 전에 제일 먼저 불려지는 메소드

- handle API
    - 특정 HTTP request를 handle 역할
    - HttpServletRequest와 HttpServeltResponse를 파라미터로 Handler를 실행하는데 책임이 있음
    - 핸들러는 애플리케이션 로직을 실행하고 DispatcherServlet를 통해 처리되는 ModelAndView 객체를 return한다.

<br>

### RequestMappingHandlerAdapter란

@RequestMapping으로 annotated된 메소드를 실행 할때 사용되는 HandlerAdapter이다.  
Request uri를 핸들러에 매핑되어 유지되는데 사용된다.  
Handler가 obtained 되면 DispatcherServlet은 request를 알맞은 Handler Adapter에 handlerMethod()를 통해 dispatch한다.  

---

<br>
