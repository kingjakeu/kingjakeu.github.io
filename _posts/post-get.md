---
title: "POST와 GET의 차이"
date: 2021-05-20 22:45:00 +0900
category: study
lastmod : 2021-05-20 14:23:00 +0900
sitemap :
  changefreq : weekly
  priority : 1.0
---

<br>

우리는 쉽게 GET은 Resource(자원)을 조회 해오는 것이고, POST는 자원을 생성하는 것이라고 설명한다.  
그렇다면 Web Service에서의 POST와 GET의 자세한 차이점을 알아 보자.  

## GET의 특징

GET은 Request-URI에 해당하는 데이터를 요청할 때 사용된다.

## POST 와 GET 차이

|  | GET | POST |
|----|----|----|
| 브라우저 방문 기록 | 파라미터가 URL의 일부이기 때문에, 브라우저 히스토리에 남음 | 내역이 남지 않음 |
| 북마크 | 가능 | 불가능 |
| 뒤로 가기 버튼 | GET 요청이 재 실행, 브라우저 캐시에 결과가 저장 되있을 경우 캐시를 보여줌 | 데이터가 다시 제출된다는 알림이 뜸 |
| 요청 길이 제한 | 평균 2KB-8KB, 옛날 브라우저일 경우 255 BYTE(URL을 통해 보내기 때문) | HTTP Body를 통해 보내기 때문에 제한 없음 |
| 보안성 | 좋지 않음, 민감한 정보일 경우 사용하면 안됌 | GET 보다는 좋음 |
| 캐싱 | 자주 캐싱 됌 | 드물게 캐싱 가능 |

https://www.diffen.com/difference/GET-vs-POST-HTTP-Requests
https://www.guru99.com/difference-get-post-http.html
https://www.w3schools.com/tags/ref_httpmethods.asp

Java HashFunction
hashCode에 M(hash bucket) 크기 % 계산
M은 소수 일때 성능 최고, 부족해지면 M*2, LoadFactor 0.75 일때까지
Java 8이전에는 보조 해시 함수로 복잡, 하지만 8이후는 커지면 레드블랙트리로 바꾸기에 간단하게 상위 16비트 값 XOR연산
String의 경우 Horner’s Method로 해쉬에 31을 재귀로 곱한 값에 각 문자열 더하기

