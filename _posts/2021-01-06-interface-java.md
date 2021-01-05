---
title: "내가 보려고 만든 인터페이스 in Java"
date: 2021-01-05 23:10:00 +0900
category: java
lastmod : 2021-01-05 23:10:00 +0900
sitemap :
  changefreq : weekly
  priority : 1.0
---

<br>

## 인터페이스(interface)란

---

메소드들과 상수를 가지고 있는 **추상 타입**으로, 자바의 주요 개념인 **추상화, 다형성, 다중 상속**에 사용된다.  

**추상화란**  
필요한 정보만을 사용자에게 보여주고, 연관되지 않는 불필요한 과정은 나타내지 않는 것.  
  
**다형성이란**  
하나의 객체가 여러가지 타입으로 선언 할 수 있게 하는것, 실행 중에 다른 형태를 가질 수 있게 하는 것이다.  
  
우리는 `implements` 키워드를 통해 인터페이스를 구현 할 수 있다.

<br>

## 인터페이스의 특징

---

- 인터페이스에는 **상수, 추상 매소드, 정적 메소드, default 메소드**만 허용된다.
- 인터페이스는 **직접 인스턴스화 할 수 없다**.
- 인터페이스는 하나의 메소드도 존재하지 않은 비어있는 채로 존재할 수 있다.
- **final**은 **인터페이스 정의**에 사용 할 수 없다. 컴파일 에러 발생.
- 모든 인터페이스 **선언**은 **public 혹은 default** 로 접근 지정자로 설정해야한다. 추상 modifier는 컴파일러로 인해 자동으로 추가된다
- 인터페이스 **메소드**는 **private, protected, final**로 선언 할 수 없다
- 인터페이스 **변수**는 **public, static, final**이 사용 가능하다

<br>

## 왜 인터페이스를 사용하는 가

---

### Behavioral Functionality

관련이 없는 클래스간에서 특정 **Behavioral Functionality(행위적 상관성)**을 인터페이스를 통해 **추가** 할 수 있다.  
  
예를 들어, Comparable, Comparator, Cloneable은 자바의 인터페이스로 **unrelated class**(서로 상속관계에 없는 클래스들)들에 의해 implements 된다.  

```java
public class Student

public class StudentGradeComparator implements Comparator<Student>
```

이렇게 `Student`와 `StudentGradeComparator` 두 개의 unrelated class들은 `Comparator`를 통해 비교를 하는 행위의 상관성을 갖게된다.  

<br>

### Multiple Inheritances

자바는 단일 상속만을 지원한다, 하지만 인터페이스를 사용하면 **다중의 인터페이스들로 상속하여 구현**이 가능하다.  

<br>

### Polymorphism

다형성이란 객체가 **실행 중에 다른 형태를 가질 수 있게 하는 것**이다.  
  
다형성의 종류로는 **Complie Time Polymorphism**과 **Runtime Polymorphism**이 존재한다.  
  
- `Complie Time Polymorphism (or Static polymorphsim) = Method Overloading`  
- `Runtime Polymorphism (or Dynamic polymorphism) = Method Overriding`  
  
인터페이스의 다형성은 **Method Overriding 형태의 런타임 중 객체가 다른 형태**를 가지게 한다.  

<br>

## 그 외

---

### default method in interface

자바 7의 인터페이스에서는 **Backward compatibility**를 지원하지 않는다.  
  
자바 7 혹은 이전 버전으로 쓰여저 있는 레거시 코드에서는, 존재하는 인터페이스에 추상 메소드를 추가하려면 구현된 모든 클래스에 새로운 추상 매소드를 오버라이드해야 한다.  
  
하지만 자바 8에서는 **default method를 통해 인터페이스 레벨**에서 구현 할수 있다.  
  
<br>

## 구현 예시

---

```java
public interface Card {
    // 결제 한도
    public int ONCE_MAX_LIMIT = 10000;

    // 결제
    void pay();     
    
    // 결제 후 알림
    default void notify (String phoneNumber){
        System.out.println("결제되었습니다: " + phoneNumber);
    }
}
```

`Card`의 기능들을 정의한 `Card` 인터페이스가 있다.  
  
- 모든 카드의 한도는 고정이기에, 인터페이스에서 상수로 선언, 모든 카드에서 동일하게 적용 되도록 한다.  
- 결제는 각 카드사마다 기능이 다르게 들어가기 때문에 오버라이드 할 수 있게 정의한다.  
- 결제 이후 알림은 전화번호로 가는 것이 원칙이며 default으로 정의된다.  

<br>

```java
public class ShinhanCard implements Card {
    
    // 결제
    @Override
    public void pay(){
        // 신한만의 결제 로직 구현
        notify("010");
    }     
}
```

`Card` 인터페이스를 implement 하는 `ShinhanCard` 클래스를 구현한다.  
  
- 오버라이드된 pay() 메소드는 각 카드사의 로직별로 구현되고, 결제 후 알림은 default를 따라가게 된다.  
  
<br>

## Spring에서의 인터페이스 활용

---

클래스를 인터페이스로 정의해두면 **Dependency Injection**에 도움이 된다.  

<br>

예를 들어, `Card` interface를 implements하는 `ShinhanCard`와 `SamsungCard` Bean이 있다.  

```java
public interface Card {};
public class ShinhanCard implements Card {};
public class SamsungCard implements Card {};
```

그리고 Card를 사용하는 `Payment` Bean이 있다.  

```java
public class Payment {
    private Card card;

    public Payment(Card card){
        this.card = card;
    }

    public makePayment(){
        this.card.pay();
    }
}
```

여기서 `Card`는 Spring의 **IoC에 의해 injection**이 이뤄질 것이다.  

<br>

만약 모든 결제가 `ShinhanCard`로 이뤄질 때에는  

```xml
<bean id="payment" class="Payment">
    <constructor-arg ref="card" />
</bean>

<bean id="card" class="ShinhanCard" />
```

반대로 `SamsungCard`로 이뤄질 때에는  

```xml
<bean id="card" class="SamsungCard" />
```

<br>

이렇게 `Card` 인터페이스를 활용하여 `Payment`의 **코드를 변경하지 않고,**  
**Spring Config 파일을을 바꾸는 것**만으로 **다른 bean으로 injection이 가능**하게 된다.  

<br>

### References

---

[https://www.baeldung.com/java-interfaces](https://www.baeldung.com/java-interfaces)  
[https://stackoverflow.com/questions/256255/spring-and-interfaces](https://stackoverflow.com/questions/256255/spring-and-interfaces)  
[https://limkydev.tistory.com/197](https://limkydev.tistory.com/197)  