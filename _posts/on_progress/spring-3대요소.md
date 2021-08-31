# Spring 3대 요소

## 제어 역전 Inversion of Control

일반적인 자바 프로그래밍의 경우, 개발자가 직접 의존성을 만들어 사용함
하지만 스프링에선 Dependency Injection(의존성 주입)을 프레임워크에 맡겨, 의존성은 외부에서 주입이 됌
즉 DI의 주체가 바뀌면서 IoC 제어의 역전이 일어남

제어 역전의 장점은 인스턴스 생성 비용이 크거나, 복잡 할 경우 직접 하지 않아도 된다는 점
그리고 인스턴스의 생성과 라이프 사이클 관리를 IoC 컨테이너가 대신 해준다는 점

IoC 컨테이너의 역할을 하는 것이 `ApplicationContext`
그리고 `ApplicationContext`이 만들고 관리하는 객체를 `Bean`

빈의 생성

- 어노테이션으로 명시, 컴포넌트 스캔을 통해 생성하는 방법
- XML, 자바 설정 파일을 통해 직접 등록하는 방법(ex. configuration 밑에 bean들)

## AOP 관점 지향 프로그래밍

핵심 관심사(비지니스 로직)과 공통 로직(ex. 로깅, 트랜잭션)을 분리하여 모듈성을 증가시키는 기법
스프링은 프록시 패턴을 통해 AOP를 사용함, 즉 흐름제어만 하지, 반환결과를 바꾸지는 않음

## PSA Portable Service Abstraction 서비스 추상화

외부 라이브러리들을 POJO로 사용할 수 있도록 추상화 한 것
Spring이 인터페이스로 의존성을 주입 받아 Low Level 코드를 알아서 처리해 주는 것

스프링은 서블릿 어플리케이션이지만 서블릿 코드를 생성하지 않음
doPost, doGet가 아닌 @GetMapping, @PostMapping 어노테이션을 통해 처리함
실제로는 서브릿으로 코드가 동작하지만 Service Abstraction 서비스 추상화를 통해 안보이는 것분

추상화 계층을 통해 서블릿을 직접 쓰지 않아도 되고, 톰캣->네티 처럼 여러가지 기술로 교체 가능

@Transactional을 사용하면, setAutoCommit(false); 나 에러 캐치 후 rollback(); 하는 행위들을 모두 한번에 처리한다

---

## DispatcherServlet

클라이언트로부터 요청이 오면 제일 먼저 요청을 처리하는 Front Contoller
HanlderMapping을 통해 해당 리퀘스트를 처리 할 수 있는 컨트롤러를 지정해두면
DispatcherServlet은 해당 컨트롤러를 찾아 HandlerAdaptor의 매소드를 통해 실제 메소드를 실행 함
결과를 출력할 View를 ViewResolver를 통해 찾고 해당 View를 응답

---

## Web Server and WAS

Web Sever
클라이언트로부터 HTTP 요청을 받아 정적인 컨텐츠를 제공

WAS
DB처리 등 로직 처리를 요구하는 동적 컨텐츠를 제공하는 Application Server

분리하는 이유

- 부하 방지
  로직이 필요없는 정적 컨텐츠까지 was에서 처리하면 자원 부족
- 보안 강화
  SSL 암복호화 처리는 웹서버가
- 여러대의 WAS 이용 가능


## Servlet JSP 차이

servlet - 자바 코드안에 html
jsp - html안에 자바 코드

추상클래스는 동일한 부모를 가진 클래스를 묶어 상속을 받아 기능을 확장 시키는 것이 목적
인터페이스는 구현 객체가 같은 동작을 하는 것을 보장하는 것이 목적


## Git

Git은 분산형 버전 관리 시스템

## Git의 필요성

협업이 쉬워진다
같은 소스를 여러명이 동시 작업 할 수 있고 개발 후에는 머지로 합쳐
여러명이 동시 작업할 수 있는 병렬 개발이 가능하다.
또한 커밋 내역을 통해 무엇을 수정 했는지 알 수 있으며, conflict를 통해 남의 코드를 덮어 씌우는 문제를 예방 해줌

Git 플로우를 통한 버전관리, 배포 용이성
깃플로우을 통해 배포까지의 과정을 효과적으로 정립 할 수 있음
태그를 통해 버전관리, 각 환경 별 테스트, 롤백의 용이성, 브랜치 생성을 통한 빠른 핫 픽스


POST GET 차이
