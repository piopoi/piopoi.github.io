---
title: "InnoDB Locking: Record Lock"
date: 2022-03-10
categories:
  - Database
tags:
  - mysql
---
> MySQL 8.0 Reference Manual로 공부하기

{% include_relative series/MySQL_8.0_Reference_Manual.md %}

# 레코드 락 (Record Lock)

Index Record에 대한 Lock으로, 
- 특정 행(Row)을 선택적으로 잠가(Lock) 다른 트랜잭션이 해당 행을 수정하거나 삭제하는 것을 방지하여 
- 특정 행에 대한 동시성 제어를 할 수 있게 해준다.  
<br>

예를 들어, 아래 쿼리는 t.c1의 값이 10인 행을 `INSERT`/`UPDATE`/`DELETE` 하는 다른 트랜잭션을 막는다.
```
SELECT c1 
FROM t 
WHERE c1 = 10 
FOR UPDATE;
```
<br>

## 인덱스가 없는 경우
레코드 락은 테이블에 **인덱스가 없더라도** 항상 인덱스 레코드를 잠근다.
- InnoDB 엔진은 인덱스가 명시적으로 생성되지 않았다면,
- **숨겨진 클러스터 인덱스(`Hidden Clustered Index`)**를 생성하고, 
- 이 인덱스를 사용하여 레코드 락을 처리한다.

<br>

## 레코드 락 상태 확인
`SHOW ENGINE INNODB STATUS` 명령어로 현재 InnoDB에서 어떤 레코드락이 걸려 있는지,
어떤 트랜잭션이 락을 잡고 있는지 등의 정보를 아래 예제처럼 확인할 수 있다.

```
RECORD LOCKS space id 58 page no 3 n bits 72 index `PRIMARY` of table `test`.`t`
trx id 10078 lock_mode X locks rec but not gap
Record lock, heap no 2 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
 0: len 4; hex 8000000a; asc     ;;
 1: len 6; hex 00000000274f; asc     'O;;
 2: len 7; hex b60000019d0110; asc        ;;
```

<br>
<br>

# References

[https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html)  