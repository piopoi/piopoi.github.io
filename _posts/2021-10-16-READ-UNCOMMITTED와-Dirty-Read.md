---
title: "READ UNCOMMITTED와 Dirty Read"
date: 2021-10-16
categories:
  - Database
tags:
  - mysql
---
> MySQL 8.0 Reference Manual로 공부하기

{% include_relative series/MySQL_8.0_Reference_Manual.md %}

앞에서 Transaction isolation level 중 `REPEATABLE READ`와 `READ COMMITTED`에 대해 알아보았다.  
이번 글에서는 `READ UNCOMMITTED`에 대해 알아보자.

<br>

# READ UNCOMMITTED

`SELECT` 문은 lock을 걸지 않는 방식(`Nonlocking fashion`)으로 수행된다.  

하지만, row의 earlier version이 사용될 수 있어 **Read에 일관성이 없다**. -> `not consistent`  
이것을 `Dirty Read`라고 한다.

그 외에는 READ COMMITTED와 같은 방식으로 작동한다.


## Dirty Read
불안정한 데이터(unreliable data), 즉 다른 트랜잭션에 의해 업데이트되었지만 **아직 커밋되지 않은 데이터를 읽는 것**을 의미한다.   
이는 `READ UNCOMMITTED` 격리 수준에서만 가능하다.

이러한 종류의 작업은 데이터베이스 설계의 `ACID` 원칙을 준수하지 않는다.

[ACID 원칙]({{ site.url }}{{ site.baseurl }}/database/ACID) 위배
: ACID 원칙이란, 데이터베이스 **트랜잭션의 안정성을 보장**하기 위한 설계 원칙이다.  
  Dirty Read는 이 중 `일관성(Consistency)`과 `격리성(Isolation)` 원칙에 위배된다.

위험성
: 데이터가 롤백되거나, 커밋되기 전에 추가로 업데이트될 수 있기 때문에 **매우 위험**하며,  
  더티 리드를 수행하는 트랜잭션은 정확하지 않은 데이터를 사용하게 된다.  

- (ex) 트랜잭션 A에서 변경한 데이터를 트랜잭션 B가 읽었는데, A가 Rollback 되면 B에서 읽은 데이터는 실제로 존재하지 않는 데이터가 된다.

이와 반대되는 것은 [`Consistent Read`(일관된 읽기)]({{ site.url }}{{ site.baseurl }}/database/Consistent-read와-Isolation-level/#consistent-read)로,
: InnoDB는 트랜잭션이 다른 트랜잭션에 의해 업데이트된 정보를(커밋을 하더라도) 읽지 않도록 보장한다.  
  이는 `READ COMMITTED` 격리 수준 이상에서 가능하다.

## vs READ COMMITTED
READ COMMITTED는 다른 트랜잭션에서 커밋된 데이터만 읽는다.  
즉, **READ COMMITTED에서는 Dirty Read가 발생하지 않는다.**


<br>
<br>

# References

[https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html)  