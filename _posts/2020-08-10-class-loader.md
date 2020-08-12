---
title: "Java ClassLoader 알아보기"
date: 2020-08-10 22:35:00 +0900
category: study
lastmod : 2020-08-12 22:35:00 +0900
sitemap :
  changefreq : weekly
  priority : 1.0
---

<br>

## 1. Class Loader 란

클래스 로더는 JRE(Java Runtime Environment)에 포함되며, **런타임 중 동적으로 자바 클래스들을 JVM에 로딩하는 역할**을 담당한다.  
  
클래스로더 덕분에 JVM은 자바 프로그램 실행 중 어떤 파일이 있어야 하는지 알지 않아도 된다.  
  
또한 자바 클래스는 한번에 메모리에 올라가지 않고, 클래스로더에 의해서 어플리케이션에서 **필요 할 때만 메모리에 올라가게 된다.**

---

## 2. 클래스 로더의 종류

### + Bootstrap ClassLoader

자바 클래스들은 `java.lang.ClassLoader` 인스턴스에 의해 로딩 된다. 그리고 클래스로더의 클래스들을 로딩하는 것이 **Bootstrap ClassLoader(aka Primordial ClassLoader)**이다.  
  
Bootstrap 클래스 로더는 `jre/lib/rt.jar`와 `$JAVA_HOME/jre/lib`에 있는 **코어 라이블러리들을 포함한 JDK의 내부 클래스들을 로딩**한다.  
  
**Bootstrap ClassLoader는 다른 ClassLoader 인스턴스의 부모** 역할을 한다. Bootstrap ClassLoader는 JVM 코어 중 하나며, **native 코드**로 작성되어있기 때문에 **실행환경(플랫폼)에 따라 다르게 구현되어야 한다.**  

> **Java 8**  
Bootstrap 클래스 로더는 `jre/lib/rt.jar`와 `$JAVA_HOME/jre/lib`에 있는 코어 라이블러리들을 포함한 JDK의 내부 클래스들을 로딩한다.
  
> **Java 9**  
`rt.jar` 가 사라지면서 안에 있던 내용들이 모듈화 되어 `jre/lib`폴더 안에 저장된다. Bootstrap ClassLoader가 로딩할 수 있던 클래스의 범위가 줄어들었다.

### + Platform ClassLoader (Extension ClassLoader in Java 8)

**Bootstrap ClassLoader의 하위 클래스로더**로 기본 자바 코어 클래스들의 **extension 클래스들을 로드**한다. JDK extension 디렉토리에서 클래스들을 로드한다.  
  
> **Java 8**  
`jre/lib/ext` 폴더나 `java.ext.dirs` 환경 변수로 지정된 폴더에 있는 클래스 파일 로딩한다.
  
> **Java 9**  
이름이 Platform ClassLoader로 변경되었으며, 더이상 `java.ext.dirs` 와 `lib/ext`를 지원 하지 않는다. Extension 클래스들을 사용하길 원한다면 class path에 JAR파일들을 놔야한다. `URLClassLoader`를 사용하지 않고, `BuiltinClassLoader`를 상속, 내부 static 클래스로 구현된다.

### + System ClassLoader (Application ClassLoader in Java 8)

**어플리케이션 레벨에 있는 모든 클래스들을 JVM에 로딩** 한다. 즉, 사용자가 작성한 코드들을 로딩한다. Plaform ClassLoader의 하위으로, classpath 환경 변수에 있는 파일들, `-classpath(or -cp)`에 있는 파일들을 로드한다.
  
> **Java 9**  
이름이 System ClassLoader로 변경.  `URLClassLoader`를 사용하지 않고, `BuiltinClassLoader`를 상속, 내부 static 클래스로 구현된다.

---

## 3. 클래스로더의 작동 방법

### + Delegation Model

**ClassLoader 인스턴스는 클래스 혹은 리소스를 찾는 것을 상위 클래스로더에 위임한다.**  
  
예를 들어, JVM에 application class를 로딩해달라는 요청이 생겼을 때, System ClassLoader는 상위 클래스로더인 Platform classloader에 클래스 로딩을 위임하고, extension classloader는 bootstrap classloader에 위임한다.  
  
Bootstrap classloader에서 클래스 로딩에 실패하고, extension classloader에서도 실패 했을 때, System Classloader가 로딩을 시도 할 수 있다. (마지막까지 클래스 로딩에 실패 했을 경우 `ClassNotFoundException`이 발생)

### + Unique Classes

Delegation Model의 결과로, 상위 클래스로더에서 클래스를 찾지 못했을 때만, 하위 클래스로더가 클래스 로딩을 시도 할 수 있다. 즉, **하위 클래스로더는 상위 클래스로더가 로딩한 클래스를 다시 로딩 하지 않기 때문에 로딩된 클래스의 유일성을 보장한다.**

### + Visibility

하위 클래스로더들은 상위 클래스로더가 로드한 클래스들을 볼 수 있다. 하지만 상위 클래스로더는 하위 클래스로더가 로드한 클래스들을 볼 수 없다.

---

### References

[https://homoefficio.github.io/2018/10/13/Java-%ED%81%B4%EB%9E%98%EC%8A%A4%EB%A1%9C%EB%8D%94-%ED%9B%91%EC%96%B4%EB%B3%B4%EA%B8%B0/](https://homoefficio.github.io/2018/10/13/Java-%ED%81%B4%EB%9E%98%EC%8A%A4%EB%A1%9C%EB%8D%94-%ED%9B%91%EC%96%B4%EB%B3%B4%EA%B8%B0/)  
[https://www.baeldung.com/java-classloaders](https://www.baeldung.com/java-classloaders)  
[https://docs.oracle.com/javase/9/migrate/toc.htm#JSMIG-GUID-A868D0B9-026F-4D46-B979-901834343F9E](https://docs.oracle.com/javase/9/migrate/toc.htm#JSMIG-GUID-A868D0B9-026F-4D46-B979-901834343F9E)  