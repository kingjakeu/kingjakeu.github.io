---
title: "Redis Stream (레디스 스트림) 기본 정리"
date: 2022-02-07 23:59:00 +0900
category: study
lastmod : 2021-02-07 23:59:00 +0900
sitemap :
  changefreq : weekly
  priority : 1.0
---

# Redis Stream이란?

Redis Stream(레디스 스트림)은 Redis 5.0부터 추가 된 자료구조로, 
log 파일처럼 **append only**로 저장되는 구조를 가지고 있다.

메시징 시스템인 Kafka와 비슷하게 동작하며, 메시지를 읽는 Consumer와 여러 Consumer를 관리하는 **Consumer Group** 또한 지원한다.

Kafka와 비교하며 쉽게 이해하자면, Kafka의 topic은 Redis의 stream, 
Kafka와 동일하게 Producer와 Consumer가 있으며, 다중 Consumer를 관리하기 위해 Redis Stream은 Consumer Group을 지원한다.

Redis Pub/Sub과 비교를 한다면, Redis Stream은 메시지가 전달 후 사라지는(fire & forget)이 아니며, 수신 여부를 구분 지을 수 있고,
다중 Consumer 존재시, Redis pub/sub과 다르게 모든 Consumer에게 메시지를 전달하는 것이 아니라 각각 나눠서 처리가 가능하다.


### Redis Stream, Kafka, Redis Pub/Sub 비교

||메시지 주제 관리|메시지 휘발성|메시지 확인 여부|Consumer 분산 처리 여부|
|---|---|---|---|---|
|Redis Stream|Stream|x|o|o|
|Kafka|Topic|x|o|o|
|Redis pub/sub|Queue|o|x|x (발행시, 모든 consumer가 읽음)|

---

## Redis Stream 기본 동작

### Stream에 데이터 추가

Append only 한 자료구조 답게 `XADD` 커맨드를 통해 Stream에 **데이터를 추가**한다. CSV 포맷으로 입력하여 한번에 여러개의 데이터를 추가할 수 있다.

```redis-cli
 > XADD <stream-key> <message-id> <field> <name> ... <field> <name>
 > XADD mystream * user-id jake tx-amount 1000
 1518951480106-0
```

메시지의 id는 `*`로 자동 생성 가능하며 권장된다. 왜냐하면 자동 생성된 id는 `<millliiseconds>-<sequenceNumber>`의 조합으로 되어 있다.  

따라서 우린 Stream을 messaging system을 넘어 time-series 저장소로도 볼 수 있고, 자동 생성된 id를 이용, message를 time-range로 조회할 수 도 있다.

### Stream 데이터에 접근

#### Querying by Range

Range를 통한 조회(범위 조회)에는 `start`와 `end id`만 있으면 된다.

``` redis-cli
> XRANGE <stream-key> <start-id> <end-id> [COUNT <count-num>]
> XRANGE mystream - + 
1)  1) ...
    2) 1)...
```

`-`는 최소 id, `+` 최대 id를 의미하며, `COUNT`옵션을 통해 개수 제한을 두고 조회 가능 하며 `ASC` 순으로 조회된다.  

`XRANGE`의 time-complexity는 `log(n)`이며, `XREVRANGE`를 이용한 Invert Range 조회도 가능하다.

#### XREAD를 통한 listen

Range를 통한 조회 접근을 하지 않을때, **Subscribe(구독)**형식으로 접근을 할때는 `XREAD` 커맨드를 사용한다.

`XREAD`를 통한 조회 + Redis Stream의 특징은 

1. Redis stream은 데이터를 기다리는 **여러 consumer들을 가질수 있음**, 새로운 아이템은 데이터를 기다리는 모든 consumer에게 전달.
  > 이 형태는 각 consumer가 다른 element를 받는 Blocking List와 다름, 하지만 fan-out 기능은 Redis Pub/Sub과 비슷하다.

2. Redis Pub/Sub은 `fire & forget`으로 메시지 발행 후 저장되지 않는 형태, Blocking List을 사용할땐 client가 메시지를 받으면 popped. 
하지만 **Redis Stream은 모든 메시지는 append** 되는 형태로, 다른 consumer들은 마지막으로 받은 메시지 id를 기억하는 걸로 새로운 메시지가 온걸 알 수 있다.

3. Stream Consumer Group은 Pub/Sub 혹은 Blocking List가 가질수 없는 **여러 레벨 control을 제공**한다.
여러 그룹이 같은 stream에 접근 했을 때, 
- `ACK`를 통해 해당 consumer가 해당 메시지를 처리한다는 것을 명시 할 수 있고, 
- `ACK`가 오지 않은 메시지들을 `Pending` 아이템을 관리 할 수 있다. 
- 이렇게 Pending된 처리 되지 않은 아이템들을 `CLAIM`을 통해 다른 Consumer가 처리 할 수 있다.

Stream에 도착한 새로운 메시지는 `XREAD` 커맨드를 통해 Listen 할 수 있다.
 ``` redis-cli
 > XREAD <COUNT> <count-num> STREAMS <stream-key> <start-id>
 > XREAD STREAMS mystream 0
 1) 1) "mystream"
    2) 1) 1) id
          2) 1)field....
 ```

non-blocking 형태의 `XREAD STREAMS mystream 0`은 0-0 보다 큰 id를 가진 모든 메시지를 가져온다.  
`XREAD STREAMS mystream yourstream 0`으로 여러 스트림에도 접근 가능하다.  
`XREAD BLOCK 0` (timeout = 0 milisecond)을 가진 `blocking` 옵션을 줄 수도 있으며, id 자리에 `$`를 통해 최대 값 조회도 가능하다.

### Stream 데이터 삭제

```redis-cli
> XDEL <stream-key> <message-id>
> XDEL mystream 1526654999635-0
```

`XDEL`를 통해 특정 메시지를 삭제 할 수 있다. 하지만 메모리 부족 같은 상황이 아니라면 **추천하지 않는다**.   
발생한 데이터를 저장하고 time-series형태로 관리 할 수 있는 것이 Redis Stream의 장점이고, 후에 나오는 `XACK`와 `XCLAIM`을 통해 메시지를 충분히 관리 할 수 있다.

---

## Consumer Groups

같은 Stream에 여러 클라이언트(=consumer)가 접근 할 경우, `XREAD`는 N-client에 대한 fan-out(모두가 메시지를 받는)을 제공한다.  

하지만 우리는 같은 Stream에 붙은 **여러 클라이언트들이 각자 다른 메시지**를 가져가기를 원하며, **Consumer Group**을 통해 해당 기능을 사용할 수 있다.

Consumer Group은 *pseudo consumer*로 Stream으로부터 데이터를 가져간다.  

즉, *Consumer Group은 여러 Consumer를 가진 Group이지만, 이론상 하나의 consumer*로 Stream으로 데이터를 가져가고, 
Consumer Group 내에서 데이터를 나눠 처리하는 것이다.

**Consumer Group의 특징은**

1. 각 메시지는 각각 다른 consumer에게 전달 된다. 즉, *여러 consumer들에게 같은 메시지는 전달 될 수 없다*.

2. Consumer Group에 속한 counsumer들은 이름으로 **identified(구분)**되어야 한다. 
Consumer는 disconnet된 후에도 Consumer Group은 모든 상태를 유지한다.

3. 각 Consumer Group은 `concept of the first ID never consumed`을 지켜야 한다.
즉, consumer가 새로운 메시지를 물어볼때, *이전에 전달이 된 적없는 메시지를 줘야한다.*

4. 메시지를 consuming했다는 것을 `ack`를 통해 **명시해줘야 한다**. 레디스는 **ack된 메시지는 처리되었다**고 생각하고, Consumer Group으로부터 제외시킨다. 

5. Consumer Group은 Pending 중인 모든 메시지를 찾을 수 있다. 
**Pending된 메시지**란 Consumer group에 있는 consumer에게 **delivered(전달)된 메시지**, 하지만 **아직 처리됐지 않은**, `ack`하지 않은 메시지를 의미한다.

#### Redis Stream 메시지 상태 정리

- `DELIVERED`: Consumer Group이 Stream으로부터 메시지를 읽어와, 그룹내 **consumer에게 전달된 상태**
- `PENDING` : Delivered 되었지만 Consumer가 **아직 메시지를 처리하지 못한 대기 상태**
- `ACK` : Consumer가 메시지를 처리하고, `XACK` 커맨드를 통해 **해당 메시지의 처리 완료를 명시한 상태**

### Consumer Group 만들기 

``` redis-cli
> XGROUP CREATE <stream-key> <group-name> <start-id> [MKSTREAM]
> XGROUP CREATE mystream mygroup $ MKSTREAM
OK
```

`XGROUP CREATE`를 통해 Consumer Group을 생성 할 수 있으며, 동시에 `MKSTREAM`옵션을 통해 스트림이 없으면 스트림 생성도 가능하다.

`start-id`설정 또한 조정 가능하다.
- `$` : 생성 후, 발생한 최신 메시지 읽기
- `0` : 처음부터 전체 메시지 읽기


### Consumer Group 통한 데이터 접근 

``` redis-cli
> XREADGROUP GROUP <group-name> <consumer-name> <COUNT> <count-num> STREAM <stream-key> <message-id> 
> XREADGROUP GROUP mygroup Alice COUNT 1 STREAMS mystream >
1) 1) "mystream"
   2) 1) 1) 1526569495631-0
         2) 1) "message"
            2) "apple"
```

`>`는  context of consumer **groups 안에서만 유효**하며, 다른 consumser에게 메시지가 전달 되지 않음을 뜻한다.

### Consumer Group 전달된 메시지 완료 처리

Consumer Group을 통해 consumer에게 전달된 메시지는 `ack`를 통해, Redis에 **처리 완료를 명시**해줘야한다.  

`XACK`를 통해 명시 가능하며, `XACK`되지 않은 메시지는 `pending`상태로 여겨진다.

``` redis-cli
> XACK <stream-key> <group-name> <message-id>
> XACK mystream mygroup 1526569495631-0
(integer) 1

> XREADGROUP GROUP mygroup Alice STREAMS mystream 0
1) 1) "mystream"
   2) (empty list or set)
```
> `XACK`후, 처리되야 하는 메시지가 없는것을 확인 할 수 있다.

---

## Redis Stream에서의 데이터 메시지 복구 처리

그렇다면 consumer가 처리에 실패해서 `Pending` 된 상태로 놓여져 있는 데이터들은 어떻게 복구하고 다시 정상 처리 할 수 있을까?  

Redis Stream은 `XPENDING`을 통해 `pending`상태의 **메시지 조회를 지원**하고, `XCLAIM`을 통해 **메시지의 재처리를 지원**한다.

### Pending 상태의 메시지 조회

``` redis-cli
> XPENDING <stream-key> <group-name> [[IDLE <min-idle-time>] <start-id> <end-id> <count> [<consumer-name>]]
> XPENDING mystream mygroup - + 10
1) 1) 1526569498055-0
   2) "Bob"
   3) (integer) 74170458
   4) (integer) 1
2) 1) 1526569506935-0
   2) "Bob"
   3) (integer) 74170458
   4) (integer) 1
```

`XPEDNING`은 스트림에 consumer group 별 `pending` 된 **메시지를 조회 할 수 있다**.  

여기서 `IDLE` 옵션을 통해 메시지를 전달 받은 후 `<min-idle-time>`(millsecond)을 지난 pending message를 조회 할 수도 있다.

> `XPENDING` 조회를 통해 나온 결과 값은 1) message-id, 2) consumer-name, 3) 메시지를 받고 지난 시간, 4) Delivered된 숫자를 의미한다.

### Claim을 통한 메시지 재처리

`XCLAIM`을 이용하면 **해당 message의 consumer를 바꿀 수 있다**.  

예를 들어, `1526569498055-0`의 consumer는 `Bob`이었지만, 처리하지 못하고 pending 상태가 되었다.

``` redis-cli
> XCLAIM <stream-key> <group-name> <consumer-name> <min-idle-time> <ID-1> <ID-2> ... <ID-N>
> XCLAIM mystream mygroup Alice 3600000 1526569498055-0
1) 1) 1526569498055-0
   2) 1) "message"
      2) "orange"
```
해당 `XCLAIM` 커맨드를 통해 `Alice`로 consumer가 바뀌었다.  

`XCLAIM`은 consumer를 바꾸기 때문에 `XPENDING` 조회시, 3) 메시지를 받고 지난 시간(idle-time), 4) Delivered된 숫자를 초기화 한다.  

또한 consumer가 바뀐거지 `ack`를 한건 아니기 때문에, `XCLAIM`후 `XACK`가 없다면, 그냥 `Alice`가 pending 중인 메시지가 된것이다.

---

## Redis Stream 정보 조회

Consumer를 설정 하고, Subscription이 자동으로 돌아가는 시스템을 구축하였다면, Redis Stream은 메시지 생성과 처리가 돌아가고 있을 것이다.  

그렇다면 우리는 **Redis Stream 세부 정보**를 `XINFO`를 통해 알아볼 수 있다.

### Stream 정보 조회

``` redis-cli
> XINFO STREAM <stream-key>
> XINFO STREAM mystream
 1) length
 2) (integer) 13
 3) radix-tree-keys
 4) (integer) 1
 5) radix-tree-nodes
 6) (integer) 2
 7) groups
 8) (integer) 2
 9) first-entry
10) 1) 1526569495631-0
    2) 1) "message"
       2) "apple"
11) last-entry
12) 1) 1526569544280-0
    2) 1) "message"
       2) "banana"
```

`XINFO STREAM`를 통해 **Stream 자체의 정보**를 알수 있다. 
> 1) 스트림의 크기, 7) consumer group 수, 10) first-id, 12) last-id

### Stream Consumer Group 정보 조회

``` redis-cli
> XINFO GROUPS <stream-key>
> XINFO GROUPS mystream
1) 1) name
   2) "mygroup"
   3) consumers
   4) (integer) 2
   5) pending
   6) (integer) 2
   7) last-delivered-id
   8) "1588152489012-0"
...
```

`XINFO GROUPS`을 통해 해당 **스트림이 가지고 있는 Consumer Group**을 조회할 수 있다.
> 1) consumer group 이름, 3) 그룹에 포함된 consumer 수, 5) 해당 group이 pending 중인 message 수

### Stream Consumer Group내 Consumer 정보 조회

```redis-cli
> XINFO CONSUMERS <stream-key> <group-name>
> XINFO CONSUMERS mystream mygroup
1) 1) name
   2) "Alice"
   3) pending
   4) (integer) 1
   5) idle
   6) (integer) 9104628
...
```

`XINFO CONSUMERS`을 통해, **해당 스트림의 해당 그룹의 Consumer들의 정보**를 알 수 있다.
> 1) consumer 이름, 3) pending 중인 message 개수, 5) 마지막 message가 도착한 후 지난 시간

---

## Conclusion

기본적인 Redis Stream의 특징, 중요 Command를 통한 사용법에 대해 정리해봤다.  

Kafka 또는 Redis pub/sub에 비해 압도적으로 정보가 적기에, 기억 할겸 누군가에게 도움이 될까 해서 작성해본다.
사실 Redis 공식 문서에 더 자세하게 설명되어 있지만, 쉽게 이해해보기 위해 노력해봤다.  

다음 글은 Springboot을 통해 Redis Stream을 사용한 예제에 대해 작성할 예정이다.

### Reference
- [https://redis.io/topics/streams-intro](https://redis.io/topics/streams-intro)
