---
title: "Springboot에서 Redis Stream 활용"
date: 2022-02-07 23:59:00 +0900
category: study
lastmod : 2021-02-07 23:59:00 +0900
sitemap :
  changefreq : weekly
  priority : 1.0
---

Springboot에서 Redis Stream(레디스 스트림)을 사용하기 위한 기본 설정

- Spring Boot 2.4.0 이상
- spring-boot-starter-data-redis 2.4.0 이상
- Redis 5.0 이상
- Redis Client로 Lettuce 사용

위 설정이 완료되면 Redis Stream을 Spring Boot를 통해 사용해보자.

## Redis 기본 Command 구현

### XACK 

``` java
public void ackStream(String consumerGroupName, MapRecord<String, Object, Object> message){
    this.redisTemplate.opsForStream().acknowledge(consumerGroupName, message);
}
```

### XCLAIM

``` java
public void claimStream(PendingMessage pendingMessage, String consumerName){
    RedisAsyncCommands commands = (RedisAsyncCommands) this.redisTemplate
            .getConnectionFactory().getConnection().getNativeConnection();

    CommandArgs<String, String> args = new CommandArgs<>(StringCodec.UTF8)
            .add(pendingMessage.getIdAsString())
            .add(pendingMessage.getGroupName())
            .add(consumerName)
            .add("20")
            .add(pendingMessage.getIdAsString());
    commands.dispatch(CommandType.XCLAIM, new StatusOutput(StringCodec.UTF8), args);
}
```

### RANGE 조회

```java
public List<MapRecord<String, Object, Object>> findStreamMessageByRange(String streamKey, String startId, String endId){
    return this.redisTemplate.opsForStream().range(streamKey, Range.closed(startId, endId));
}

// Range 조회를 활용한 message 단 건 조회
public MapRecord<String, Object, Object> findStreamMessageById(String streamKey, String id){
    List<MapRecord<String, Object, Object>> mapRecordList = this.findStreamMessageByRange(streamKey, id, id);
    if(mapRecordList.isEmpty()) return null;
    return mapRecordList.get(0);
}
```

### XGROUP - ConsumerGroup

``` java
public void createStreamConsumerGroup(String streamKey, String consumerGroupName){
    // Stream이 존재 하지 않으면, MKSTREAM 옵션을 통해 만들고, ConsumerGroup또한 생성한다
    if (Boolean.FALSE.equals(this.redisTemplate.hasKey(streamKey))){
        RedisAsyncCommands commands = (RedisAsyncCommands) this.redisTemplate
                .getConnectionFactory()
                .getConnection()
                .getNativeConnection();

        CommandArgs<String, String> args = new CommandArgs<>(StringCodec.UTF8)
                .add(CommandKeyword.CREATE)
                .add(streamKey)
                .add(consumerGroupName)
                .add("0")
                .add("MKSTREAM");

        commands.dispatch(CommandType.XGROUP, new StatusOutput(StringCodec.UTF8), args);
    }
    // Stream 존재시, ConsumerGroup 존재 여부 확인 후 ConsumerGroupd을 생성한다
    else{
        if(!isStreamConsumerGroupExist(streamKey, consumerGroupName)){
            this.redisTemplate.opsForStream().createGroup(streamKey, ReadOffset.from("0"), consumerGroupName);
        }
    }
}

// ConsumerGroup 존재 여부 확인
public boolean isStreamConsumerGroupExist(String streamKey, String consumerGroupName){
    Iterator<StreamInfo.XInfoGroup> iterator = this.redisTemplate
            .opsForStream().groups(streamKey).stream().iterator();

    while(iterator.hasNext()){
        StreamInfo.XInfoGroup xInfoGroup = iterator.next();
        if(xInfoGroup.groupName().equals(consumerGroupName)){
            return true;
        }
    }
    return false;
}
```

## Redis Stream Consumer 구현

대부분 Redis Stream을 사용하는 이유로는 Consumer Group을 설정하고, Consumer Group을 통해 Redis Stream의 메시지를 Event-Driven 형태로 Subscribe하기 위함일 것이다.

Spring Boot에서 해당 기능들을 구현 하기 위해선,
1. 메시지를 받을 수 있는 `StreamListener`를 implement 한 Consumer Bean을 생성
2. `StreamMessageListenerContainer`을 통해 Redis에서 StreamMessage을 받을 수 있게 설정
3. 해당 `StreamMessageListenerContainer`을 사용하는 `Subscription`을 Stream 정보와 함께 설정해야 한다.

우선 Consumer Bean 부터 생성해보자.

```java
@Slf4j
@Component
@RequiredArgsConstructor
public class RedisStreamConsumer implements StreamListener<String, MapRecord<String, Object, Object>>, InitializingBean, DisposableBean {
 
    private StreamMessageListenerContainer<String, MapRecord<String, Object, Object>> listenerContainer;
    private Subscription subscription;
    private String streamKey;
    private String consumerGroupName;
    private String consumerName;

    // 위에 구현한 Redis Streamd에 필요한 기본 Command를 구현한 Component
    private final RedisOperator redisOperator; 
    ....
}
```

Consumer Bean은 3개의 interface를 implements 하도록 한다.
- `InitializingBean`: Bean 생성시 `StreamMessageListenerContainer`와 `Subscription` 생성

```java
@Override
public void afterPropertiesSet() throws Exception {
    // Stream 기본 정보
    this.streamKey = "mystream";
    this.consumerGroupName = "consumerGroupName";
    this.consumerName = "consumerName";

    // Consumer Group 설정
    this.redisOperator.createStreamConsumerGroup(streamKey, consumerGroupName);

    // StreamMessageListenerContainer 설정 
    this.listenerContainer = this.redisOperator.createStreamMessageListenerContainer();

    //Subscription 설정
    this.subscription = this.listenerContainer.receive(
            Consumer.from(this.consumerGroupName, consumerName),
            StreamOffset.create(streamKey, ReadOffset.lastConsumed()),
            this
    );

    // 2초 마다, 정보 GET
    this.subscription.await(Duration.ofSeconds(2));

    // redis listen 시작
    this.listenerContainer.start();
}

// RedisOperator :: 기본 StreamMessageListenerContainer 생성 
public StreamMessageListenerContainer createStreamMessageListenerContainer(){
    return StreamMessageListenerContainer.create(
            this.redisTemplate.getConnectionFactory(),
            StreamMessageListenerContainer
                    .StreamMessageListenerContainerOptions.builder()
                    .hashKeySerializer(new StringRedisSerializer())
                    .hashValueSerializer(new StringRedisSerializer())
                    .pollTimeout(Duration.ofMillis(20))
                    .build()
    );
}
```

- `DisposableBean`: Bean destroy시 `StreamMessageListenerContainer`와 `Subscription`의 연결을 해제

``` java
@Override
public void destroy() throws Exception {
    if(this.subscription != null){
        this.subscription.cancel();
    }
    if(this.listenerContainer != null){
        this.listenerContainer .stop();
    }
}
```

- `StreamListener`: RedisStream에서 메시지가 도착했을 때, 처리

``` java
@Override
public void onMessage(MapRecord<String, Object, Object> message) {
    // 처리할 로직 구현 ex) service.someServiceMethod();
    // 이후, ack stream
    this.redisOperator.ackStream(streamKey, message);
}
```

## Pending Message 처리 Scheduler 구현 

Redis Stream의 메시지를 수신하는 Consumer는 구현됐지만, `Consumer.onMessage`에서 로직을 처리를 하다가 에러가 발생, `XACK`를 하지 않은 메시지들은 어떻게 될까?

당연히 `pending`상태로 Stream에 저장되어 있다. 그렇다면, `pending` 상태의 메시지들을 조회하여, 처리할 수 있는 `PendingMessageScheduler`를 만들어 보자.

```java
@Slf4j
@EnableScheduling
@Component
@RequiredArgsConstructor
public class PendingMessageScheduler implements InitializingBean {
    private String streamKey;
    private String consumerGroupName;
    private String consumerName;
    private final RedisOperator redisOperator;
    ...
}
```

`PendingMessageScheduler`는 `@EnableScheduling`를 통해 스케쥴 된 시간마다, 아래의 로직을 처리 할 것이다.

- `pending` 상태의 메시지를 조회
- 해당 메시지를 `XCLAIM`을 통해 consumer를 변경
- 메시지 처리 후 `XACK`을 통해 `ack` 처리

``` java
@Scheduled(fixedRate = 10000) // 10초 마다 작동
public void processPendingMessage(){
    // Pending message 조회
    PendingMessages pendingMessages = this.redisOperator
            .findStreamPendingMessages(streamKey, consumerGroupName, consumerName);

    for(PendingMessage pendingMessage : pendingMessages){
        // claim을 통해 consumer 변경
        this.redisOperator.claimStream(pendingMessage, consumerName);
        try{
            // Stream message 조회
            MapRecord<String, Object, Object> messageToProcess = this.redisOperator
                    .findStreamMessageById(this.streamKey, pendingMessage.getIdAsString());
            if(messageToProcess == null){
                log.info("존재하지 않는 메시지");
            }else{
                // 해당 메시지 에러 발생 횟수 확인
                int errorCount = (int) this.redisOperator
                        .getRedisValue("errorCount", pendingMessage.getIdAsString());

                // 에러 5회이상 발생
                if(errorCount >= 5){
                    log.info("재 처리 최대 시도 횟수 초과");
                }
                // 두개 이상의 consumer에게 delivered 된 메시지
                else if(pendingMessage.getTotalDeliveryCount() >= 2){
                    log.info("최대 delivery 횟수 초과");
                }else{
                    // 처리할 로직 구현 ex) service.someServiceMethod();
                }
                // ack stream
                this.redisOperator.ackStream(consumerGroupName, messageToProcess);
            }
        }catch (Exception e){
            // 해당 메시지 에러 발생 횟수 + 1
            this.redisOperator.increaseRedisValue("errorCount", pendingMessage.getIdAsString());
        }
    }
}
```

