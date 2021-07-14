---
title: "내가 보려고 쓰는 Spring AOP"
date: 2021-02-02 22:50:00 +0900
category: springboot
lastmod : 2021-02-02 22:50:00 +0900
sitemap :
  changefreq : weekly
  priority : 1.0
---

## AOP 란

**Aspect-Oriented Programming**(관점 지향 프로그래밍)는 **Cross-cutting Concern**(황단 관심사)를 이용하여 **핵심** 관심사와 **공통** 관심사를 분리하여 **프로그램의 모듈성을 증가**시키는 프로그래밍 기법이다.  
  
핵심 관심사에는 비지니스 로직이 해당되며, 해당 비지니스 로직의 **영향 혹은 수정 없이**, 여러 모듈에서 사용할 수 있는 **공통 로직을 적용**하는 방법이다.  
  
데이터 베이스 트랜잭션을 설정하는 `@Transactional`이 대표적인 AOP aspect라고 할 수 있다.

## AOP 구성 요소

### Business Object

- 평범한 비지니스 로직을 가지고 있는 클래스. Spring 관련 어노테이션이 붙지 않은 보통 클래스

### Aspect

- 여러 클래스를 통해 발생될 수 있는 모듈화 된 관점

### JoinPoint

- 프로그램 실행 중 Aspect가 발생하는 포인트
- 메소스 실행 혹은 예외 처리 시점이 JoinPoint

> Spring AOP에서는 매소드 실행이 JoinPoint

### PointCut

- 포인트 컷은 특정 JoinPoint에 Aspect로 인해 적용될 Advice을 매치 시켜주는 속성
- 포인트식 표현식을 통해, Aspect가 적용될 JoinPoint를 필터링한다

### Advice

- 특정 JoinPoint에서 aspect로 인해 실행될 액션
- `Around`, `Before`, `After`, `AfterReturning`, `AfterThrowing`이 있음
    > 실행 순서  
    > Around -> Before -> AfterReturning, AfterThrowing -> After -> Around
- Spring에서 Advice는 인터셉터처럼 모델화 되어있음, 인터셉터 체인을 포함

## Spring AOP 구현 예제

> **CASE**  
> 카드사 연동을 통한 결제 모듈을 만들고 있다.  
> 카드사에 결제 요청을 보내기전 모든 거래 정보는 거래 상태를 `requsting`으로 저장한다.  
> 카드사에 보낸 결제 요청이 성공하면 `success`, 실패할 경우 `failed`로 저장한다.  
> 또한, 카드사 연동 결제 요청은 총 처리시간을 로그에 남겨야 한다.  

- [구현 전체 코드 참고](https://github.com/kingjakeu/spring-study/tree/aop-study)  

우선 Aspect를 적용할 JointPoint에 구분하여 표시할 Custom Annotation 만든다.

``` java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface RequestingPayment {
}
```

이후 해당 Aspect를 통해 처리할 공통 로직을 구현한다.  
`@RequestingPayment` 어노테이션은 PointCut 표현식에 들어가 적용 Aspect를 필터링한다.

``` java
@Slf4j
@Aspect
@Component
@RequiredArgsConstructor
public class RequestingPaymentAspect {

  private final PaymentTxInfoRepository paymentTxInfoRepository;

  // @RequestingPoint가 달린 메소드만 적용한다.
  // Joint Point가 실행되기 전
  @Before("@annotation(com.kingjakeu.springstudy.domain.payment.annotation.RequestingPayment)")
  public void onBefore(JoinPoint joinPoint) { 
      PaymentTxInfo paymentTxInfo 
                = (PaymentTxInfo)joinPoint.getArgs()[0]; // 메소드 첫번째 파라미터
      paymentTxInfo.setStatus("requesting"); // 상태 변경
      this.paymentTxInfoRepository.save(paymentTxInfo); // 저장
  }

  // Joint Point가 실행된 후 
  @AfterReturning("@annotation(com.kingjakeu.springstudy.domain.payment.annotation.RequestingPayment)")
  public void onAfterReturning(JoinPoint joinPoint){
      PaymentTxInfo paymentTxInfo = (PaymentTxInfo)joinPoint.getArgs()[0];
      paymentTxInfo.setStatus("success");
      this.paymentTxInfoRepository.save(paymentTxInfo);
  }

  // Joint Point가 실행되고 에러가 발생했을 때
  @AfterThrowing("@annotation(com.kingjakeu.springstudy.domain.payment.annotation.RequestingPayment)")
  public void onAfterTrowing(JoinPoint joinPoint){
      PaymentTxInfo paymentTxInfo = (PaymentTxInfo)joinPoint.getArgs()[0];
      paymentTxInfo.setStatus("failed");
      this.paymentTxInfoRepository.save(paymentTxInfo);
  }

  // JointPoint 실행 전, 후
  @Around("@annotation(com.kingjakeu.springstudy.domain.payment.annotation.RequestingPayment)")
  public Object around(ProceedingJoinPoint joinPoint) throws Throwable { 
      long currentTime = System.currentTimeMillis();
      Object proceed = joinPoint.proceed(); // JointPoint 실행
      log.info(String.valueOf(System.currentTimeMillis() - currentTime));
      return proceed;
  }
}
```

이렇게 Aspect를 만들고 해당 모듈에 `@RequestingPayment`를 통해 Aspect를 적용한다.

``` java
  @RequestingPayment
  public void requestCardPayment(PaymentTxInfo paymentTxInfo){
      // 카드 결제 요청
  }
```
