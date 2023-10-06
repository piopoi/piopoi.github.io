---
title: "Isolation level과 REPEATABLE READ"
date: 2021-06-02
categories:
  - Database
tags:
  - MySQL
toc: true
toc_sticky: true
#published: false
#use_math: true
---
> MySQL 8.0 Reference Manual로 공부하기

MySQL에서 매우 중요한 개념인 Transaction isolation levels에 대한 개념과 
innoDB의 default isolation level인 REPEATABLE READ에 대해 알아본다.

<br>

# Transaction isolation levels (트랜잭션 격리 수준)

트랜잭션 격리(Transaction isolation)는 데이터베이스 처리의 기초 중 하나로,
여러 트랜잭션이 동시에 변경하고 쿼리를 수행할 때 성능과 안정성, 일관성 및 결과 재현성 간의 균형을 미세 조정하는 설정이다. (ACID의 I)

격리 수준이 낮을 수록 많은 사용자가 동시에 동일한 데이터에 액세스할 수 있는 능력이 높아지지만,
사용자가 겪을 수 있는 동시성 효과(dirty reads, lost updates 등)도 늘어난다.
반대로 격리 수준이 높을 수록 사용자에게 발생할 수 있는 동시성 효과 유형은 줄어들지만, 
더 많은 시스템 리소스가 필요하고 한 트랜잭션이 다른 트랜잭션을 차단할 가능성이 높아진다.

InnoDB는 SQL:1992(=[SQL-92](https://en.wikipedia.org/wiki/SQL-92)) 표준에서 설명하는 네 가지 트랜잭션 격리 수준을 모두 제공한다.
- READ-UNCOMMITTED
- READ-COMMITTED
- **REPEATABLE-READ (InnoDB default)**
- SERIALIZABLE

InnoDB는 다양한 Locking strategy를 사용하여 각 트랜잭션 격리 수준을 지원한다.
- Locking strategy: 트랜잭션이 다른 트랜잭션에 의해 쿼리되거나 변경되는 데이터를 보거나 변경하지 못하도록 보호하는 시스템이다.
  Locking strategy는 데이터베이스 작업의 **신뢰성**과 **일관성**(ACID 원칙), 우수한 동시성을 위해 필요한 **성능** 간의 균형을 유지해야 한다.

ACID 규정 준수가 중요한 중요한 데이터에 대한 작업의 경우 기본 REPEATABLE READ 수준을 사용하여 높은 수준의 일관성을 적용할 수 있다.
또는 대량 보고(bulk reporting)와 같이, 정확한 일관성(precise consistency)과 반복 가능한 결과(repeatable results)가
Locking을 위한 오버헤드를 최소화하는 것보다 덜 중요한 상황에서는 READ COMMITTED 또는 READ UNCOMMITTED로 일관성 규칙을 완화할 수 있다.
SERIALIZABLE은 REPEATABLE READ보다 더 엄격한 규칙을 적용하며, 주로 XA 트랜잭션과 같은 특수한 상황에서 동시성 및 교착 상태(deadlock)와 관련된 문제를 해결하는 데 사용된다.
- XA: 분산 트랜잭션을 조정하기 위한 표준 인터페이스로, 여러 데이터베이스가 ACID 규정을 준수하면서 트랜잭션에 참여할 수 있다.
  XA 분산 트랜잭션 지원은 기본적으로 활성화되어 있습니다.

## REPEATABLE READ
InnoDB의 **기본 격리 수준**으로, **트랜잭션 내에서 첫 번째 SELECT 시점에 snapshot을 만들고 이 snapshot을 조회**한다.
즉, 동일한 트랜잭션 내에서 `SELECT` 문(nonlocking)을 여러 개 실행하는 경우에도 일관성을 유지한다. ([Consistent Nonlocking Reads](https://dev.mysql.com/doc/refman/8.0/en/innodb-consistent-read.html)) 

locking reads(`SELECT` with `FOR UPDATE` or `FOR SHARE`)/`UPDATE`/`DELETE` 문의 경우,
locking은 명령문이 고유한 검색 조건이 있는 unique index를 사용하는지 또는 range-type 검색 조건을 사용하는지에 따라 달라진다.
- 고유한 검색 조건(unique search condition)이 있는 unique index의 경우,
  - InnoDB는 발견된 index record만 잠그고(**record lock**) 그 앞의 gap은 잠그지 않는다.
- 다른 검색 조건의 경우, 
  - InnoDB는 gap locks 또는 next-key locks를 사용하여 검색된 인덱스 범위를 잠그고, 해당 범위가 포함하는 gap에 다른 세션이 삽입하는 것을 차단한다.

### record lock
index record에 대한 lock 이다.  
예를 들어, 아래 sql문은 다른 트랜잭션이 `t.c1 = 10`인 row 들을 삽입/수정/삭제하지 못하도록 한다.
```
SELECT c1 
FROM t 
WHERE c1 = 10 
FOR UPDATE;
```

### gap lock
index record 사이의 간격(gap)에 대한 lock.
또는 첫 번째 또는 마지막 index record 앞뒤의 gap에 대한 lock이다.  
예를 들어, 아래 sql문은 범위 내의 모든 기존 값 사이의 gap이 잠겨 있으므로 
다른 트랜잭션이 t.c1 열에 15라는 값을 삽입하지 못하도록 한다. (t.c1에 이미 해당 값(15)이 있는지 여부와 관계없이)
```
SELECT c1 
FROM t 
WHERE c1 BETWEEN 10 AND 20 
FOR UPDATE;
```

gap lock은 성능과 동시성 간의 절충안의 일부이며, 
일부 트랜잭션 격리 수준에서는 사용되지만 다른 트랜잭션 격리 수준에서는 사용되지 않는다.

### next-key lock
index record에 대한 record lock와 index record 앞의 gap에 대한 gap lock의 조합이다.

<br>
<br>

# References

[https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html)
[https://en.wikipedia.org/wiki/Isolation_(database_systems)](https://en.wikipedia.org/wiki/Isolation_(database_systems))