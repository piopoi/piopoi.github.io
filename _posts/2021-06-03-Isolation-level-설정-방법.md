---
title: "Isolation level 설정 방법"
date: 2021-06-03
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
> MySQL 8.0 Reference Manual로 공부하기

{% include_relative series/MySQL_8.0_Reference_Manual.md %}

InnoDB에서 제공하는 4가지 트랜잭션 격리수준은 다음과 같다.
- READ-UNCOMMITTED
- READ-COMMITTED
- **REPEATABLE-READ (default)**
- SERIALIZABLE

<br>
<br>

# 서버의 기본 격리 수준 설정
서버 시작 시 또는 설정 파일에서 아래 옵션을 사용하면 된다.
```
--transaction-isolation=level
```

<br>
<br>

# 런타임 중 설정

## Global
전역 설정을 변경하면 모든 후속 세션에 대한 격리 수준이 설정되며, 기존 세션은 영향을 받지 않는다.
```
SET GLOBAL transaction_isolation = 'REPEATABLE-READ';
```

## Session
세션 내에서 수행되는 모든 후속 트랜잭션에 대한 격리 수준을 설정한다.
```
# 아래 명령문 중 하나를 사용하면 된다.
SET @@SESSION.transaction_isolation = 'REPEATABLE-READ';
SET SESSION transaction_isolation = 'REPEATABLE-READ';
SET transaction_isolation = 'REPEATABLE-READ';
```
- 트랜잭션 내에서 허용되지만 현재 진행 중인 트랜잭션에는 영향을 주지 않는다.  
- 트랜잭션 간에 실행되는 경우 다음 트랜잭션 격리 수준을 설정하는 모든 선행 문을 재정의 한다.  

## Next single transaction
세션 내에서 수행되는 다음 단일 트랜잭션에 대해서만 격리 수준을 설정한다.
```
SET @@transaction_isolation = 'REPEATABLE-READ';
```
- 후속 트랜잭션은 session isolation level로 돌아간다.
- 트랜잭션 내에서는 허용되지 않는다.




<br>
<br>

# References

[https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html)