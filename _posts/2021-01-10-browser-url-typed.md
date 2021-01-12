---
title: "웹 브라우저에 주소를 입력하면 일어나는 일"
date: 2021-01-11 00:40:00 +0900
category: study
lastmod : 2021-01-11 00:40:00 +0900
sitemap :
  changefreq : weekly
  priority : 1.0
---

<br>

## 1. 브라우저의 URL 문법 확인

---

일단 웹 브라우저의 주소창에 `example.com`와 같은 웹 페이지 주소를 입력하게 되면, 웹 브라우저는 입력된 **URL의 문법**을 확인한다.  

<br>

## 2. DNS 조회

---

올바른 URL 이라고 판단이 된 후, 해당 URL을 **DNS Server에 조회** 한다.  
  
> 이때 먼저 **브라우저 캐시**에 해당 URL이 있는지 확인한다.  
> 해당 URL이 브라우저 캐시에 캐싱되어 있지 않다면 **로컬의 hosts 파일**에서 URL를 찾아보고,  
> 둘 다 존재하지 않는 경우에 **DNS Server**로 조회를 해당 URL를 조회한다.

### DNS 란

**DNS(Domain Naming System)**는 `example.com` 같은 사람이 읽을 수 있는 도메인 이름를 `192.0.2.1`과 같은 **IP 주소로 변환하는 프로토콜**이다.  

#### DNS 동작 원리

1. 인터넷 주소창에 `example.com`을 입력한다.  

2. `example.com`의 대한 요청을 **DNS 루트 이름 서버(.)에 전달**한다.  

3. 루트 서버는 **TLD(Top Level Domain, 최상위 도메인)** DNS 서버에 요청을 전달한다. `example.com`의 TLD는 `.com`.  

4. TLD 서버는 `example.com`의 **Domain Name Server(도메인 이름 서버)의 IP 주소**를 응답한다.  

5. TLD로 부터 응답 받은 **Domain Name Server**로 `example.com`대한 요청을 보내고, 응답 값으로 **해당 도메인의 IP주소를 확인** 받는다.  

> 기본적인 DNS 설정은 **ISP**에 의해 설정되어 있다.  

<br>

## 3. ARP 확인

---

**ARP(Address Resolution Protocol)**을 이용하여 서버의 IP주소와 MAC 주소를 찾는다.  

### ARP란

ARP(Address Resolution Protocol)는 **IP 주소의 MAC(Media Access Control) / 물리적 주소**를 찾기 위해 사용되는 프로토콜이다.

### ARP 동작 원리

ARP 요청은 목적지 IP 주소에 물리적 주소를 `FF:FF:FF:FF:FF:FF`로 설정하여 **브로드캐스트**한다.  
해당 IP에 설정된 서버는 **MAC 주소를 응답**한다. 이후, 응답 받은 MAC 주소를 이용하여 통신한다.  

<br>

## 4. 서버 연결

---

IP주소와 포트번호로 3-way handshaking을 통해 **TCP 연결**을 시작한다.  
  
서버와의 연결을 확인 한 후, **HTTP통신**을 통해 **서버로부터 리소스를 받아온다**.  
  
브라우저가 서버로부터 리소스를 받게되면 **해당 리소스를 브라우저에 랜더링**을 하게 되고, 사용자는 렌더링된 브라우저를 통해 해당 웹페이지를 볼 수 있게된다.  

<br>

## 정리

---

1. 사용자는 브라우저에 웹 사이트 주소 입력
2. 브라우저는 입력된 **URL 문법 확인**
3. **DNS** 조회, **IP 주소** 획득
4. **ARP**를 이용, **MAC 주소** 획득
5. **TCP 연결** 시작
6. 연결 후, **HTTP 통신** 이용하여 서버는 웹 사이트 **리소스 응답**
7. 응답 받은 리소스, **브라우저 랜더링**
8. 랜더링된 페이지 사용자 이용

<br>

### References

---

[https://aws.amazon.com/ko/route53/what-is-dns/](https://aws.amazon.com/ko/route53/what-is-dns/)  
[https://www.cloudflare.com/ko-kr/learning/dns/what-is-dns/](https://www.cloudflare.com/ko-kr/learning/dns/what-is-dns/)  
[https://www.geeksforgeeks.org/how-address-resolution-protocol-arp-works/](https://www.geeksforgeeks.org/how-address-resolution-protocol-arp-works/)  
