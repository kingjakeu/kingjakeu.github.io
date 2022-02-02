# Redis Stream

redis 5.0부터 추가 된 자료구조
log처럼 append only 형태로 저장
Consumer Groups, blocking operation을 통한 대기 지원
Consumer Groups는 kafka와 비슷

append only 한 자료구조 답게  `XADD` 커맨드를 통해 Stream에 데이터를 입력함
mutiple field - value 형태 지원, CSV 포맷으로 입력하면 됌

```redis-cli
 > XADD mystream * user-id jake tx-amount 1000
```

`*`로 자동 생성, id는 `<millliiseconds>-<sequenceNumber>`의 조합으로 되어 있음

따라서 우린 Stream을 messaging system을 넘어 time-series 저장소로 봐야한다.
message를 time-range로 조회 할수 도 있다
consumer의 입장에서는 stream에 여러가지 방법으로 접근 할 수 있다.
message을 파티셔닝해 여러 consumer가 메시지의 일부만 볼수도 이ㅅ고, 

## Stream 데이터에 접근하는 법

### Querying byu Range

range를 통한 조회에는 start와 end id만 있으면 된다

``` redis-cli
> XRANGE mystream - + 
1)  1) ...
    2) 1)...

```

`-`는 최소, `+` 최대, `COUNT`옵션을 통해 개수 제한을 두고 조회 가능 `ASC` 순으로 나옴
`XRANGE`는 `log(n)`, `XREVRANGE` invert range 조회

## XREAD를 통한 listen

range를 통한 접근을 하지 않을때, 대부분 subscribe 형식으로 접근
Redis pub/sub과 비슷

1. stream은 데이터를 기다리는 여러 consumer들을 가질수 있음, 새로운 아이템은 데이터를 기다리는 모든 consumer에게 전달.
  
  > 이 형태는 각 consumer가 다른 element를 받는 blocking list와 다름, 하지만 fan out 기능은 pub/sub과 비슷

2. pub/sub은 fire & forget 저장되지 않는 형태, blocking list을 사용할땐 client가 메시지를 받으면 popped. 
하지만 stream은 다름. 모든 메시지는 append 되는 형태, 다른 consumer들은 마지막으로 받은 메시지 id를 기억하는 걸로 새로운 메시지가 온걸 알수 있음

3. Stream consumer group은 pub/sub 혹은 blocking list 가 가질수 없는 control 레벨을 제공.
다른 그룹이 같은 stream에 접근 했을 때,
 아이템을 처리한다는 ack를 explicit할 수 있고, pending 아이템을 inspect 할 수 있고, 
 처리 되지 않은 item들을 claiming 할수 있고 ,coherent history visibility for each single client, that is only able to see its private past history of messages.

 Stream에 도착한 새로운 메시지는 `XREAD` 커맨드를 통해 listen 할 수 있음

 ``` redis-cli
 > XREAD STREAMS mystream 0
 1) 1) "mystream"
    2) 1) 1) id
          2) 1)field....
 ```

non-blocking 형태의 `XREAD`, `STREAMS mystream 0` 0-0 보다 큰 id를 가진 모든 메시지를 가져옴
`STREAMS mystream yourstream 0` 여러 스트림에 접근 가능
`XREAD BLOCK 0` timeout 0 mili sec을 가진 block 옵션을 줄수도 있음, id 자리에 `$`를 통해 최대 값 조회도 가능

## Consumer Groups

같은 Stream에 여러 클라이언트가 접근 할 경우, `XREAD`는 N-client에 대한 fan-out을 제공
하지만 우린 같은 Stream에 붙은 여러 client들이 각자 다른 메시지를 가져가기를 원함

1. 각 메시지는 다른 consumer에게 전달 됌, 즉 여러 consumer들에게 같은 메시지는 전달 될 수 없음

2. Consumer Group에 속한 Counsumer들은 이름으로 identified 되야함, 
disconnet 된 후에도 stream consumer group은 모든 상태를 유지함.

3. 각 consumer group은 concept of the first ID never consumed 을 가지고 있음, 
consumer가 새로운 메시지를 물어볼때, 이전에 deliver된적없는 메시지를 줘야함

4. 메시지를 consuming 하기 위해 special command를 통해 ack를 explict 해줘야 함. 레디스는 ack를 메시지가 processed 되었다고 생각함

5. consumer group은 Pending 중인 모든 메시지를 찾아감 (Consumer group에 있는 consumer에게 delivered 된 메시지, 하지만 아직 처리됐다고 ack하지 않은것)

- `XGROUP`: Consumer group을 만들거나 삭제
- `XREADGROUP`: consumer group을 통해 read
- `XACK`: 메시지를 pending으로 표시

``` redis-cli
> XGROUP CREATE mystream mygroup $ MKSTREAM
```
 
Consumer 그룹 생성, `$` 생성후 최신 메시지 읽기 `0`을 통해 전체 가능, `MKSTREAM` 스트림이 없으면 스트림도 만듬

``` redis-cli
> XREADGROUP GROUP mygroup Alice COUNT 1 STREAMS mystream >
```

> alice는 consumer 이름
`>`는  context of consumer groups 안에서만 valid, 다른 consumser에게 메시지가 전달 되지 않음을 뜻함
1
