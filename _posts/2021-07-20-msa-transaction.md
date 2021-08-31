---
title: "MSA에서의 트랜잭션"
date: 2021-07-20 17:00:00 +0900
category: study
lastmod : 2021-07-20 17:00:00 +0900
sitemap :
  changefreq : weekly
  priority : 1.0
---

MSA 환경에서의 트랜잭션, 특히 atomicity(원자성)은 항상 문제가 된다.  

Monolithic 환경에서는 간단하게 모두 commit 혹은 rollback을 하면 되지만,
MSA 환경에서는 각 서비스들은 서로 DB가 다를 수 있기 때문에, 한 트랜잭션에 묶어 진행을 할 수 없다.
따라서 모든 서비스에 commit 혹은 rollback을 보장할 수 없다.  

하지만 불일치 상태로 데이터를 남길 수는 없기 때문에, `2 Phase commit protocol`또는 `Sagas pattern`을 통해 이를 보완하려고 한다.

## 2 Phase Commit Protocol

2 Phase commit Protocol에서는 `Commit requestig phase` 와 `Commit phase`로 커밋 단계를 두 단계로 나눠 진행을 한다.  

- `Commit requestig phase` 이후, 데이터 상태는 `prepare`로 바뀌며,
- `Commit phase`이후 `commit` 상태로 바뀐다.

2 Phase Protocol은 `Participanats`와 `Transaction Coordinator`로 구성되어 두 단계의 커밋을 진행한다.

- `Participanats` - 트랜잭션에 참여하는 서비스들(ex. 서버 or DB)
- `Transaction Coordinator` - 각 participant들을 한 트랜잭션으로 동작하는 것처럼 관리하는 객체
  
### 2 Phase Commit Protocol 알고리즘

![2-phase-commit](https://drive.google.com/uc?id=1QXqHoEb4yE_PmbpciL_69FdBL4UlwECY)

1. 클라이언트는 Coordinator에게 데이터 갱신 요청을 한다.
2. Coordinator는 모든 Participanats에게 `Commit requestig`을 요청한다.
3. Participanats는 `prepare`로 데이터 상태를 변경 후, Coordinator에게 `OK` 응답을 한다.
4. 모든 Participanats에게 `OK` 응답이 오면, Coordinator는 모든 Participanats에게 `Commit`을 요청한다.
5. Participanats는 `commit`로 데이터 상태를 변경 후, Coordinator에게 `OK` 응답을 한다.
6. 2-5 도중 Participanats가 `OK` 응답을 하지 않는 다면, Coordinator는 모든 Participanats에게 `rollback` 요청을 한다.

### 2 Phase Commit 딘점

- **느리다**: 모든 Participanats에게 응답을 받고 진행 해야 하기 때문에 커밋 과정이 느리다.
- **종속적이다**: Participanats에게 매우 종속적이다. 만약 느린 Participanats 서비스가 존재 한다면 모든 서비스가 느려진다. 또한 하나의 서비스에 의해 모든 롤백이 이뤄 진다.
- **병목현상이 발생할 수 있다**: Transaction Coordinator가 모든 응답을 받아 처리하기 때문에, Coordinator이 모든 응답을 처리 하기 전까지 다른 서비스들은 block 된 상태가 될 수 있으며, 이는 병목 현상을 발생시킨다.

## Saga Pattern

Saga는 연속적으로 있는 `local transaction`들의 순서이다.

- `local transaction`- 각 서비스마다 진행하는 트랜잭션

각 `local transaction`은 데이터를 갱신하고, 메시지 혹은 이벤트를 발행하여 다음 `local transaction`이 잰행 될 수 있게 한다.  

만약 `local transaction`이 실패 한다면, saga는 진행된 트랜잭션을 rollback하는 보상 트랙잭션들을 연속적으로 진행한다.

### Saga Pattern 종류

![2-phase-commit](https://drive.google.com/uc?id=18oVfqG9h_bRXoj2P3JThe5p__wYIfpZD)

- **Choreography** - 각 local transction은 다른 서비스의 local transaction을 작동 시키는 이벤트을 발행한다.

![2-phase-commit](https://drive.google.com/uc?id=1j51JgW-SOeSF5l5FoxpVACOP3xdrvS_k)

- **Orchestration** - orchestrator이 각 서비스들의 local transaction을 실행 하도록 관리한다.

### References

- [https://medium.com/geekculture/distributed-transactions-two-phase-commit-c82752d69324](https://medium.com/geekculture/distributed-transactions-two-phase-commit-c82752d69324)
- [https://microservices.io/patterns/data/saga.html](https://microservices.io/patterns/data/saga.html)
- [https://www.popit.kr/rest-%EA%B8%B0%EB%B0%98%EC%9D%98-%EA%B0%84%EB%8B%A8%ED%95%9C-%EB%B6%84%EC%82%B0-%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98-%EA%B5%AC%ED%98%84-1%ED%8E%B8/](https://www.popit.kr/rest-%EA%B8%B0%EB%B0%98%EC%9D%98-%EA%B0%84%EB%8B%A8%ED%95%9C-%EB%B6%84%EC%82%B0-%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98-%EA%B5%AC%ED%98%84-1%ED%8E%B8/)
