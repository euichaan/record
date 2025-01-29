# Lock
Database의 Lock을 의미하며, MySQL InnoDB 스토리지 엔진 기준으로 설명한다.  

## Shared and Exclusive Locks  
InnoDB `shared lock(S)` 과 `exclusive lock(X)` 의 두 가지 잠금 유형이 있는 표쥰 row-level 잠금을 구현한다.  
- S lock은 잠금을 보유한 트랜잭션이 행을 읽을 수 있도록 허용한다.  
- X lock은 잠금을 보유한 트랜잭션이 행을 업데이트하거나 삭제할 수 있도록 허용한다.  
  
트랜잭션 T1이 행 r에 대한 S lock을 보유하고 있는 경우, 일부 개별 트랜잭션 T2의 행 r에 대한 잠금(lock) 요청은 다음과 같이 처리된다.  
- T2의 S lock 요청은 즉시 승인될 수 있다. 결과적으로 T1과 T2는 모두 r에 대해 S lock을 보유한다.  
- T2의 X lock 요청은 즉시 승인될 수 없다.  
  
트랜잭션 T1이 행 r에 대해 X lock을 보유하고 있는 경우, 다른 트랜잭션 T2가 행 r에 대해 두 가지 유형의 잠금(S, X)을 요청해도 즉시 승인할 수 없다.  
대신 트랜잭션 T2는 트랜잭션 T1이 행 r에 대한 잠금을 해제할 때까지 기다려야 한다.  
  
## Intention Locks
intention lock은 테이블 수준 잠금으로, 트랜잭션이 나중에 테이블의 행에 대해 어떤 유형의 잠금(공유 또는 독점)을 필요로 하는지를 나타낸다. intention lock에는 두 가지 유형이 있다.  
- IS (intention shared lock)는 트랜잭션이 테이블의 개별 행에 S lock을 설정하려고 함을 나타낸다.  
- IX (intention exclusive lock)는 트랜잭션이 테이블의 개별 행에 X lock을 설정하려고 함을 나타낸다.  
  
예를 들어, `SELECT ... FOR SHARE`는 IS lock을 설정하고, `SELECT ... FOR UPDATE는 IX lock을 설정한다.  
intention lock 프로토콜은 다음과 같다.  
- 트랜잭션이 테이블의 행에 대한 S lock을 획득하려면 먼저 테이블에서 IS 잠금 이상을 획득해야 한다.  
- 트랜잭션이 테이블의 행에 대한 X lock을 획득하려면 먼저 테이블에서 IX 잠금을 획득해야 한다.  
  
intention lock은 `LOCK TABLES ... WRITE`와 같은 전체 테이블 요청을 제외하고는 어떤 것도 차단하지 않는다.  
의도적 잠금의 주요 목적은 특정 테이블에서 누군가가 행을 잠그고 있거나, 잠글 예정임을 나타낸다.  
  
출처: MySQL 8.4 Reference Manual