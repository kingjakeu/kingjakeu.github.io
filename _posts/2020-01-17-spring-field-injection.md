---
title: "Field Injection is not recommended"
date: 2020-01-17 22:45:00 -0400
categories: study
---

Field Injection is not recommended
=====

[##_Image|kage@cJpSAy/btqAMlpwINA/YI9wNnFDNc9XWwp5HdC3gk/img.png|alignCenter|width="100%"|_##]
<br>
`@Autowired`를 Bean 주입을 받다 보면 warning이 뜬다. `@Autowired`를 사용한 Bean 주입 방법은 비권장방식인 **Field Inejction** 방법을 사용 해서 Bean을 주입했기 때문이다.<br><br>

Dependency Injection 관련된 [Spring 공식 문서](https://docs.spring.io/spring/docs/5.0.3.RELEASE/spring-framework-reference/core.html#beans-factory-collaborators)
에서는 두 개의 권장된 Bean 주입 방식을 알려준다.

+ [Constructor-based dependency inejction (생성자 기반 주입)](#constructor-based-dependency-inejction)
+ [Setter-based dependency injection (Setter 기반 주입)](#setter-based-dependency-injection)
<br>

### Constructor-based dependency inejction

생성자 기반 주입 방식  
생성자 기반 주입 방식에서는 **클래스 생성자**에 `@Autowired`를 달아 놓고, 생성자의 **매개변수로** 의존성을 주입을 해야 할 것들을 알려준다. 생성자 기반 주입 방식의 장점으론 주입되어야 하는 **필수 적인 Bean들**을 `final`롤 선언하여 클래스가 인스턴트화 할 때 생성 할 수 있다.  
```{java}
@Controller
public class ExampleController {

    private final ExampleBean exampleBean;

    @Autowired
    public ExampleController(ExampleBean exampleBean){
        this.exampleBean = exampleBean;
    }
    
}
```
<br>

### Setter-based dependency injection
Setter 기반 주입 방식  
Setter 기반 주입 방식에서는 setter 매소드에 `@Autowired`를 달아준다. Spring 컨테이너는 이러한 setter 매소드들을 Bean이 **no-arument constructor** 혹은 **no-argument static factory method**를 이용하여 인스턴트화 할 때 불러 실행한다.  
```{java}
@Controller
public class ExampleController {

    private ExampleBean exampleBean;

    @Autowired
    public void setExampleBean(ExampleBean exampleBean){
        this.exampleBean = exampleBean;
    }

}

```

<br><br> 

### Field 기반 의존성 주입의 단점
-----

#### 불변 필드 주입 불가
**Immutable/final(불변)** 필드는 생성자 기반 방식으로만 가능하다.

#### 단일 책임 원칙 위배 
Field 기반 의존성 주입은 SOLID(객체 지향 설계)의 원칙 중 하나인 **S : Single responsibility priciple**를 위배하기 쉽다. 필드 기반 방식은 쉽게 의존성을 필요할 때마다 추가하면 된다. 따라서 단일 책임 원칙을 위배하는지 알아채기 힘들다. 하지만 생성자 기반의 방식은 의존성 주입이 필요할때 마다 생성자의 매개변수가 늘어나게 되고, **방대해진 생성자**는 해당 코드가 잘못되가고 있다는 것을 알 수 있다.

#### 의존성 주입 컨테이너 간의 강한 결합성 문제
Field 기반 의존성 주입이 비권장적인 가장 큰 이유는 의존성 주입이 **강한 결합성(Tightly Coupled)** 을 가졌기 때문이다.  
Field 기반 방법으로 `@Autowired`된 필드를 사용 하려면 **클래스를 인스턴트화**하고 **reflection**을 이용해 주입하는 방법 밖에 없다. 아니면 해당 필드는 계속 **null**인 상태로 남고 사용할 수 없다.  
의존성 주입 디자인 패턴은 **클래스 생성 관리와 의존성 주입의 책임을 분리** 시켜놓는다. 이렇게 하면 결합성이 느슨해지며 **SOLID**를 지킬 수 있게 된다.  
따라서 Field 기반 의존성 주입을 사용하면 **개별 생성**이 불가능 하다. `@Autowired`된 필드를 사용하려면, Spring 컨터이너를 이용해 클래스를 인스턴트화를 하는 방법 밖에 없다. 이러한 생성 방법은 **unit testing 용이성**에 영향을 준다.  

#### 숨겨진 의존성
**DI(Dependency Injection) 패턴**을 사용할 때는, 의존성 책임과 관리에 **명확한 표현이 노출되어** 표기되어야 한다. 하지만 Field기반 의존성 주입은 이러한 의존성 관련 정보를 **클래스 내부에 숨겨서** 가지고 있다.
