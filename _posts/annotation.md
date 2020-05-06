
Map으로 받아 온 데이터를 도메인 모델로 변환 시켜줘야 하는 일이 생겼다.
하나씩 map.get으로 정보를 꺼내오기에는 변환 시킬 때마다 map의 key를 명시해야 했다.
관리와 효율성 측면에서 너무 구렸다. 모델로 변환 시켜주는 매퍼 유틸을 만들고
유틸에서 Annotation으로 지정해준 키값을 가져올 수 있게 작업을 시작했다

Custom Annotation 생성 방법
Custom Annotation은 `@interface` 선언해서 만들 수 있다.
``` java
public @interface MyAnnotation {
}
```

이렇게 Custom Anntation을 만든 후에는

`@Retention`을 통해 컴파일러와 런타임에 해당 Annotation이 유지(retain)되어야 하는지 명시한다.
`@Retention(RetentionPolicy.SOURCE)` : 컴파일러와 런타임 둘 다 해당 Annotation을 retain하지 않음
`@Retention(RetentionPolicy.CLASS)` : 컴파일러만 retain, 런타임에는 버려짐
`@Retention(RetentionPolicy.RUNTIME)` : 컴파일러와 런타임 둘 다 해당 Annotation을 retain

``` java
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation {
}
```
컴파일과 런타임 둘 다 필요 했기에, `@Retention(RetentionPolicy.RUNTIME)`로 구현했다.

`@Target`을 통해 Annotation의 Element Type을 지정 해줄 수 있다. Element Type으로는 총 [8가지](https://docs.oracle.com/javase/tutorial/java/annotations/predefined.html) 이지만,
가장 필요 했던 Field level은 `@Target(ElementType.FIELD)`선언 할수 있고
그 외 Class level은 `@Target(ElementType.TYPE)` 
Method level은 `@Target(ElementType.METHOD)`로 선언 할 수 있다.
``` java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface MyAnnotation {
}
```

Annotation은 Method를 Member로 가져 갈 수 있다.
흔히 보는 `@SomeAnnotation(key = "jake")` 같은 형식의 Annotation에서 key가 Annoatation의 Member라 할 수 있다.

내가 필요했던 기능은 Map에 들어있는 값을 객체의 필드와 매핑하는 것이기에
Map의 Key값, 그리고 해당 Value의 Type을 지정했다.

``` java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface MyAnnotation {
    String key();

}
```
주의할 점 
Annotation의 Member의 returnType으로는 
A primitive type (int, short, long, byte, char, double, float, or boolean)
String
Class (with an optional type parameter such as Class<? extends MyClass>)
An enum type
An annotation type
An array of the preceding types
으로 지정 해줘야 한다. (List 이런건 안됌)

또한 Parameter가 있는 Method로 선언 할 수 없고, exception도 throw 할 수 없다.
default로 해당 Member method의 기본 return 값을 지정 할 수 있다.

Custom Annotation 적용 예시
