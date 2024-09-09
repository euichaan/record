# ACID

# 트랜잭션이란

---

하나의 작업 단위로 취급되는 쿼리의 모음. 하나의 작업 단위는 논리적으로 분할할 수 없음을 의미한다.

Transcation Lifespan 은 다음과 같다.

- Transaction BEGIN
- Transaction COMMIT or ROLLBACK

transaction integrity(트랜잭션 무결성)은 트랜잭션이 수행될 때, 데이터의 일관성과 신뢰성을 보장하는 원칙과 매커니즘을 의미한다. 이는 주로 ACID 원칙을 통해 설명된다.

# Atomicity

---

- 트랜잭션 내의 모든 쿼리는 성공해야 한다.
- 하나의 쿼리라도 실패하면, 이전의 성공적으로 수행되는 모든 쿼리들은 롤백되어야 한다.
- 만약 데이터베이스가 트랜잭션 커밋 이전에 다운되었다면, 트랜잭션 내에서 성공적으로 실행된 모든 쿼리들은 롤백되어야 한다.
- 데이터베이스가 재시작 된 후 이것을 정리해야 한다.

트랜잭션은 하나의 작업 단위이며, 나눌 수 없다.

# Isolation

---

[https://en.wikipedia.org/wiki/Isolation_(database_systems)](https://en.wikipedia.org/wiki/Isolation_(database_systems))

이 글에 정리가 잘 되어있다.

- 진행 중인 트랜잭션이 다른 트랜잭션에 의해 수행된 변경사항을 볼 수 있을까?
- 다른 사용자 및 시스템에 트랜잭션 무결성이 어떻게 보이는지 결정한다.
- Read Phenomena
- Isolation Levels

격리 수준이 낮을수록 많은 사용자가 동시에 동일한 데이터에 액세스 할 수 있는 능력이 높아지지만 사용자가 겪을 수 있는 동시성 효과(Dirty Reads, Lost Updates) 수도 늘어난다. 반대로 격리 수준이 높을수록 사용자가 겪을 수 있는 동시성 효과 유형은 줄어들지만 시스템 리소스가 더 많이 필요하고 한 트랜잭션이 다른 트랜잭션을 차단할 가능성이 높아진다.

읽기 현상(Read phenomena)

- Dirty Reads: 어떤 트랜잭션에서 처리한 작업이 완료되지 않았음에도 다른 트랜잭션에서 볼 수 있는 현상
    - Dirty Reads는 READ UNCOMMITED 격리 수준에서만 발생한다.
    - 트랜잭션이 한 행을 두 번 검색하고 그 행이 그 사이에 커밋되지 않은 다른 트랜잭션에 의해 업데이트되는 경우 발생한다.
- Non-Repetable Reads: 하나의 트랜잭션에서 같은 쿼리를 실행했을 때 같은 결과를 가져오지 않는 현상
    - 트랜잭션이 한 행을 두 번 검색하고 그 행이 그 사이에 커밋된 다른 트랜잭션에 의해 업데이트되는 경우 발생한다.
- Phantom Reads: 다른 트랜잭션에서 수행한 변경 작업에 의해 레코드가 보였다 안 보였다 하는 현상
    - 트랜잭션이 한 행을 두 번 검색하고 그 행이 그 사이에 다른 트랜잭션이 커밋하여 해당 집합에 새로운 행이 삽입되거나 제거될 때 발생한다.

Non-repeatable Read와 Phantom Read를 방지하는 두 가지 전략

- Lock-based Concurrency Control, 잠금 기반 동시성 제어
- MVCC

잠금 기반 동시성 제어에서는 SELECT를 수행할 때 읽기 잠금이 획득되지 않거나, SELECT 가 수행된 후에 영향받은 행에 대한 잠금이 즉시 해제될 때 non-repeatable read와 phantom read가 발생할 수 있다.

Mysql,  MariaDB 에서는 MVCC를 사용.

MVCC 에서는 커밋 충돌로 영향을 받은 트랜잭션이 롤백되어야 한다는 요구사항이 완화될 때 non-repeatable read와 phantom read가 발생할 수 있다.

Isolation Level

여러 트랜잭션이 동시에 처리될 때 특정 트랜잭션이 다른 트랜잭션에서 변경하거나 조회하는 데이터를 볼 수 있도록 할 지 말지 결정하는 것.

- Read uncommitted
- Read committed
- Repeatable Read
- Serializable

Isolation: 각 트랜잭션 간의 독립성을 보장하는 속성. 이 속성은 여러 트랜잭션이 동시에 실행될 때, 각 트랜잭션이 서로 간섭하지 않고 독립적으로 수행되도록 보장한다. 즉, 한 트랜잭션에서 처리 중인 데이터가 다른 트랜잭션에 영향을 미치지 않도록 한다.

# Consistency

---

트랜잭션이 수행된 후 데이터베이스가 일관된 상태를 유지해야 한다.

트랜잭션이 시작되기 전과 후에 DB가 반드시 정의된 모든 규칙과 제약을 만족해야 한다.

- Consistency in Data
    - 참조 무결성에 의해 강제된다.
    - 참조 무결성은 데이터베이스에서 데이터의 일관성을 보장하기 위한 규약으로, 데이터베이스의 두 테이블 간의 관계를 유지하고 한 테이블의 외래 키가 다른 테이블의 기본 키와 일치하도록 보장한다. 참조 무결성이 깨지면 데이터베이스의 일관성이 손상될 수 있다.
- Consistency in reads
    - 값이 업데이트되고 커밋된 후에 값을 읽었을 때 이전 버전을 읽는다면, 불일치(inconsistency)이다.
    - 이는 궁극적 일관성(Eventual consistency)을 통해서 해결할 수 있다.

# Durability

---

커밋된 트랜잭션으로 인한 변경은 지속성이 있는 비휘발성 저장소에 저장되어야 한다.

Durability techniques

- WAL - Write ahead log (MySQL에서의 Redo Log)
- Asynchronous snapshot (비동기 snapshot)
- AOF(Append Only File)

### WAL

데이터를 디스크에 직접 쓰기 전에, 먼저 변경 사항을 로그에 기록한다.

이 로그는 나중에 변경 사항을 적용하거나 복구할 때 사용된다.

WAL에 변경 사항을 기록한 후에는, 나중에 적절한 시점에 데이터베이스의 실제 데이터 파일에 그 변경 사항을 반영한다. 이 과정은 보통 백그라운드에서 비동기적으로 처리되며, 데이터 페이지가 주기적으로 디스크에 기록된다.

데이터베이스는 다수의 쓰기 작업을 하나의 디스크 I/O로 묶어 처리할 수 있기 때문에, WAL을 통해 성능을 크게 향상시킬 수 있다.

# 추가적으로 공부하면 좋은 내용

---

- 2PL(Two-Phase Locking)
- 2PC(Two-Phase Commit)
- Lock-based Concurrency Control
- MVCC
- Database Implementation of Isolation
- 궁극적 일관성(Eventual consistency)
- MySQL 리두(Redo) 로그
- AOF