---
title: "알면 좋은 String in Java"
date: 2020-08-12 23:35:00 +0900
category: java
lastmod : 2020-08-12 23:35:00 +0900
sitemap :
  changefreq : weekly
  priority : 1.0
---

<br>

### String

자바에서 String은 불변하다. JVM은 literal String마다 단 하나의 복사본만 String pool에 저장한다.  
  
즉, 같은 문자열을 가진 literal String을 여러개 생성하면, JVM은 그 문자열을 String pool에 저장하고 나머지는 String pool내의 문자열을 참조한다.  

---

### String Pool

String Pool은 Java7에서부터 Perm영역이 아닌 Heap영역에 위치한다.  
  
Perm영역에 위치 할 경우, 한정된 메모리 크기에 과도하게 literal String을 생성할 경우 OutOfMemeory가 발생을 한다. 또한 Garbage Collection의 도움도 받지 못한다.  
  
따라서, Java7에서부터 Heap 영역에 별도 위치에 위치하여, 가비지 컬랙션의 도움을 받아 OutOfMemory현상을 회피 할 수 있게 됐다.

String 객체의 생성자로 String을 선언 할 경우는 좀 달라진다. 생성자로 생성한 String은 String pool이 아닌 일반 객체들 처럼 일반 Heap에 관리가 된다.  
  
즉, 같은 값의 문자열이더라도 다른 객체로 인식되어 매번 생성되고, 참조하는 대상이 다르다.

---

### String compareTo

일반적인 compareTo는 A > B == 1, A < B == -1의 기능을 한다. 하지만 String에서 compareTo는 각 문자의 unicode를 비교한다.  

즉, lexicographically(사전 순서)로 결과가 나오며, 결과의 차이는 일치하지 않는 문자의 차이다.  
  
각 문자의 비교가 동일 할 경우에는 a.compareTo(b) = a.length-b.length의 값을 반환한다.

---

### String, StringBuilder, StringBuffer

String은 불변 객체이기 때문에, 한번 생성되면 이후 메모리 공간이 변하지 않는다.  
  
즉, + 연산을 통해 String을 연장 시키면, 기존 문자열 + 새로운 문자열이 아닌 둘을 합친 새로운 문자열이 생성되는 것이다.

#### StringBuilder & StringBuffer

Builder와 Buffer 둘 다 새로운 문자열을 연산 할 때, String과 다르게 기존 객체에 연장을 시키는 형태로, 연산시 기존 객체의 공간이 부족하다면 버퍼크기를 늘려 유연하게 동작한다.  
  
차이점은 Builder는 Thread-safe하지 않고, Buffer는 safe하다.
