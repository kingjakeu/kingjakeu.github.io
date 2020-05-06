---
title: "Mybatis ResultMap 활용"
date: 2020-03-16 22:45:00 -0400
category: springboot
---

Mybatis 사용 중  
1. VO 도메인 객체의 변수 이름이 데이터베이스의 테이블 필드명과 다르거나
2. VO 안에 VO가 있는 도메인 객체를 사용할 때
ResultMap을 활용하여 보다 간편하게 DB에 데이터를 가져오거나 작성 할 수 있다.

일단 결제 정보를 담는 이런 도메인 객체가 있을떄
|PAY_NO||
|USER_ID||
|USER_NAME|
|MCNT_ID|
|MCNT_NAME|

```java
@Getter
@Setter
public class PaymentInfo{
    private String paymentNo;
    private UserInfo userInfo;
    private MerchantInfo merchantInfo;
}
```

```java
@Getter
@Setter
public class UserInfo{
    private String userId;
    private String userName;
}
```

```java
@Getter
@Setter
public class MerchantInfo{
    private String merchantId;
    private String merchantName;
}
```

일반적인 `resultType = com.example.PaymentInfo`으로는 데이터가 매핑이 되지 않는다.

데이터베이스 컬럼명도 다를 뿐더러, VO안에 VO가 있기 때문에 일반적인 매핑으로는 가능하지 않다는 걸 알 수 있다.

이럴떄 ResultMap을 사용 하여 내가 원하는 객체의 프로퍼티를 원하는 테이블에 필드로 지정하여 사용하면 간편하게 해결된다.
```xml
    <resultMap id = "PaymentInfo" type ="com.example.PaymentInfo">
        <id property="paymentNo" column="PAY_NO"/>
        <association property="UserInfo" javaType="com.example.UserInfo">
            <result property="userId" column="USER_ID"/>
            <result property="userName" column="USER_NAME"/>
        </association>
        <association property="MerchantInfo" javaType="com.example.MerchantInfo">
            <result property="merchantId" column="MCNT_ID"/>
            <result property="merchantName" column="MCNT_NAME"/>
        </association>
    </resultMap>
```
이렇게 ResultMap을 지정해주고
```xml
    <select id="selectPaymentInfoByPaymentNo" resultMap="PaymentInfo">
        SELECT  PAY_NO,
                USER_ID,
                USER_NAME,
                MCNT_ID,
                MCNT_NAME
        FROM    PAY_INFO
        WHERE   PAY_NO = #{paymentNo}
    </select>
```
이렇게 SELECT를 하면 resultMap에서 지정한 대로 매핑이 되어 있는 것을 확인 할 수 있다.

출처 https://mybatis.org/mybatis-3/ko/sqlmap-xml.html#Result_Maps