# Referential Integrity
참조 무결성에 대해서 알아보기 전에 ACID 중 C, Consistency에 대해서 간단하게 설명하자면 다음과 같다.  
Consistency: 트랜잭션이 수행된 후 데이터베이스가 일관된 상태를 유지해야 한다. 트랜잭션이 시작되기 전과 후에 DB가 반드시 정의된 모든 규칙과 제약을 완료해야 한다.  
Consistency는 Consistency in Data와 Consistency in Reads로 구분할 수 있는데, Consistency in Data는 참조 무결성에 의해 강제된다.  
      
잠시 MySQL 공식 문서를 살펴보자.  
* [MySQL FOREIGN KEY Constraints](https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html)  
MySQL은 테이블 간에 관련 데이터를 상호 참조할 수 있는 외래 키와 관련 `데이터의 일관성`을 유지하는 데 도움이 되는 외래 키 제약 조건을 지원한다.  
외래 키 관계에는 초기 열 값을 보유하는 부모 테이블과 부모 열 값을 참조하는 열 값을 가진 자식 테이블이 포함된다. 외래 키 제약 조건은 **자식 테이블**에 정의된다.  
  
외래 키 제약은 참조 무결성을 구현하는 수단으로 볼 수 있다.  
  
**참조 무결성**은 데이터베이스에서 데이터의 일관성을 보장하기 위한 규칙으로, 데이터베이스의 두 테이블 간의 관계를 유지하고 한 테이블의 외래 키가 다른 테이블의 기본 키와 일치하도록 보장한다. 참조 무결성이 깨지면 데이터베이스의 일관성이 손상될 수 있다.  
**외래 키 제약**은 참조 무결성을 위해 데이터베이스에서 사용하는 메커니즘이다. 외래 키는 한 테이블의 열이 다른 테이블의 기본 키와 연결될 때 사용된다.  
  
MySQL 공식 문서에 사용한 예제를 살펴보자.  
```SQL
CREATE TABLE PARENT(
    ID INT NOT NULL,
    PRIMARY KEY (id)
) ENGINE = INNODB;

CREATE TABLE CHILD(
    ID INT NOT NULL,
    parent_id INT,
    INDEX par_ind (parent_id),
    FOREIGN KEY (parent_id) REFERENCES PARENT(id) ON DELETE CASCADE
) ENGINE = INNODB;
```
외래 키 제약 조건을 자식 테이블에 정의한 것을 알 수 있다.  
CASECADE 옵션은 상위 테이블에서 행을 삭제 또는 업데이트하면 하위 테이블에서 일치하는 행이 자동으로 삭제 또는 업데이트된다. ON DELETE CASCADE와 ON UPDATE CASCADE가 모두 지원된다.