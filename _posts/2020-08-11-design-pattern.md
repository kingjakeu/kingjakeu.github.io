---
title: "디자인 패턴(정리중)"
date: 2020-08-11 19:35:00 +0900
category: study
lastmod : 2020-08-11 19:35:00 +0900
sitemap :
  changefreq : weekly
  priority : 1.0
---



### Strategy Pattern

+ 객체를이 실행 할 수 있는 **행위에 대해 Strategy 클래스를 생성**하고, 유사한 행위들을 캡슐화한다.  
+ **행위를 Strategy 클래스로 캡슐화**하여 객체의 행위를 **동적으로 바꿀 수 있게** 한다.  
+ 따라서 객체의 행위를 **직접적으로 수정하지 않고 Strategy만 바꿔 행위 자체를 유연하게 확장**한다. (행위 패턴)

> ex. 결제(행위)를 가진 가게(클래스)가 있고, 가게 클래스를 상속받는 편의점과 음식점이있다.  
편의점에서는 카드로 결제를 하고, 음식점에서는 현금으로 결제를 한다. 어느 순간, 음식점에서도 카드 결제가 가능하다면 어떻게 해야할까?  
> -> 결제라는 행위를 Strategy 클래스로 생성한다. 그리고 해당 전략을 상속받는 카드결제 Strategy, 현금 결제 Strategy 만든다. 음식점과 편의점은 상황에 따라 결제 Strategy 설정하고 결제를 하면된다.

### Factory Method Pattern

+ 조건에 따라 객체를 다르게 생성해야 할 때, 객체 생성 처리를 Factory에게 위임하여 처리하는 방식이다.
+ 객체 생성의 변화에 대비하는데 유용하다.(생성 패턴)

### Facade Pattern

+ 퍼사드 패턴은 서브시스템의 일련의 인터페이스에 대한 통합된 인터페이스를 제공한다.
+ 퍼사드에서 복잡한 서브시스템에 대한 통합된 인터페이스를 제공하여, 쉽게 사용할 수 있게 한다.

> 예를 들어, 커피 기계가 있다. 카푸치노를 만들려면, 원두를 갈고, 에스프레소를 내리고, 우유거품을 만들어야 한다.  
이런 일련의 과정들을 카푸치노 내리기 기능으로 통합해서 제공하는 것이 커피 기계 퍼사드다.

### Adapter Pattern

+ 클라이언트는 Target 인터페이스만 접근한다. 
+ Adapter가 Target과 호환이 되지 않는 인터페이스를 Target에 맞게 변환 하여 제공한다.

> 예를 들어, A사의 결제 시스템과 B사의 결제 시스템이 있다. A사가 B사에 합병되어, 사용자는 이제 B사의 인터페이스로 A사의 결제시스템을 이용해야한다.  
이 경우, A사가 B사 인터페이스에 맞게 서비스를 제공하기 위해 Adapter를 만들고 A사의 시스템을 기존 B사의 시스템기능에 맞게 implement한다.  

### Template Method Pattern

+ 알고리즘의 구조를 메소드 안에 정의하고, 각 단계는 서브클래스를 따라간다. 
+ 알고리즘 구조 변경 없이, 서브클래스의 변경으로만 알고리즘을 재정의 하는 패턴이다.

> 예를 들어, 포인트 결제나 카드 결제나 결제를 하는 행위는 같다. 사용자에게 결제수단을 받고, 결제 요청을 보내고, 결제를 확정한다.  
> 이와 같은 기본 결제 알고리즘은 템플릿 메소드에 정의한다. 그리고 포인트와 카드는 이 기본 결제 템플릿을 확장하여 각자 어떻게 결제수단을 받는지, 결제 요청을 보내는 지, 내부 구현을 한다.
