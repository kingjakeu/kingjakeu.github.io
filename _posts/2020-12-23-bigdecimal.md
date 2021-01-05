---
title: "BigDecimal 기본 정리, BigDecimal 활용 in Java"
date: 2020-12-23 13:35:00 +0900
category: java
lastmod : 2020-12-23 13:35:00 +0900
sitemap :
  changefreq : weekly
  priority : 1.0
---

<br>

## BigDecimal을 언제, 왜 사용하는가

_____

**거대하고 정확한 실수 값**을 표현 할 때, 우리는 **BigDecimal**이 필요하다.  
  
Java에서는 실수 표현을 위해, 기본 자료형으로 `double` 제공한다. 하지만 `double`은 실수의 값을 정확하게 표현하는 것이 아닌 [부동 소수점 표현](https://en.wikipedia.org/wiki/Floating-point_arithmetic)에 의해 **근사치의 값**을 가지고 있기 때문이다. 예를들어, 돈과 관련된 금액 계산을 구현해야 한다면, **BigDecimal은 필수**다.  

또한 `double`의 범위는 *(1.7976931348623157 x 10308, 4.9406564584124654 x 10-324)*로 **범위 이상의 값**을 표현 할 때도 `BigDecimal`이 필요하다.  
<br>

## BigDecimal은 값을 어떻게 저장할까

_____

BigDecimal은 `intVal(BigInteger), percision(int), scale(int), intCompact(int)`로 값을 구성한다.  

`percision`은 해당 숫자의 총 자리 수를 나타내며, `scale`은 소수점 이후의 자리 수를 나타낸다.  
`intVal`은 정수로 표현된 `BigDecimal`의 값을 `BigInteger`에 저장한다. (ex. 123.45 -> 12345)  

만약 `BigDecimal`의 길이("." 포함)가 **18자리 이하**일 땐, `intVal`에 값을 따로 저장하지 않고 `intCompact`에 **정수 값을 저장**한다.

``` java
    // persion 6, scale 2, intVal = null, intCompact = 12345
    BigDecimal b1 = new BigDecimal("123.45"); 

    // persion 18, scale 0, intVal = null, intCompact = 123451234512345123
    BigDecimal b2 = new BigDecimal("123451234512345123"); 

    // persion 18, scale 8, intVal = 123451234512345123, intCompact = 123451234512345123
    BigDecimal b3 = new BigDecimal("1234512345.12345123"); 
```

<br>

## BigDecimal 생성

_____

`String, char[], int, long, double, BigInteger`를 **BigDecimal로 생성**할 수 있다.

``` java
    int i = 123;
    BigDecimal id = new BigDecimal(i);

    long l = 12345;
    BigDecimal ld = new BigDecimal(l);

    double d = 123.45;
    BigDecimal dd = new BigDecimal(d); // 이건 쓰면 안됌

    String s = "1234";
    BigDecimal sd = new BigDecimal(s);

    char[] c = {'1', '2', '3'};
    BigDecimal cd = new BigDecimal(c);
```

여기서 **double을 BigDecimal로** 만들 때 **주의** 해야한다.

```java
public void doubleTest(){
    double d = 0.1;
    BigDecimal bdc = new BigDecimal(d);
    System.out.println(bdc.toString());
}
```

해당 코드의 출력 값은 다음과 같다.  
![double-to-decimal](https://drive.google.com/uc?id=1DSfoVcTFokJOpYqeYNvfOFYZDYOfI1ol)

`double d = 0.1`로 0.1이라는 값을 선언한 것과는 다르게 `BigDecimal`에는 **더 많은 값들**이 들어간 것을 볼 수 있다. 이는 `double` 자체가 0.1를 완벽하게 표현하는 것이 아닌 부동 소수점 표현에 의해 **0.1의 근사치의 값**을 가지고 있기 때문이다.  

`double`은 정확한 지수 표현이 아니며, **언제나 근사치**를 표현한다. 이는 정확한 계산에 **BigDecimal를 써야하는 이유**이기도 하다.  

따라서 `double`을 `BigDecimal`로 만들 때는 **다음과 같이 선언**해야 하는 것을 기억해야 한다.  

``` java
    double d = 0.1;
    BigDecimal db1 = BigDecimal.valueOf(d);
    BigDecimal db2 = new BigDecimal(String.valueOf(d));
```

<br>

## BigDecimal 지원 기능

_____

`BigDecimal`은 **arithmetic(연산), scale manipulation(범위 조정), rounding(반올림), comparison(비교), hashing(해싱), and format conversion(포멧 변경)**을 지원한다.  

<br>

### Rounding (반올림/반내림/절삭)
_____

`BigDecimal`은 rounding에 대해 **사용자가 설정**하게 되어있다.  

즉, Rounding에 대한 별도 설정 없이 rounding이 필요한 **arithmetic(연산)이나 scale manipulation(범위 조정)**을 했을 경우 **예외가 발생**한다.  

```java
    BigDecimal bdc = new BigDecimal("0.33333");
    bdc = bdc.setScale(2); // 예외 발생, scale(5) -> scale(2), Rounding 설정 없이, Rounding이 필요한 Scale manipulation 실행

    BigDecimal a = new BigDecimal("1");
    BigDecimal b = new BigDecimal("3");
    BigDecinal c = a.divide(b); // 예외 발생, 0.3333..., Rouding이 필요한 arithmetic 실행 
```

### BigDecimal이 지원 하는 Round Option

|ROUND_UP|올림 (0으로 부터)|
|ROUND_DOWN|내림 (0으로 부터)|
|ROUND_CEILING|올림 (무한대 양수를 향해)|
|ROUND_FLOOR|내림 (무한대 음수를 향해)|
|ROUND_HALF_UP|반올림 (버려지는 수가 0.5 이상이면 올림)|
|ROUND_HALF_DOWN|반내림 (버려지는 수가 0.5 이하이면 버림, 0.5 초과면 올림)|
|ROUND_HALF_EVEN|짝수일 경우 내림 (버려지는 수 옆의 수가 짝수이면 ROUND_HALF_DOWN, 홀수이면 ROUND_HALF_UP)|
|ROUND_UNNECESSARY|rounding 미 설정, rounding 필요시 exception 발생 (`BigDecimal` 생성시 기본값)|

<br>

### Arithmetic (연산)

_____

`BigDecimal`은 **add(덧셈), subtract(뺄셈), multiply(곱샘), divide(나눗셈)**의 네가지 계산 기능을 제공한다. 결과 값의 scale은 계산 후 달라진다.  

각 operation의 **기본적인 Scale 계산 법**은 다음과 같다.

|Operation|Scale Calculation|
|-----|-----|
|Add|max(addend.scale(), augend.scale())|
|Subtract|max(minuend.scale(), subtrahend.scale())|
|Multiply|multiplier.scale() + multiplicand.scale()|
|Divide|dividend.scale() - divisor.scale()|

여기서 Divide의 경우, 계산에 따라 **scale 값이 크게 달라질 수 있기 때문에,** divide 연산을 할 때는 **scale과 rounding을 설정**하는 것이 좋다.  
  
또한, `BigDecimal`은 **불변 객체**이기 때문에 연산 결과는 **기존의 객체의 값을 변화시키지 않고, 새로운 객체를 return** 한다.

```java
    BigDecimal a = new BigDecimal("1.0");
    BigDecimal b = new BigDecimal("3.0");
    BigDecimal add = a.add(b);
    BigDecimal sub = a.subtract(b);
    BigDecimal mul = a.multiply(b);
    BigDecimal div = a.divide(b, 2, BigDecimal.ROUND_DOWN); // 소수 둘째자리까지, 내림
```

<br>

### Comparison(비교)

_____

`BigDecimal`은 `Comparable` interface가 implement 된 클래스이다. 따라서 `BigDecimal` 간의 비교는 `compareTo`을 통해 할 수 있다.

``` java
    BigDecimal a;
    BigDecimal b;
    a.compareTo(b); // a == b = 0, a > b = 1, a < b = 1
```

<br>

### toString

_____

`BigDecimal`의 `toString()`은 **canonical representation**을 하기 때문에,  
`0.0000003` 처럼 상수없이 0으로 이어지는 지수 중에 **0이 연속적으로 6개 이상** 이어질 경우 `E-notation`형태로 반환한다.  

즉, `0E-8` 같은 지수표현식이 toString을 통해 나올 수 있다.  

만약 `E-notation`형태의 String을 원하지 않는다면, `toPlainString()`을 통해 쌩 문자열을 사용하기를 바란다.  

``` java
    BigDecimal d = new BigDecimal("0.0000005");
    String s1 = d.toString(); // "5E-7"
    String s2 = d.toPlainString(); // "0.0000005"
```

<br>

## 번외, BigInteger가 값을 저장하는 법

______

`BigInteger`는 integer 값을 **sign(부호)** 값을 정하는 `signum`, 그리고 **정수 값을 저장**하는 `int[]`인 `mag`로 값을 저장한다.  

여기서 mag는 **big endian**표현으로 integer의 값을 저장하는데,  
**unsigned integer의 max** 값인 `4294967295`를 넘어가면 **mag 배열에 크기가 하나 증가**한다.  
  
`BigInteger a8 = new BigInteger("4294967295");`를 디버그 해보면, `mag의 길이는 1`, `mag[0] = -1` 인것을 확인 할수 있다.

이는 -1을 binary로 바꾸면 `1111 1111 1111 1111 1111 1111 1111 1111`  
-> **unsigned integer의 max** 값이 나온다.  
  
여기서 unsigned integer의 `max + 1인 4294967296`을 `BigInteger`에 넣으면  
`BigInteger a8 = new BigInteger("4294967296");` / `mag[0] = 1, mag[1] = 0`이 저장된다.  

이걸 또 Binary로 바꾸면  
`0000 0000 0000 0000 0000 0000 0000 0001 0000 0000 0000 0000 0000 0000 0000 0000`

즉, **4294967296를 의미한다.**  

<br>

### Reference

_____

[https://docs.oracle.com/javase/8/docs/api/java/math/BigDecimal.html](https://docs.oracle.com/javase/8/docs/api/java/math/BigDecimal.html)  
[https://www.baeldung.com/java-bigdecimal-biginteger](https://www.baeldung.com/java-bigdecimal-biginteger)