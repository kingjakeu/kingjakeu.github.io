자바에서 쓰레드는 운영체제의 자원인 시스템-레벨 쓰레드로 설정된다
병렬 처리(pallelism)을 위해 쓰레드간의 context switching도 운영체제에 의해 됌, 따라서 쓰레드가 마구잡이로 생겨 버리면 자원을 잡아먹음
Pallelism은 - 

많은 쓰레드가 생성될 수록 각 쓰레드가 실제 작동하는 시간이 줄어듬
쓰레드 풀 패턴은 멀티 쓰레드 어플리케이션에서 리소스를 아끼고 제한적인 리소스를 유지하며 병렬처리를 할수 있음
쓰레드 풀을 사용할때, 동시성의 코드를 병렬적 테스크의 형태로 쓰고, 스레드풀에 실행을 등록
쓰레드풀은 이 작업을 실행 할때 여러 재사용 쓰레드들을 관리

이 패턴은 어플리케이션이 생성하는 쓰레드의 갯수와 쓰레드의 라이프 사이클을 관리함

Task summitter로 부터 오는 Task들을 task Queue에 저장하고 
Task queue에서 부터 pop되는 task를 쓰레드 풀로 보냄

Executors Executor and ExecutorService
Executors helper class는 미리 설정된 쓰레드 풀 인스턴스의 생성을 위한 여러 메소드들을 가지고 있음

Executor와 ExecutorService 인터페이스들은 기존과 다른 쓰레드풀을 implementation하는데에 사용됌
실제 쓰레드 풀 implementation과 커스텀 코드들을 분리해서 유지 해야 한다.

Executor 인터페이스는 Runnable 인스터스 실행을 위해 summit하는 하나의 execute 메소드를 가지고 있다.

ExecutorService 인터페이스는 테스크 진행을 관리하고 서비스 종료를 관리하기 위한 많은 숫자의 메소드들을 가지고 있음
이 인터페이스를 사용하여, 테스크들의 실행을 submit하고, return된 Future인스턴스를 사용하여 execution을 관리 할 수 있음


ThreadPoolExecutor(AbstractExecutorService extend -> ExecutorService의 implementation)

ThreadPoolExecutor의 주요 설정 파라미터는 corePoolSize, maximunPoolSize, keepAliveTime

Pool은 고정된 숫자의 코어 스레드들을 계속 유지함.
그리고 추가적인 쓰레드들이 생성될 수 있고, 필요하지 않으면 추가된 쓰레드들은 종료됌.

corePoolSize는 코어 스레드의 갯수
만약 모든 코어 쓰레드들이 작업 중이고, 내부 큐가 가득 찼다면, maximumPoolSize까지 풀이 커짐

keepAliveTime은 추가적인 스레드들이 놀고 있는 상태로 얼마나 존재할 수 있는지 정의
allowCoreThreadTimeOut(true) 메소드들을 통해 코어 메소드 또한 같은 규칙으로 지울 수 있음
하지만 기본은 non-core 쓰레드만 없애는 거임

일반적인 설정은 Executors에 스태틱 메소드들로 미리 정의되어 있음
ex. newFixedThreadPool -> corePoolSize == maximunPoolSize
ex. newSingleThreadExecutor() -> corePoolSize = 0, maximunPoolSize = INT.MAX, keepAliveTime = 60s 
 - SynchronousQueue를 사용함, SynchronousQueue는 항상 insert와 remove가 동시에 일어남, 따라서 항상 사이즈가 0
ex. newSingleThreadExecutor() -> corePoolSize == maximunPoolSize == 1, event-loop에 적합(하나 실행후 하나)

ThreadPoolExecutor은 불변 wrapper로 되있음, 따라서 생성후 재설정이 불 가능. 

ScheduledThreadPoolExecutor은 ThreadPoolExecutor 클래스의 extend로 
ScheduledExecutorService 인터페이스 또한 implementation하고 있음

https://www.baeldung.com/thread-pool-java-and-guava