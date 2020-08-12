---
title: "Garbage Collection"
date: 2020-08-11 21:35:00 +0900
category: study
lastmod : 2020-08-11 21:35:00 +0900
sitemap :
  changefreq : weekly
  priority : 1.0
---

<br>

## 1. 가비지 컬랙션이 하는일

자바는 OS 메모리 영역에 직접적으로 접근하지 않는다. 대신 JVM을 통해 간접적으로 접근한다.  
**JVM이 Object가 필요해지지 않는 시점에서 알아서 메모리를 확보**한다.

---

## 2. Java의 메모리 구조

### Stack

+ Stack은 각 **스레드마다 하나씩 존재**한다.
+ 스레드가 시작될때 생성된다. 다른 스레드들은 각자의 스택외의 **타 스레드의 스택에 접근할 수 없다**.
+ Stack에는 Heap 영역에서 생성된 Object타입의 **데이터의 참조 값들이 할당**된다.
+ 또한 int, long, char 등, **원시타입(primitive type)**의 데이터를 **참조 값이 아닌 실제 값**이 할당된다.

### Heap

+ Heap은 다수의 스레드가 존재하더하도 **하나의 Heap영역만 존재**한다.
+ Heap에는 긴 생명주기를 가지는 데이들이 저장된다. **모든 Object 타입들은 Heap영역에 생성**된다.

---

## 3.가비지 컬랙션 구성

Heap 영역에 생성된 데이터는 Stack에서 참조 변수를 통해 참조한다.  
  
해당 Stack에서 참조 변수가 pop되어 사라져셔 Heap의 **생성된 Object를 참조하는 변수가 없을 때**, 해당 객체를 **Unreachable** 하다고 한다.
Garbage Collection, **GC의 대상은 Unreachable 객체의 메모리 할당을 정리하는 것**이다.  
  
Heap의 영역은 **Young Generation**와 **Old Generation**으로 나눌 수 있다.  
여기서 Young Generation은 세분화 하여 **Eden, Survival 0, Survival 1** 으로 나눌 수 있다.  
  
+ **Young Generation** 영역에는 새롭게 생성된 객체들이 **Allocation(할당)**된다.  
Young 영역의 많은 객체들이 금방 Unreachable 상태가 되고 사라진다. Young영역에서 객체가 사라지는 것을 **Minor GC**가 발생한다고 한다.  
  
+ **Old Generation** 영역에는 Young에서 살아남은 객체가 **Promotion(승격)**되어 복사된다.  
Old 영역에서 객체가 사라지는것을 **Major GC(or Full GC)**가 발생한다고 한다.  
  
---

## 4. 가비지 컬랙션 과정

1. 새로운 오브젝트는 **Eden영역**에 할당, Survial 0(S0)과 Survial 1(S1)는 비워져있는 상태에서 시작한다.
2. Eden 영역이 가득차면, **Minor GC**가 발생한다.
3. Minor GC가 발생하면, Eden 영역의 **Reachable한 오브젝트들은 S0영역으로** 옮겨진다. Unreachable 오브젝트는 Eden영역의 클리어와 함께 사라진다.
4. 다시 Eden 영역이 가득차서, Minor GC가 발생하면 Eden영역의 Reachable 오브잭트들은 **계속해서 S0영역으로** 옮겨진다.
5. S0영역이 가득차면, **S0의 Reachable한 오브잭트들을 S1으로** 옮긴다. 이후 **Eden과 S0를 클리어** 한다.
6. **Surivor Space에서 Survior Space로(ex. S0 -> S1)으로 이동** 할 때, 살아남은 오브잭트의 **age 값이 증가**된다.
7. 2-6번의 과정이 반복된다(Survior Space의 From-To가 바뀌면서). 반복하면 **특정 오브젝트들의 age 값이 특정 값 이상**이되면 Old Generation으로 **Promotion(승격하여 옮겨짐)**하게 된다.
8. Minor GC의 반복, Promotion의 반복을 통해 Old Generation이 가득차면 **Major GC**가 발생한다.

---

## 5. Garbage Collector의 종류

### Serial GC (-XX:+UseSerialGC)

+ Serial GC는 Java 5 & 6의 기본 가비지 컬랙터다. Minor GC와 Major GC를 모두 순차적으로 시행이 된다.
+ Serial GC는 Old 영역의 GC를 위해 mark-sweep-compact 알고리즘을 사용한다.
+ mark-sweep-compact
  1. Old 영역에 살아있는 객체를 식별(mark)한다.
  2. Heap의 앞에서 부터 살아 있는 것만 남긴다(sweep).
  3. 살아남은 객체들을 힙에 가장 앞에서 부터 채워, 객체가 존재하는 부분과 없는 부분으로 나눈다(Compact).
+ Serial GC는 코어가 하나만 있을때 사용하기 위해서 만든 방식이기 때문에, 운영서버에서 사용하면 성능이 떨어진다. 적은 메모리와 CPU 코어 갯수일 때 적합하다.

### Parallel GC (-XX:+UseParallelGC)

+ Parallel GC는 기본 알고리즘은 Serial GC와 동일하지만, Parallel GC는 Young 영역에 대한 GC를 멀티스레드로 수행한다.
+ Parallel GC 옵션을 설정하였더라도, CPU의 코어가 하나라면 Serial GC가 Default로 사용된다.
+ 또한 `-XX:+UseParallelOldGC` 옵션으로 Old 영역의 GC도 멀티스레딩을 사용할 수 있다.(대신 Mark-Summary-Compaction 알고리즘으로 수행)

### Concurrent Mark Sweep (CMS) GC (-XX:+UseConcMarkSweepGC)

+ CMS GC는 대부분의 **가비지 컬랙션 작업을 애플리케이션 스레드와 동시에 수행**, stop-the-world 시간을 최소화한다.
+ CMS GC의 과정
  1. **Initial Mark**를 통해 **클래스 로더**에서 **가장 가까운 살아있는 객체** 만을 찾는다. 이때 발생하는 stop-the-world는 매우 짧다.
  2. **Concurrent Mark**단계에서 Initial Mark에서 **살아있던 객체에서 참조하는 객체들을 따라가며 확인**한다. Concurrent Mark는 **다른 스레드가 실행 중인 상태**에서 동시에 진행된다.
  3. 그리고 **Remark** 단계에서 Concurrent Mark단계에서 새로 추가되거나 참조가 끊긴 객체를 확인한다(짧은 stop-the-world 발생).
  4. 마지막으로 **Concurrent Sweep**을 통해 사용하지 않는 객체를 정리한다. Concurrent Sweep **다른 스레드가 실행되는 상황**에서 동시에 진행된다.

+ CMS GC는 **stop-the-world의 시간이 짧아**, 애플리케이션의 응답 속도가 빨라진다. 따라서 **Low Latency GC**라고도 부른다.
+ 하지만 많은 메모리와 CPU 사용을 하며, **Compaction을 하지 않아 메모리 파편화**가 발생한다.
+ 따라서 메모리 파편화를 해결하기 위한 stop-the-world 시간이 길기 때문에 Compaction 작업이 얼마나 자주, 오랫동안 수행 되는지 확인하여 사용해야한다.

### G1 Garbage Collector

+ G1 GC는 Java 7부터 사용이 가능하고, **Java 9부터는 Default**로 설정되어있다.
+ 기존의 GC와는 다르게 영역을 **Young과 Old로 나누지 않는다**.
+ G1 GC에서 **Heap을 일정한 크기의 바둑 판 처럼 지역을 나눈다**. 그리고 각 지역은 **Eden Space, Survivor Space 그리고 Old Generation**로 나뉘며 할당된다.
**각 세트의 크기(지역의 갯수)는 유연하게 할당**되며, 역할은 이전 GC들과 동일하다.
+ G1 GC의 GC 수행 과정은 CMS와 비슷하다.
  1. **Concurrent** 하게 사용하지 않는 오브잭트들을 **마킹**한다.
  2. G1 GC는 메모리가 **가장 많이 비어있는 지역**부터 **회수**를 시작하는데, 이 과정을 **Garbage-First**라고 부른다.
  3. G1 GC는 **evacuation(배출)**을 통해 메모리를 비운다. Evacuation 중, 하나 이상의 오브잭트들은 **하나의 지역으로 복사되어 옮겨진다**.
  즉, **메모리를 비우는 작업과 Compact를 동시에** 한다.
  4. evacuation는 Concurrent하게 다른 메소드의 처리와 동시에 처리되기 때문에, stop-the-world시간이 줄어든다.

---

### References

[https://d2.naver.com/helloworld/1329](https://d2.naver.com/helloworld/1329)  
[https://yaboong.github.io/java/2018/06/09/java-garbage-collection/](https://yaboong.github.io/java/2018/06/09/java-garbage-collection/)  
[https://www.oracle.com/technetwork/tutorials/tutorials-1876574.html](https://www.oracle.com/technetwork/tutorials/tutorials-1876574.html)  
