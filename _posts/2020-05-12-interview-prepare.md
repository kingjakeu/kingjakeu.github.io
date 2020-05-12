---
title: "인터뷰 준비"
date: 2020-05-12 23:20:00 -0400
category: study
---

## 네트워크

RESTful
Representational State Transfer
URI를 통해 Resource를 명시, Stateless Protocol
HTTP 매소드를 통해 해당 자원의 CRUD Operation을 적용
Create(POST), Read(GET), Update(PUT), Delete(DELETE), HEAD(HEAD)

장점
- HTTP프로토콜 인프라 사용으로 REST API를 위한 별도 인프라 불필요
- 서버와 클라 역할 분리

단점
- 매소드 제한적
- 표준 존재 x

서버에 Client의 context(세션, 쿠키, 클라이언트의 다양한 상태 정보)를 저장 하지 않음, 즉 서버는 context정보를 신경쓰지 않아도 됌

캐시 처리 가능
계층화 - 구조적 유연성

[NHN 정리 글](https://meetup.toast.com/posts/92)
[TCP/IP 정리 글](https://d2.naver.com/helloworld/47667)
[네트워크 정리](https://asfirstalways.tistory.com/327)

HTTP status
200 - 정상
201 - 리소스 생성 요청 성공
400 - 요청 부적절
401 - 인증 안된 요청
403 - 리소스 존재, 하지만 응답하고 싶지 않음
405 - 사용 불가한 매소드
301 - URI이 변경 됐음
500 - 서버 문제 발생

HTTP & HTTPS
HTTP는 평문 통신, 통신 상대 확인 x, 완전성x
HTTP에 SSL이나 TLS를 조합해서 암호화 -> HTTPS
SSL은 상대를 확인 하는 증명서, SSL로 상대를 확인
상대 확인 안하면, 원래 보낸 곳인지 모름, 어디서 보냈는지 모름, 의미없는거 수신 -> DoS
정보의 정확성 = 안전성 -> SSL 사용해서 암호화 
원래 HTTP는 TCP와 직접 통신
HTTPS는 HTTP는 SSL, SSL은 TCP와 통신

HTTP
GET  
- HTTP Request Message에 Header URL에 ? 하고 정보가 보이고 데이터 크기가 제한적
- 캐싱 가능

POST - HTTP Message Body에 데이터 담김