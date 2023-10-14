---
title: "SERIALIZABLE과 autocommit"
date: 2021-11-21
categories:
  - Database
tags:
  - MySQL
  - MySQL 8.0 Reference Manual
toc: true
toc_sticky: true
#published: false
#use_math: true
---
> MySQL 8.0: Transaction Isolation levels

{% include_relative series/MySQL_8.0_Reference_Manual.md %}

앞에서 Transaction isolation level 중 `REPEATABLE READ`와 `READ COMMITTED`, `READ UNCOMMITTED`에 대해 알아보았다.  
이번 글에서는 마지막으로 남은 `SERIALIZABLE`에 대해 알아보자.

<br>

# SERIALIZABLE

SERIALIZABLE 격리 수준은 트랜잭션 간의 일관성과 동시성을 최대한 보장하되, 이로 인해 성능에 부하가 생길 수 있다.

이 격리 수준은 REPEATABLE READ와 유사하다.  
한 트랜잭션 내에서 같은 데이터를 여러 번 읽을 때 항상 동일한 결과를 반환하는 것을 보장한다. (**일관성 보장**)

## autocommit이 비활성화된 경우
InnoDB는 모든 일반 `SELECT` 문을 `SELECT ... FOR SHARE`로 암시적으로 변환한다.  
이는 선택한 행에 `Shared Lock`을 걸어 **다른 트랜잭션이 해당 행을 수정하지 못하게 한다**.

- Shared Lock
  - 다른 트랜잭션이 잠긴 객체를 읽을 수 있고 다른 Shared Lock을 획득할 수 있지만, 
  - 쓰기는 할 수 없도록 하는 Lock이다. 
  - Exclusive Lock의 반대 개념.

## autocommit이 활성화된 경우
각 `SELECT` 문은 자체 트랜잭션으로 취급된다.  
따라서, `SELECT` 문은 읽기 전용(`Read only`)이다.  

`SELECT`문이 `Consistent (nonlocking) read`로 수행되는 경우,
직렬화(`Serialization`)할 수 있으며, 다른 트랜잭션에 대해 차단(Block)할 필요가 없다.

- [Consistent (nonlocking) read]({{ site.url }}{{ site.baseurl }}/database/Consistent-read와-Isolation-level)
  - 트랜잭션 격리 수준에 따라 다른 트랜잭션에 의해 변경되는 중간 상태의 데이터를 읽지 않고, 
  - 일관된 상태의 데이터만을 읽는다는 것을 의미한다. 
  - `Nonlocking`은 이 작업이 다른 트랜잭션에 Lock을 걸지 않는다는 것을 의미한다.

- SELECT 문의 직렬화(Serialization)
  - SELECT 문이 다른 트랜잭션과의 **동시 실행 없이 순차적으로 실행**될 수 있음을 의미한다. 
  - 다시 말해, 이 SELECT 문은 다른 트랜잭션에 의한 데이터 변경의 영향을 받지 않으므로, 트랜잭션들이 순차적으로 일어나는 것처럼 시스템을 안전하게 유지할 수 있다.

<br>
<br>

# References

[https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html)  