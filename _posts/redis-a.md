---
title: "Redis(레디스) 적용기"
date: 2020-11-09 22:35:00 +0900
category: study
lastmod : 2020-11-09 22:35:00 +0900
sitemap :
  changefreq : weekly
  priority : 1.0
---

우선 Docker를 활용하여 레디스를 실행 해야한다.

``` sh
    docker pull redis
    docker run -d -p 6379:6379 --name redis redis
```

도커로 레디스 실행후 레디스를 명령어로 조작 할 수 있는 redis cli 또한 실행

``` sh
    docker run -it --link redis:redis --rm redis redis-cli -h redis -p 6379
```

간단한 CLI 명령어
KEYS * : 모든 키 보기
GET <key>: String value 조회
HGET <key>: Hash value 조회
HGETALL <key>: Hash value 전체 조회
lrange <key> <start> <end>: start - end 범위의 리스트 조회
smembers <key>: Set 조회
ZRANGEBYSCORE <key> <min> <max>: sortlist 조회
type <key>: 키 타입 조회


Springboot 프로젝트와의 연동
`build.gradle`에 `compile ('org.springframework.boot:spring-boot-starter-data-redis')` 의존성 추가를 한다.

`application.yml`에 redis host와 port를 정해준다.

``` yml
spring:
    redis:
        host: localhost
        port: 6379
```

Redis 연결을 위한 Configuration Bean을 생성한다.

``` java
@Configuration
@EnableRedisRepositories
public class RedisConfiguration {

    @Value("${spring.redis.host}")
    private String redisHost;

    @Value("${spring.redis.port}")
    private int redisPort;

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory(redisHost, redisPort);
    }

    @Bean
    public RedisTemplate<?, ?> redisTemplate() {
        RedisTemplate<byte[], byte[]> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        return redisTemplate;
    }
}
```

이제 레디스에 데이터를 저장해보자
공통 코드 객체를 저장하려고 한다. 객체는 Serializable 해야 함

``` java
@Getter
@ToString
@RedisHash("CommonCode")
public class CommonCode implements Serializable {

    @Id
    private String id;
    private String message;
    private LocalDateTime updateTime;

    @Builder
    public CommonCode(String id, String message, LocalDateTime updateTime){
        this.id = id;
        this.message = message;
        this.updateTime = updateTime;
    }

    public void refresh(String message, LocalDateTime updateTime){
        if(this.updateTime.isBefore(updateTime)){
            this.message = message;
            this.updateTime = updateTime;
        }
    }
}
```

``` java
public interface CommonCodeRedisRepository extends CrudRepository<CommonCode, String> {
}
```

``` java
@RunWith(SpringRunner.class)
@SpringBootTest
public class CommonCodeRedisTest {

    @Autowired
    private CommonCodeRedisRepository commonCodeRedisRepository;

    @Test
    public void createCommonCodeOnRedis(){
        String id = "nike";
        String message = "jordan";
        CommonCode commonCode = CommonCode.builder()
                .id(id)
                .message(message)
                .updateTime(LocalDateTime.now())
                .build();

        this.commonCodeRedisRepository.save(commonCode);

        Optional<CommonCode> optionalCommonCode = this.commonCodeRedisRepository.findById(id);
        if(!optionalCommonCode.isPresent()) throw new NoSuchElementException();
        final CommonCode savedCommonCode = optionalCommonCode.get();

        Assert.assertEquals(savedCommonCode.getId(), id);
        Assert.assertEquals(savedCommonCode.getMessage(), message);
        System.out.println(savedCommonCode.toString());
    }
}
```

이후 redis에서 `KEYS *`로 어떤 키들이 생성 됐는지 확인



https://riptutorial.com/ko/redis
https://jojoldu.tistory.com/297
https://stackoverflow.com/questions/37953019/wrongtype-operation-against-a-key-holding-the-wrong-kind-of-value-php
https://www.daleseo.com/docker-networks/

``` sh
docker network create [network]
docker network connect [network] [container]
docker network disconnect bridge container
docker network inspect [network]
docker build --network [network]
```

