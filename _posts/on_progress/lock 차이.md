## SELECT FOR UPDATE와 LOCK IN SHARE MODE

- 먼저 lock을 획득한 세션의 로우들이 update 쿼리후 commit 될기 전까지 다른 세션들은 수정하지 못함
- 다른 세션들 LOCK WAIT TIME 동안 대기
- FOR UPDATE: 대상 레코드가 다른 트랜잭션의 읽기나 쓰기 잠금 사용 중이면 대기
- LOCK IN SHARE MODE: 대상 레코드에 쓰기 잠금이 걸려 있으면 대기, 읽기 잠금은 통과
- LOCK IN SHARE MODE에서 쓰기 락을 획득하려면 데드락 유발

## Lock의 적용 요소에 따른 종류

- Shared Lock : read lock, shared lock이 걸려 있는 동안 해당 row에 exclusive lock 획득 불가능
- Exclusive Lock: writed lock, 걸려 있으면 read & write 둘 다 획득 불가능
- Intention Lock : table-level lock으로, 해당 row에 락을 걸 거라고 미리 의도하는 것

## Lock이 적용되는 상황에 따른 종류

- Record Lock: 해당 레코드에만 락을 거는것
- Gap Lock: 범위(Gap)에 락을 거는 것
- next-key Lcok: record와 gap을 복합 사용

## NOWAIT과 SKIP LOCKED

- skip locked 사용시 락이 걸린 로우를 제외한 로우들을 가져옴
- no wait: 해당 로우에 락이 걸려있으면 기다리 않고 바로 실패 처리

## DB 설계 기준

- 상품 가입을 하면 내역이 하나 쌓이는 구조
- 상품 테이블에 상태 및 카운트를 표시하면, 정합성을 맞춰야 하기 때문에 뺌

## HTTP 응답 기준

- 요청 값이 잘못 되거나 투자 할 수 없는 상품(sold-out)에 요청 했을 경우 400에러를 줌. 잘못 된 상품을 요청 했기 때문에 bad request라 생각
- 이미 투자한 상품일 경우, 중복으로 인한 409 conflict
- 해당 리소스가 없으면 404
- 나머지 에러가 날 경우 500 서버 에러

## DTO와 Domain 차이점

각 Layer 간 이동에는 DTO로 구현하고, DB 엔티티는 Domain으로 차이를 둠

## Stream에 대해, 장점, 단점

- 배열 또는 컬랙션에 함수 여러개를 조합하여 원하는 결과를 필터링하고 가공된 결과를 얻을 수 있게함
- 병렬 처리 가능

## Mapper와 DAO

- Mapper : 마이바티스 매핑 xml에 기재된 SQL을 호출하기 위한 인터페이스, 3.0부터 지원
- SqlSession 등록 x, dao 구현 x, 네임스페이스 + SQL ID 호출 x, 그외 메소드 사용 x

## Asterisk(*)

값을 되돌려 줄 때 테이블의 모든 컬럼을 반환 비효율적
SQL 파서는 Data dictionary에서 해당 테이블에 대한 모든 컬럼을 읽어 대체

