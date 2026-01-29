# High-Performance SQL
# Join Types  
SQL에서 JOIN은 서로 다른 테이블의 컬럼들을 결합한 결과를 만들어 준다.  
  
- CROSS JOIN  
- INNER JOIN  
- OUTER JOIN  
- NATURAL JOIN  
- LATERAL JOIN  
  
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
  
## NATURAL JOIN




# Join Performances
```sql
select distinct
s.id, s.first_name, s.last_name
from student s
join student_grade sg
on sg.student_id = s.id
where sg.grade = 10
order by s.id
```
Hash Join, 5천 건의 레코드에 대해 4 ms 소요. Hash Join은 버킷