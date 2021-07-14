---
title: "트랜잭션의 특징, 격리, 전파"
date: 2020-08-06 22:35:00 +0900
category: study
lastmod : 2021-01-06 23:35:00 +0900
sitemap :
  changefreq : weekly
  priority : 1.0
---

## 트랜잭션이란

데이터베이스의 상태를 변환하는 하나 이상의 데이터베이스 연산의 논리적 단위이다.  
  
DBMS는 하나의 트랜잭션에 여러 개의 연산이 **전부 처리되어 Commit**되거나 아니면 **전부 적용 되지 않도록 Rollback**한다.

## 트랜잭션의 특징 (ACID)

### Atomicity (원자성)

- 트랜잭션이 모두 반영되던가 아니면 전체가 반영되지 않는(**all or nothing**)을 보장하는 속성  

> **ex .**  
> A에서 3000원을 인출 -> B에 2000원 입금,  
> 나머지 1000원을 A -> C에 입금하는 로직이 하나의 트랜잭션에 묶여있다.  
>  
> A -> B는 성공, A -> C 도중 에러가 발생한다면, 트랜잭션의 **원자성**으로 인해 **처음 상태**인 A에 3000원이 있고, 나머지는 0원으로 **Rollback** 되어야 한다.

### Consistenecy (일관성)  

- 트랜잭션이 끝난 후에, 트랙잭션 시작 전과 **동일한 DB상태를 유지하는 일관성**을 보장하는 속성  

> **ex .**  
> A와 B 두 개의 테이블이 있다.  
> B는 A의 `ID` 필드를 `A_ID`라는 외래키 필드로 사용하고 있다.  
> `ID` 필드의 타입은 문자열이고 참조하는 `A_ID` 또한 그렇다.  
>  
> 여기서 만약 `ID` 필드의 타입을 숫자로 바꾼다면 `A_ID` 또한 바뀌어 **일관성**을 유지 해야한다.

### Isolation (고립성)

- 여러개의 트랜잭션이 **서로간의 간섭없이 개별적으로 동작**하는 속성

> **ex .**  
> 은행원이 A 계좌에서 돈을 인출하고 있다.  
> 
> A 계좌에서 돈을 인출하고 새로운 잔고를 갱신하는 트랜잭션이 끝나기 전까지 다른 은행원들은 A 계좌에 값을 바꿀 수 없도록 **고립성**을 유지해야 한다.

### Durability (영구성)  

- 성공한 끝난 트랜잭션은 **DB에 영구적으로 반영**되는 속성

> **ex .**  
> A 계좌 잔고를 3000원으로 바꾸는 트랜잭션이 성공적으로 끝났다.  
>  
> 이후 시스템 장애로 인해, 서버가 다운이 되었어도, 완료된 트랜잭션은 반영되어 **영구성**을 유지해야 한다.

## 트랜잭션 격리 수준

### READ_UNCOMMITTED

- 커밋되지 않은 **처리중인 데이터 읽기 허용**
- **Dirty Read**가 발생  
    - 다른 트랜잭션에서 커밋되지 않은 데이터를 읽어, **데이터의 정합성**이 어긋나는 문제

### READ_COMMITTED

- **커밋된 데이터만 읽기** 허용
- 오라클 기본 값
- **Non-Repeatable Read** 발생  
    - 하나의 트랜잭션이 데이터를 조회 했을떄 **항상 같은 결과가 아닌 다른 트랜잭션에 의해 변경된 값**을 읽어 오는 것

### REPEATABLE_READ

- 트랜잭션이 완료될 때까지 Shared Lock을 걸어, 수정이 불가능하게 함
- MYSQL의 경우 innoDB가 lock을 걸지 않고, SELECT 당시의 **snapshot을 만들어 undo영역을 생성**하여 동일한 데이터가 조회 될 수 있도록 함
- MYSQL 기본
- **Phantom Read** 발생  
    -  한 트랜잭션 내에서 동일한 쿼리를 두 번 실행 했을 경우, **첫 번째 쿼리에서는 없던 데이터가 다른 트랜잭션에 의해 INSERT되어 두 번째 쿼리에 나타나는 현상**

### SERIALIZABLE

- 트랜잭션이 완료될 때까지 Shared Lock, **수정 및 입력 불가능**하게 한다
- 활용도가 낮고 사용하는 곳이 적음

## 트랜잭션 전파 (Propagation)

스프링 `@Transactional` 기준, 트랜잭션의 생성 혹은 참여하는 전파 옵션

- **REQUIRED**  
    Default 옵션, 부모 트랜잭션이 있으면 부모 트랜잭션에 참여한다. 없으면 신규 트랜잭션 생성한다.  

- **SUPPORTS**  
    부모 트랜잭션이 있으면 참여, 없으면 트랜잭션 없이 진행한다.  

- **REQUIRES_NEW**  
    부모 트랜잭션을 무시하고, 항상 새로운 트랜잭션을 생성. 부모 트랜잭션이 진행 중일 경우 보류 상태로 만든다.  

- **MANDATORY**  
    트랜잭션이 있으면 참여하고, 없으면 예외를 발생시킨다. 독립적으로 진행되면 안되는 Case일때 적용한다.  

- **NOT_SUPPORTED**  
    트랜잭션이 없이 진행한다. 부모 트랜잭션이 진행 중일 경우 보류 상태로 만든다.  

- **NEVER**  
    트랜잭션이 없이 진행한다. 부모 트랜잭션이 존재할 경우 예외를 발생시킨다.  

- **NESTED**  
    진행 중인 트랜잭션이 있으면 중첩해서, 트랜잭션 안에 트랜잭션을 만든다. 중첩된 트랜잭션내에서 자식 트랜잭션은 부모에게 영향을 주지 못한다.  

### References

[https://nesoy.github.io/articles/2019-05/Database-Transaction-isolation](https://nesoy.github.io/articles/2019-05/Database-Transaction-isolation)  
[http://www.dbguide.net/db.db?cmd=view&boardUid=148216&boardConfigUid=9&boardIdx=138&boardStep=1](http://www.dbguide.net/db.db?cmd=view&boardUid=148216&boardConfigUid=9&boardIdx=138&boardStep=1)  
[https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Propagation.html](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Propagation.html)