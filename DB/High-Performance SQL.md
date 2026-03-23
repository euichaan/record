# High-Performance SQL
# Join Types  
SQL에서 JOIN은 서로 다른 테이블의 컬럼들을 결합한 결과를 만들어 준다. (Projection)  
  
- CROSS JOIN  
- INNER JOIN  
- LEFT JOIN  
  
## CROSS JOIN
cartesian product와 같은 결과.  
```sql
-- SQL:92
select
    ranks.name as rank,
    suits.symbol as suit
from
    ranks
cross join
    suits
order by
    ...

--- SQL:89 theta-style syntax
select
    ranks.name as rank,
    suits.symbol as suit
from
    ranks, suits
    ...
```  
## INNER JOIN  
inner join은 두 테이블을 조인하여 부모와 자식에 대한 프로젝션(열 단위로 원하는 데이터를 조회)을 생성할 수 있게 해준다.  
```sql
select 
    p.id as post_id,
    p.title as post_title,
    pc.review as review
from
    post p
inner join
    post_comment pc on pc.post_id = p.id
order by
    pc.id
```
필터링을 하면서 댓글이 있는 게시물만 join하도록 지정하고 있다. 일종의 필터링된 Cartesian 곱과 같다고 할 수 있다.  
on 절을 사용하여 관계를 정의하고 외래 키 컬럼과 기본 키 컬럼을 매칭하여 조인 결과 집합을 얻었다.  
  
on을 사용할 때와 using을 사용할 때 약간의 차이가 존재한다.  
select *를 할 때 on절은 양쪽 테이블 컬럼이 다 나오고, 조인 컬럼이 중복되지만 using은 조인에 사용된 컬럼은 한 번만 나온다.  
  
```text
NATURAL JOIN은 두 테이블에서 이름이 같은 컬럼을 자동으로 찾아서 그 컬럼을 기준으로 JOIN하는 방식이야.
SELECT *
FROM employees NATURAL JOIN departments;

SELECT *
FROM employees e JOIN departments d
  ON e.department_id = d.department_id;
두 테이블에 department_id라는 동일한 이름의 컬럼이 있으면, 별도로 ON 절을 쓰지 않아도 자동으로 그 컬럼 기준으로 조인된다.
실무에서 거의 안 쓰는 이유는 명확한데, 컬럼 이름에 전적으로 의존하기 때문이야. 예를 들어 두 테이블에 name이나 created_at 같은 공통 컬럼이 의도치 않게 존재하면 그것까지 JOIN 조건에 포함되어 결과가 완전히 달라진다. 테이블에 컬럼 하나 추가하는 것만으로 기존 쿼리가 깨질 수 있어서, JOIN 조건을 명시적으로 쓰는 INNER JOIN ... ON이 훨씬 안전하고 가독성도 좋다.
SQL 표준에는 포함되어 있고 MariaDB/MySQL에서도 지원하지만, 실무 코드나 JPA/QueryDSL에서는 쓸 일이 없다고 보면 돼.
```
# JOIN Performances
서브쿼리와 조인 중 무엇을 사용해야 할까?  
  
```sql
select distinct
s.id, s.first_name, s.last_name
from student s
inner join student_grade sg
on sg.student_id = s.id
where sg.grade = 10
order by s.id
```
이러한 유형의 쿼리가 자주 나오는 것은 일반적이다.  
Hash Join, 5천 건의 레코드에 대해 4 ms 소요. Hash Join은 버킷을 생성해야 하므로 시간이 걸린다. 마지막에 정렬을 수행하여 최종적으로 학생들만 얻는다.  
  
요구사항으로 다시 돌아가 보면 학생들만 필요하다.  
```sql
select 
s.id, s.first_name, s.last_name
from student s
where exists (
    select 1
    from student_grade sg
    where sg.student_id = s.id and sg.grade = 10
) order by id
```
이 방법은 Semi Join을 사용한다. 스캔한 row 수가 줄었는데, 10이라는 값을 얻으면 조인 실행을 건너뛰고 다음 사용자로 넘어갈 수 있기 때문이다.  
  
복합 프로젝션이 필요하다면 당연히 조인을 사용해야 한다. 하지만 필터링만 하고 싶고 한 테이블만 프로젝션 하려면 서브쿼리를 사용하는 것이 조인보다 효율적일 수 있다.  
  
# JOIN Algorithms
1. Nested Loops  
2. Hash Join  
3. Merge Join  

## Nested Loops 
Nested Loops를 자바 코드로 예시를 든다면 중첩 루프를 사용하는 것과 같다.  
```java
for (Student student : students) {
    for (StudentGrade studentGrade : studentGrades) {
        if (student.getId().equals(studentGrade.getStudentId())) { // on절
            tuples.add() ...
        }
    }
}
```
O(N2)으로 레코드가 많을 때 Nested Loop 알고리즘은 효율적이지 않을 것.  
데이터베이스가 실행 계획을 생성할 때 사용자가 원하는 것을 알고 있을 뿐 아니라 데이터베이스가 보유한 데이터도 알아야 한다. 이를 위해 통계를 사용한다.  
  
## Hash Join
Hash Join은 두 번의 단계로 나누어진다.  
1. 해시 맵이나 해시 테이블을 생성하는 과정. It creates a hash table from the records of the table that has fewer elements using the join attribute as key. (레코드 수가 더 적은 테이블을 기준으로, 조인 컬럼을 키로 해서 해시 테이블을 만든다.)  
2. 더 큰 테이블을 순회하면서 해시 구조를 사용해서 일치하는 레코드를 찾는다.(반복)  
  
더 많은 레코드를 가진 테이블을 선택하면 메모리 사용량이 증가하고, 모든 연결에서 이렇게 하면 데이터베이스의 메모리가 부족해질 수 있다.  
  
시간 복잡도는 O(N+M)  
  
## Merge Join
정렬된 테이블에서 작동한다. (조인 조건을 기준으로 정렬되어야 한다.)  
Merge Join은 조인 키 기준으로 정렬된 두 테이블을 앞에서부터 순차적으로 비교하며 병합하는 조인 방식이다.  
핵심 아이디어는 정렬된 두 배열을 merge 하는 과정과 거의 같다.  
  
두 테이블 R, S가 각각 조인 컬럼 기준으로 오름차순 정렬되어 있다고 가정하자.  
각 테이블의 현재 위치를 가리키는 포인터를 하나씩 두고, 두 포인터가 가리키는 조인 키를 비교하면서 진행한다.  
  
예를 들어 현재 값이:  
- R.key < S.key 이면  
R.key는 현재 S.key와 절대 매칭될 수 없으므로, R의 포인터를 다음 행으로 이동한다.  
- R.key > S.key 이면  
반대로 S의 포인터를 다음 행으로 이동한다.  
- R.key == S.key 이면  
조인 조건을 만족하므로 결과를 생성한다.  
  
이 과정을 한쪽 테이블이 끝날 때까지 반복한다.  
  
시간 복잡도는 O(Nlog(N) + Mlog(M))  
  
Q1. 서브쿼리가 조인보다 유리할 때  
**다른 테이블의 조건을 이용해 현재 테이블 레코드를 걸러내고 싶을 때**    
Q2. 조인 알고리즘 중 한 테이블(주로 작은 테이블)의 데이터를 이용하여 해시 맵을 구축하고, 다른 테이블을 스캔하며 해시 맵을 조회하여 일치하는 레코드를 찾는 방식  
해시 조인  
  
데이터베이스는 통계를 바탕으로 데이터 크기와 분포를 파악해 비용 기반 최적화로 가장 효율적인 실행 계획을 선택한다. 이는 옵티마이저의 핵심 기능이다.  
  
query example:  
```sql
select
    p.title as post_title,
    count(pc.*) as comment_count
from post p
inner join post_comment pc on p.id = pc.post_id
group by p.title
order by comment_count desc
limit 3
```
  
최적화 하려면, 조인을 하기 전에 group by를 하면 된다.  
```sql
select
    p.title as post_title,
    count(pc.*) as comment_count
from post p
join (
    select 
    post_id,
    count(*) as comment_count
    from post_comment
    group by post_id
    order by comment_count desc
    limit 3
) pc on p.id = pc.post_id
order by comment_count desc
```  
# SubQueries
## EXISTS
서브쿼리는 행을 반환하는지 여부를 판단하기 위해 실행됩니다.  
서브쿼리가 최소 한 행이라도 반환하면 EXISTS는 true로 평가됩니다. 반환되는 행이 없으면 false로 평가됩니다.  
서브쿼리는 바깥 쿼리의 변수를 참조할 수 있습니다.  
서브쿼리는 끝까지 완전히 실행되는 것이 아니라, 최소 한 행이 반환되는지 확인되는 시점까지만 실행됩니다.  
  
## NOT EXISTS
반환되는 행이 없으면 NOT EXISTS는 true로 평가됩니다. 반대로 서브쿼리가 최소 한 행이라도 반환하면 NOT EXISTS는 false로 평가됩니다.  

row_constructor IN (subquery)    
```
(id, admission_score) in (
    select student_id, avg(grade)
    from student_grade
    group by student_id
)
```
## IN, NOT IN
```sql
select id, first_name, last_name
from student
where id in (
    select 2
    union all
    select null 
)
order by id
```
in은 내부적으로 or로 풀리기 때문에 where id = 2 or id = null (unknown)이므로 2만 반환한다.  
  
```sql
select id, first_name, last_name
from student
where null in (
    select 2
    union all
    select null 
)
order by id
```
이 경우 where null = 2 or null = null로 아무런 값도 반환되지 않는다.  
```sql
select id, first_name, last_name
from student
where id not in (
    select 2
    union all
    select null 
)
order by id
```
not in을 쓸 때는 주의해야 하는데, not in은 and로 풀린다.  
따라서 where id <> 2 and id <> null 이므로 항상 빈 결과 집합이 반환된다.  
  
**not in을 사용하면서 null값이 포함되는 것은 상당히 위험하다.**  
  
## Column Expression
A subquery can be used in the select clause as a column expression  
the subquery must return 'a single scalar value'  
  
서브쿼리가 단일 값을 생성해야 한다.  
하나의 값을 반환하지 못하면 오류가 발생한다.  
  
각 행마다 스칼라 값을 계산하는 방식. 본질적으로 상관 서브쿼리.  
```sql
select id, first_name, last_name,
(
    select avg(sg.grade)
    from student_grade sg
    where sg.student_id = student.id
) as avg_grade
from student
order by id
limit 100
```
학생 100명 각각에 대해 서브쿼리가 한 번씩 실행.  
  
from 절의 인라인 뷰(파생 테이블)로 바꾸면 집계를 한 번만 수행한다.  
```sql
select s.id, s.first_name, s.last_name, sg.avg_grade
from student s
join (
    select student_id, avg(grade) as avg_grade
    from student_grade
    group by student_id
    order by student_id
    limit 100
)
as sg on s.id = sg.student_id
order by s.id
```
  
## ANY and ALL
왼쪽 값을 서브쿼리 결과의 각 행과 비교해서, 하나라도 조건을 만족하면 true.  
```sql
-- admission_score가 자기 성적 중 하나라도 보다 크면 포함
select id, fisrt_name, last_name, admission_score
from student
where admission_score > any (
    select grade from student_grade
    where student_grade.student_id = student.id
)
```
ANY의 경우 ANY는 OR로 풀리길 때문에 서브쿼리에 NULL이 섞여 있어도 NULL이 아닌 값에서 조건이 성립하면 true를 반환한다.  
예를 들어 score > ANY (8, NULL)은 score > 8이 true면 전체가 true다.  
    
```sql
-- admission_score가 자기 성적 전부보다 낮아야 포함
select id, first_name, last_name, admission_score
from student
where admission_score < all (
    select grade from student_grade
    where student_grade.student_id = student.id
)
```
ALL의 경우 AND로 풀리기 때문에 서브쿼리에 NULL이 하나라도 있으면 SCORE > ALL (8, NULL)은 score > NULL이 NULL로 평가되고  
AND를 하면 빈 결과를 반환하기 때문에 주의해야 한다.  
  
**NOT IN과 ALL 사용시 서브쿼리에 NULL이 포함되지 않도록 주의해야 한다.**  
방어하는 방법은 `where column is not null`을 넣거나 `not in` 대신 `not exists`를 쓰는 것이다.  
    
# TODO (study)
semi join