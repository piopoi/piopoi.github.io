---
title: "InnoDB Locking: Intention Lock"
date: 2022-04-16
categories:
  - Database
tags:
  - mysql
---

> MySQL 8.0 Reference Manual로 공부하기

{% include_relative series/MySQL_8.0_Reference_Manual.md %}

# Intention Lock (의도 잠금)

Intention Lock(이하 인텐션 락)은 MySQL의 InnoDB 엔진에서 **다중 세분화 잠금(Multiple granularity locking)을 지원**하는 메커니즘이다.

- 다중 세분화 잠금은 `Row Lock`과 `Table Lock`이 동시에 존재할 수 있게 한다.
- 예를 들어, `LOCK TABLES ... WRITE` 문은 지정된 테이블에 배타 락(Exclusive Lock)을 설정한다.

InnoDB에서 **인텐션 락은 테이블 레벨에서 설정**되며,  
이후에 **행 레벨에서 어떤 유형의 락(공유 락 또는 배타 락)이 필요**한지 나타낸다.

이는 `의도(Intention)`를 표시하는 것으로, 실제 행에 대한 락을 설정하기 전에 먼저 테이블 레벨에서 이러한 인텐션 락이 설정된다.

<br>

## 사용 이유

**다중 세분화 잠금을 실용적으로 만들기 위해서**이다.  
즉, 테이블 레벨과 행 레벨에서 다양한 락을 동시에 다룰 수 있게 하려는 의도이다.

예를 들어, 하나의 트랜잭션이 특정 행에 대한 배타 락을 설정하려고 할 때, 먼저 해당 테이블에 대한 Intention Exclusive Lock(IX)을 설정해야 한다.

이렇게 하면, **다른 트랜잭션이 동일한 테이블에 대해 어떤 작업을 할 수 있는지에 대한 정보를 시스템이 미리 알 수 있다.**

<br>

## 유형 (Type)

### Intention Shared Lock (IS)

트랜잭션이 테이블의 개별 행에 공유 락을 설정할 계획이라는 것을 나타낸다.

```sql
SELECT ...FOR SHARE
```

### Intention Exclusive Lock (IX)

트랜잭션이 테이블의 개별 행에 배타 락을 설정할 계획이라는 것을 나타낸다.

```sql
SELECT ...FOR UPDATE
```

<br>

## Intention locking protocol

이 프로토콜은 트랜잭션이 특정 행에 대한 락을 설정하기 전에 해당 테이블에 대한 인텐션 락을 먼저 설정해야 한다는 규칙을 명시한다.

- Intention Shared Lock (IS)
  - 트랜잭션이 **행에 공유 락을 획득하기 전**에 먼저 **테이블 레벨에서 IS 락이나 그 이상의 락을 획득**해야 한다.
- Intention Exclusive Lock (IX)
  - 트랜잭션이 **행에 배타 락을 획득하기 전**에 먼저 **테이블 레벨에서 IX 락을 획득**해야 한다.

### 호환성

락을 요청하는 트랜잭션은 기존 락과 호환되어야만 락이 부여된다.
테이블 레벨의 락 유형 간 호환성은 다음과 같다:

|    | X        | IX         | S          | IS         |
|----|----------|------------|------------|------------|
| X  | Conflict | Conflict   | Conflict   | Conflict   |
| IX | Conflict | Compatible | Conflict   | Compatible |
| S  | Conflict | Conflict   | Compatible | Compatible |
| IS | Conflict | Compatible | Compatible | Compatible |

- 기존에 공유 락이 있고 새 트랜잭션이 공유 락을 요청한다면, 락이 부여된다.  
- 기존에 배타 락이 있고 새 트랜잭션이 공유 락/배타 락을 요청하면, 락이 부여되지 않고 기존 락이 해제될 때까지 대기한다.


### Deadlock (교착 상태)

두 개 이상의 트랜잭션이 서로의 락이 해제되기를 기다리면서 발생하는 무한 대기 상황을 Deadlock(교착 상태) 이라고 한다.  
인텐션 락 요청이 기존 락과 충돌하여 Deadlock을 일으킬 것 같다면, 시스템은 오류를 발생시킨다.

### 차단 제외

인텐션 락은 `LOCK TABLES ... WRITE`와 같은 테이블 전체 요청을 제외하고는 어떤 것도 차단하지 않는다.  
이는 인텐션 락의 실제로 행을 lock 하는 것이 아니라, lock을 설정할 `의도(Intention)`만을 나타내기 때문이다. 

<br>

## 트랜잭션 데이터 조회

인텐션 락에 대한 트랜잭션 데이터는 `SHOW ENGINE INNODB STATUS` 명령으로 다음과 같이 확인할 수 있다:

```
TABLE LOCK table `test`.`t` trx id 10080 lock mode IX
```

<br>
<br>

# References

[https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html)  