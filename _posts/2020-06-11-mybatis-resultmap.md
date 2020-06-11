---
title: "Mybatis ResultMap 활용"
date: 2020-06-11 22:45:00 +0900
category: springboot
lastmod : 2020-06-11 22:45:00 +0900
sitemap :
  changefreq : weekly
  priority : 1.0
---

Mybatis 사용 중, **ResultMap**을 활용하여 보다 간편하게 DB에 데이터를 객체로 가져올 수 있다.  
ResultMap은 **VO안에 VO가 있는 경우, 즉 하나의 객체가 여러 객체로 복잡하게 구성**되어 있을 떄, 효과적으로 사용할 수 있다.  

예를 들어, 결제 정보를 저장하는 DB 테이블이 있다고 하자.  
  
결제 정보를 저장하는 테이블이 `결제 번호, 결제 금액, 회원 ID, 회원 이름, 가맹점 ID, 가맹점 이름`으로 구성 되어 있다.
그리고 DB에 저장되는 필드명을 아주 간단하게만 써야 하는 상황이라 DB 테이블이 이렇게 구성 됐다고 한다면  

|PAY_NO|AMT|USER_ID|USER_NM|MERCHANT_ID|MERCHANT_NM|
  
그럼 이제 이 테이블에서 정보를 가져와 저장하는 객체 `PaymentInfo`가 있다.  
  
`PaymentInfo`에서 `결제 번호, 결제금액`은 변수로, 회원 정보는 `UserInfo` 객체에 저장을 하고, 가맹점 정보는 `MerchantInfo`에 저장을 하려한다.  
  
그렇게 되면 도메인 객체의 구성은 아래와 같을 것이다.  

```java
@Getter
@Setter
public class PaymentInfo{
    private String paymentNo;
    private BigDecimal paymentAmount;
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

**주의 : resultMap을 이용하여 Mapping을 할 필드는 `Setter`를 무조건 가지고 있어야 한다!**

<br>

이제 DB의 컬럼과 객체 변수 명도 다르고, **객체안에 객체**가 있는 형태가 됐다.  
  
사용하는 SQL문에 하나씩 Mapping 해서 사용하는 원시적인 방법도 있지만, **재사용성과 JOIN이 포함된 복잡한 구문들**을 위해 ResultMap을 사용해보자.  

Mybatis에선 ResultMap을 사용한 내가 원하는 대로, 테이블의 컬럼들을 객체에 **Mapping** 시킬 수 있다.  
  
위의 예제는 아래와 같이 ResultMap으로 만들 수 있다.  

```xml
    <resultMap id = "PaymentInfo" type ="com.example.PaymentInfo">
        <id property="paymentNo" column="PAY_NO"/>
        <result property="paymentAmount" column="AMT"/>
        <association property="UserInfo" javaType="com.example.UserInfo">
            <result property="userId" column="USER_ID"/>
            <result property="userName" column="USER_NM"/>
        </association>
        <association property="MerchantInfo" javaType="com.example.MerchantInfo">
            <result property="merchantId" column="MERCHANT_ID"/>
            <result property="merchantName" column="MERCHANT_NM"/>
        </association>
    </resultMap>
```

<br>

#### 간단 resultMap 속성 설명  

|속성|설명|
|`id`|컬럼 한개, 간단한 데이터 타입 필드 Mapping. 객체의 식별자 필드를 `id`로 지정|
|`result`|컬럼 한개, 간단한 데이터 타입 필드 Mapping. 객체의 일반적인 필드는 `result`로 지정|
|`association`|간단한 데이터 타입 필드가 아닌, 객체를 Mapping|
|`collection`|Collection, 즉 List같은 Collection을 Mapping|

<br>
이렇게 만든 resultMap을 select 구문에 resultMap으로 지정해주면  

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

이렇게 resultMap에서 지정한 대로 `PaymentInfo`에 Mapping이 되어 있는 것을 확인 할 수 있다.  

출처 : [MyBatis 공식 메뉴얼](https://mybatis.org/mybatis-3/ko/sqlmap-xml.html#Result_Maps)