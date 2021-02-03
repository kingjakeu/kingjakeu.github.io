---
title: "내가 보려고 쓰는 Singleton in Java"
date: 2021-02-03 17:00:00 +0900
category: java
lastmod : 2021-02-03 17:00:00 +0900
sitemap :
  changefreq : weekly
  priority : 1.0
---

<br>

## Singleton Pattern

---

+ 전역 변수를 사용하지 않고, **객체 인스턴스를 하나만 생성**, 생성된 객체를 **어디에서든 참조**할 수 있도록 하는 패턴.
+ 싱글톤을 구현하는 방법은 여러가지가 존재하지만 공통적인 핵심은 **private constructor** 와 **global access method**이다.
	1. private 생성자로 외부에서 객체를 생성하지 못하게 하는 것
	2. public static 매소드로 클래스의 인스턴스를 모든 곳에서 반환 받을 수 있게 하는것이다(global access method)

---

<br>

## Singleton 구현 방식

---

### Eager Initilization(이른 초기화)

싱글톤 클래스가 로딩되는 시점에 인스턴스를 생성한다. 사용되지 않는 싱글톤 클래스도 생성되기 때문에 자원 낭비를 초래한다.

``` java
public class Singleton {
    private static final Singleton instance = new Singleton();
    private Singleton(){};
    public static Singleton getInstance(){
        return instance;
    }
}
```

<br>

### Lazy Initialization(게으른 초기화)

인스턴스를 global access method에서 생성한다. 객체가 실제로 사용되는 시점에 생성되기 때문에 이른 초기화 방법보다 자원을 아낄 수 있다.  
하지만 멀티스레드 환경에서 동시에 `getInstance()`에 접근한다면, 여러개의 객체가 생성될 수 있다.  

``` java
public class Singleton {
    private static Singleton instance;
    private Singleton(){};
    public static Singleton getInstance(){
        if(instance == null) instance = new Singleton();
        return instance;
    }
}
```

<br>

### Thread Safe Singleton

Lazy Initialization 하면서 Thread-safe한 싱글톤을 만드는 가장 쉬운 방법은 global access method에 `synchronized` 키워드를 다는 것이다.  
하지만 `synchronized`는 접근 하는 모든 메소드에 lock을 걸기 때문에 프로그램의 성능을 저하 시킨다.

``` java
public class Singleton{
    private static Singleton instance;
    private Singleton(){};
    public static synchronized Singleton getInstance(){
        if(instance == null) instance = new Singleton();
        return instance;
    }
}
```

<br>

### Lazy initialization with Double check locking

`synchronized`는 프로그램의 오버헤드를 발생시키기 때문에,  
기존의 `getInstance()`애 `synchronized`를 제거하고 인스턴스를 생성 할 때만 시켜, 오버해드를 최소한으로 줄이는 방식  

``` java
public class Singleton{
    private static Singleton instance;
    private Singleton(){};
    public static Singleton getInstance(){
        if(instance == null) {
            synchronized (Singleton.class){
                instance = new Singleton();
            }
        } 
        return instance;
    }
}
```

<br>

### Bill Pugh Singleton Implementation

Inner static helper class를 사용하는 방식으로, helper class는 싱글톤 클래스의 global access method가 처음 호출될 때 클래스가 로드된다.  
로드된 helper 클래스는 싱글톤 클래스의 인스턴트를 만든다.  

```java
public class Singleton {
    private static Singleton instance;
    private Singleton() {};
    private static class SingletonHelper {
        private static final Singleton INSTANCE = new Singleton();
    }
    public static Singleton getInstance(){
        return SingletonHelper.INSTANCE;
    }
}
```

<br>

### Reflection을 통한 Singleton의 문제

Bill Pugh Singleton 방식이 최적의 구현 방식이지만, Reflection을 통해 Singleton 상태를 망가뜨릴 수 있다.

``` java
    Singleton instance1 = Singleton.getInstance();
    Singleton instance2 = null;
    try {
        Constructor[] constructors = Singleton.calss.getDeclaredConstructors();
        for(Constructor constructor : constructors) {
            constructor.setAccessible(true); // private 생성자를 접근 가능
            instance2 = (Singleton) constructor.newInstance(); // 새로운 객체 생성
            break;
        }
    } catch(Exception e){

    }
```

<br>

### Enum Initialization

Enum을 활용한 Singleton 구현은 Lazy Initialization을 할 수 없다.  
하지만 직렬화/역직렬화 구현을 고민할 필요 없고, Reflection으로 부터 안전하게 Singleton을 유지 할 수 있다.  
또한 애초부터 Thread-safe하기 때문에 Multi-Thread 환경에서 안전하다.  

``` java
public enum Singleton {
    INSTANCE;
}
```

---

<br>
