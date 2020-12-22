# BigDecimal

1. BigDecimal은 값을 어떻게 저장하는 가

BigDecimal은 BigInteger와 percision and scale로 값을 저장한다.
percision은 해당 숫자의 총 자리 수를 나타내며, scale은 소수점 이후의 자리 수를 나타낸다.
즉, 123.456이라는 BigDecimal이 있다면, percision은 6, scale은 3이 된다.

String, Char 배열, int, long, double, BigInteger를 BigDecimal로 생성할 수 있는데,
여기서 double을 BigDecimal로 만들 때 주의 해야한다.

```java
public void doubleTest(){
    double d = 0.1;
    BigDecimal bdc = new BigDecimal(d);
    System.out.println(bdc.toString());
}
```

해당 코드의 출력 값은 다음과 같은데
사진
`double d = 0.1`로 0.1이라는 값을 선언한 것과는 다르게 BigDecimal에는 더 많은 값들이 들어간 것을 볼 수 있다.
이는 double 자체가 0.1를 완벽하게 표현하는 것이 아닌 0.1의 근사치의 값을 가지고 있기 때문이다.
double은 정확한 지수 표현이 아니며, 언제나 근사치를 표현한다. 이는 정확한 계산에 BigDecimal를 써야하는 이유이기도 하다.

따라서 double을 BigDecimal로 만들 때는 다음과 같이 선언해야 하는 것을 기억해야 한다.

```java
        double d = 0.1;
        BigDecimal bdc1 = BigDecimal.valueOf(d);
        BigDecimal bdc2 = new BigDecimal(String.valueOf(d));
```

2. BigDecimal 지원 기능

BigDecimal은 arithmetic, scale manipulation, rounding, comparison, hashing, and format conversion을 지원한다.


Rounding
BigDecimal은 rounding에 대해 사용자가 설정하게 되어있다.
죽, Rounding에 대한 별도 설정 없이 rounding이 필요한 arithmetic, scale manipulation을 했을 경우 예외가 발생한다.

```java
    BigDecimal bdc = new BigDecimal("0.33333");
    bdc = bdc.setScale(2); // 예외 발생, scale(5) -> scale(2), Rounding 설정 없이, Rounding이 필요한 Scale manipulation 실행

    BigDecimal a = new BigDecimal("1");
    BigDecimal b = new BigDecimal("3");
    BigDecinal c = a.divide(b); // 예외 발생, 0.3333..., Rouding이 필요한 arithmetic 실행 
```

BigDecimal이 지원 하는 Round Option은 총 8개로 MathContextObejct를 전달 함으로 rouding 옵션을 정할 수 있다
ROUND_UP - 올림
ROUND_DOWN - 내림
ROUND_CEILING - celling
ROUND_FLOOR - flooring
ROUND_HALF_UP - 반올림
ROUND_HALF_DOWN - 반내림
ROUND_HALF_EVEN - 짝수일 경우 올림
OUND_UNNECESSARY - round 불 필요, exception 발생

arithmetic
BigDecimal은 add, subtract, multiply, divide의 네가지 계산 기능을 제공한다.

계산 후 결과 값의 Scale은 달라지는 며, 각 operation의 기본적인 Scale 계산 법은 다음과 같다.
|Operation|Scale Calculation|
|Add|max(addend.scale(), augend.scale())|
|Subtract|max(minuend.scale(), subtrahend.scale())|
|Multiply|multiplier.scale() + multiplicand.scale()|
|Divide|dividend.scale() - divisor.scale()|

여기서 Divide의 경우, 계산에 따라 Scale 값이 크게 달라질 수 있다.
따라서 Divide 연산을 할 때는 scale과 rounding을 설정하는 것이 좋다.

toString
BigDecimal의 toString()은 canonical representation을 해준다.
이말인 즉슨, `0E-8` 같은 지수표현식이 toString을 통해 나올 수 있다.
BigDecimal의 toString()를 보면 `0.0000003` 처럼 상수없이 0으로 이어지는 지수 중에 0이 연속적으로 6개 이상 이어질경우
`E-notation`형태로 반환한다.
만약 `E-notation`형태의 String을 원하지 않는다면, `toPlainString`을 통해 그대로의 스트링을 사용하기를 바란다.

음수일때 BigDecimal은 unscaled value 를 10의 - 제곱근으로 곱한다

### BigInteger
BigInteger는 integer 값을 sign 값을 정하는 signum
그리고 값을 저장하는 int[]인 mag로 저장한다.
여기서 mag는 big endian표현으로 integer의 값을 저장하는데
unsigned integer의 max 값인 4294967295를 넘어가면 배열에 크기가 하나 증가한다.

`BigInteger a8 = new BigInteger("4294967295");`를 디버그 해보면
mag의 길이는 1, mag[0] = -1 인것을 볼 수 있는데, 
-1을 binary로 바꾸면 `1111 1111 1111 1111 1111 1111 1111 1111` unsigned integer의 max 값이 나온다.

여기서 unsigned integer의 max + 1인 4294967296을 BigInteger에 넣으면
`BigInteger a8 = new BigInteger("4294967296");`
mag[0] = 1, mag[1] = 0이 들어간 것을 알 수 있다.
이걸 또 Binary로 바꾸면 `0000 0000 0000 0000 0000 0000 0000 0001 0000 0000 0000 0000 0000 0000 0000 0000`이며
즉, 4294967296를 의미한다.


Reference
https://docs.oracle.com/javase/8/docs/api/java/math/BigDecimal.html
https://www.baeldung.com/java-bigdecimal-biginteger