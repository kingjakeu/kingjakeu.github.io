---
title: "HTTP POST와 PUT의 차이"
date: 2020-07-15 22:45:00 +0900
category: study
lastmod : 2020-07-15 22:45:00 +0900
sitemap :
  changefreq : weekly
  priority : 1.0
---

흔히들 POST와 PUT의 차이를 물어본다.  
그때 마다 "POST는 resource를 create하는 거고, PUT은 update 하는 거에요~?!" 라고 뜬 구름 잡듯이 얘기해왔다.  
말도 안되는 답변이란 걸 나도 알기에, 좀 더 정확하게 알아보고자 한다.  
<br>

## 1. HTTP POST에 대해

### POST의 정의

Request의 포함된 **Entity(Http body에 해당)**을 **Request-URI에 정의된 리소스**의
하위(Suboridiate) Entity로 **새롭게 생성하는** 요청을 서버에 보낼때 사용되는 **Http Method**이다.  
따라서 Request-URI는 리소스의 Entity를 나타내는 **Collection URI**여야 한다.  

> 예를 들어, 신발들의 정보를 저장하는 `shoes`가 있다. 그리고 각 신발의 정보들은 `shoes`라는 큰 카테고리 밑에 하위 item으로 등록되고 저장될 것이다.  
>
> 결국, `shoes`는 신발들이 모인 **Collection**이다. 이걸 URI로 적용해본다면 신발 정보의 **Collection-URI**는 `/shoes`가 된다.

POST는 `/shoes`라는 collection URI의 하위에 **새로운 신발(Entity)**을 **Create(생성)**할 때 사용되는 Http Method라고 할 수 있다.

### POST의 특징

+ **POST Request는 Idempotent(멱등) 하지 않다**  
Idempotent(멱등) 하지 않다 = 여러번의 **재시도에 대한 모든 결과값이 동일하지 않다**는 것이다.  
즉, POST로 동일한 entity의 Request를 **N번 보낸다면, N개의 다른 리소스들이 생성되는 것이다**.

+ **POST Request의 Response는 Caching 가능 하다**  
POST request는 **Cache-Control or Expires**가 Http header 올바르게 정의되어 있다면 Response(응답) 값을 캐싱해도 된다. 대신 Response를 캐시로 응답 했다면, **HTTP 300** 으로 해당 응답이 캐시에서 왔다는 것을 표시해줘야 한다.

<br>

## 2. HTTP PUT에 대해

### PUT의 정의

Request-URI에 있는 **Resource가 존재한다면**, Request에 있는 Entity에 값으로 리소스를 **Update(갱신)**한다.  
만약 **Resource가 존재하지 않고**, Request-URI와 ㄲesource-URI가 올바르다면 리소스를 **Create(생성)** 할 때 사용되는 Http Method이다.
> if resource 존재 -> update  
> else -> create

### PUT의 특징

+ **Resource Identifier**  
PUT은 기존의 `/shoes`라는 collection URI에 더하여,`/shoes/{shoe_id}`로 해당 resource의 **Resource Identifier**를 나타내줘야 한다.

+ **PUT Request는 Idempotent(멱등) 하다**  
PUT request로는 새로운 정보가 계속되서 생성되지 않는다. 여러번 재시도를 하더라도, 동일한 결과 값을 받는다. 즉, PUT request는 **idempotent(멱등) 하다**.

+ **POST Request의 Response는 Caching 할 수 없다**  
PUT request는 idempotent하다. 하지만 **Response(응답) 값을 캐싱하면 안된다**.

<br>

## 3. POST & PUT 예시

`POST /shoes`  
   = Http Body에 있는 정보로 *새로운 Shoe 하위 Resource 생성*  

`PUT /shoes/{존재하는_SHOE_ID}`  
   = 존재하는_SHOE_ID에 *존재하던 정보를 Overwrite(덮어쓰기)해서 정보를 갱신*  

`PUT /shoes/{존재하지_않는_SHOE_ID}`  
   = 존재하지_않는_SHOE_ID로 *새로운 Resource 생성*  

<br>

## 4. POST & PUT 비교

||POST|PUT|
|------|---|---|
|Resouce identifier 유무| x | o |
|Idempotent(멱등) 한가?| x | o |
|Response를 Caching 해도 되는가?| o (대신 300으로 표시) | x |

<br>

### References

[https://restfulapi.net/rest-put-vs-post/](https://restfulapi.net/rest-put-vs-post/)  
[https://www.keycdn.com/support/put-vs-post](https://www.keycdn.com/support/put-vs-post)  
[(https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html)  
[https://restfulapi.net/resource-naming/](https://restfulapi.net/resource-naming/)  
