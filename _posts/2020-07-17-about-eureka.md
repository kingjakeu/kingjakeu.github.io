---
title: "Eureka(유레카)란"
date: 2020-07-17 15:35:00 +0900
category: springboot
lastmod : 2020-07-17 15:35:00 +0900
sitemap :
  changefreq : weekly
  priority : 1.0
---

<br>

## 1. Eureka란

**Eureka(유레카)**란 **로드 밸런싱**과 **Middle-tier server**(Client와 Application server 사이)장애시 **장애 조치 처리를 목적**으로 한 **REST 기반 서비스**다.(넷플릭스에서 만듬)  

### 좀 더 자세히 말하자면

**MSA**에서 각 서비스별로 Application들이 나뉘어져 있다. 각 Application들은 서로 간의 통신을 위해, 서로 **연결 정보 (ip, hostname, port)**를 알아야한다. 그럼 이 연결 정보를 **Load Balancer**에 다 등록하게 된다.  

하지만 MSA 구조를 **클라우드**에 구축을 하게 되면, 각 Service Application들의 연결 정보는 **계속해서 바뀌게 된다**. 그럼 그때마다 Load Balancer에 구성된 설정들을 바꿔야 하는 말도 안되는 상황이 벌어진다.  

클라우드 환경에서의 연결 정보 (ip, hostname, port)의 **Registering(등록)과 De-registering(해지)**를 바로바로 적용할 수 있게 해주는 게 **Eureka**이다.  

### Service Discovery & Registry

Eureka에 대한 자료를 찾아보면, Eureka는 **Service Discovery & Registry**이란 말이 많이 나온다.  

+ **Discovery** = 다른 서비스의 정보를 **찾는 것**
+ **Registry** = 서비스의 정보를 **등록하는 것**

즉, Registry를 통해 등록 된 서비스들의 정보를 찾는 것이 Discovery다.  

-----

## 2. Eureka의 구성

Eureka는 Eureka **Client(클라이언트)**와 Eureka **Server(서버)**로 구성된다.  

### Eureka Client  

+ MSA에서 **각 Service들**은 Eureka Client다.
+ Eureka Client들은 **스스로를** Eureka Server에 등록한다.
+ Eureka Client들은 자신들의 **연결 정보(ip, hostname, port)의 Meta Data들**을 Eureka 서버에 등록한다.
+ Eureka Client들은 Eureka Server에 저장된 **Registry 정보를 수신**하고, 자신의 **Local환경에 저장**한다.
+ Eureka Client들은 Eureka Server에게 전달 받은 **Registry 정보를 통해 다른 Client들의 정보**를 알 수 있다.

### Eureka Server

+ Eureka Server는 **Registry**에 Eureka Client들을 등록한다.
+ Eureka Client들이 보낸 **Meta Data들을 Eureka Server는 Registry에 등록**한다.
+ Eureka Server는 Client들로부터 **30초마다**(Default value) Client들이 작동 중임을 알 수 있는 **Heartbeat**를 수신한다.
+ **Heartbeat가 오지 않은 경우** Server는 Client가 죽었다고 판단, 90초 내에 해당 Client의 정보를 **Registry에서 삭제**한다.
+ 이렇게 유지된 Registry 정보를 **Server는 Client들에게 전달**한다.

<br>

### References

[https://github.com/Netflix/eureka/wiki](https://github.com/Netflix/eureka/wiki)  
[https://www.baeldung.com/spring-cloud-netflix-eureka](https://www.baeldung.com/spring-cloud-netflix-eureka)  
[https://github.com/phantasmicmeans/Spring-Cloud-Netflix-Eureka-Tutorial](https://github.com/phantasmicmeans/Spring-Cloud-Netflix-Eureka-Tutorial)  
