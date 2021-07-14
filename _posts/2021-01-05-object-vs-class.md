---
title: "객체(Object)와 클래스(Class)의 차이"
date: 2021-01-05 23:10:00 +0900
category: java
lastmod : 2021-01-05 23:10:00 +0900
sitemap :
  changefreq : weekly
  priority : 1.0
---

## 객체(Object)와 클래스(Class)의 차이

||객체|클래스|
|:---|:---|:---|
|정의|**클래스의 인스턴스** (=메모리에 할당된 실체)|객체가 만들어 질 수 있는 **템플릿** 같은 존재|
|생성 가능 개수|필요에 따라 **여러번 생성** 가능|**한번만 선언(declare)** 가능|
|메모리 할당|**생성될 때** 메모리에 할당|생성될 떄 **메모리에 할당되지 않음**|
|생성 방법|new 키워드, newInstance 메소드, 팩토리 메소드, 역직렬화|class 키워드를 통해 정의|

## 클래스 / 객체 / 인스턴스 예시

```java
// 클래스 
public class Human {
    String name;
}

// 객체
Human man;
Human woman;

// 인스턴스화, 메모리 할당
man = new Human();
```

## 그 외

우리는 `.java` 파일을 생성 하면서, `class` 키워드를 통해 클래스를 미리 정의해둔다. 그리고 컴파일을 통해 `.class`파일이 생성된다.  
  
우리가 `new`를 통해 객체를 인스턴스화 할 때, 자바의 [ClassLoader](https://kingjakeu.github.io/java/2020/08/10/class-loader/)는 해당 객체에 알맞은 클래스를 찾고 해당 클래스를 메모리에 올리게 된다.  
  
또한 `interface`는 클래스가 아니다. 각 인터페이스에 해당되는 `.class`파일들이 컴파일시 생성이 되지만, 클래스라고 할 수 없다. 이는 `enum`, `annotation`도 마찬가지이다. 우리가 객체를 `interface`,`enum`, `annotation`로 인스턴스화를 할 수 없는 것을 생각하자.

### References

[https://www.javatpoint.com/difference-between-object-and-class](https://www.javatpoint.com/difference-between-object-and-class)  
[https://gmlwjd9405.github.io/2018/09/17/class-object-instance.html](https://gmlwjd9405.github.io/2018/09/17/class-object-instance.html)  
[https://stackoverflow.com/questions/11720288/is-an-interface-a-class](https://stackoverflow.com/questions/11720288/is-an-interface-a-class)  