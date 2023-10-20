---
title: "InnoDB Locking: Shared Lock과 Exclusive Lock"
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
> MySQL 8.0 Reference Manual로 공부하기

{% include_relative series/MySQL_8.0_Reference_Manual.md %}

# 공유 락과 배타 락 (Shared and Exclusive Locks)

InnoDB는 표준 행 수준(row-level) locking을 구현하며,  
여기에는 공유 락(S lock, Shared lock)과 배타 락(X lock, Exclusive lock)이라는 두 가지 유형이 있다.

## 공유 락(S lock, Shared lock)

락을 보유한 트랜잭션에게 행을 `읽기만` 할 수 있는 권한을 부여한다.

- 어떤 행에 공유 락이 걸려있다면
  - 다른 트랜잭션이 
    - 그 행을 읽을 수 있고,
    - 공유 락을 획득할 수 있지만,
    - 수정이나 삭제는 할 수 없다.

## 배타 락(X lock, Exclusive lock)

락을 보유한 트랜잭션에게 행을 `수정/삭제` 할 수 있는 권한을 부여한다.

- 다른 트랜잭션이 동일한 행을 잠글 수 없게 하는 Lock이다.  
- 트랜잭션 격리 수준(Transaction isolation level)에 따라 다른 트랜잭션이 동일한 행에 쓰거나 읽는 것을 차단할 수 있다. 
  - 기본 InnoDB 격리 수준인 `REPEATABLE READ`는 `일관된 읽기(consistent read)`라는 기술로 **배타 락이 있는 행을 읽을 수 있게 함으로써 동시성을 높인다.**

## Example

트랜잭션 T1과 T2가 **동일한 행**에 lock을 보유하려고 한다고 가정하자.

| T1이 소유한 lock | T2가 요청한 lock  | T2의 요청에 대한 승인여부              |
|--------------|---------------|------------------------------|
| s lock       | s lock        | 가능                           |
| s lock       | x lock        | 불가. T1이 lock을 해제할 때까지 대기해야 함 |
| x lock       | s lock/x lock | 불가. T1이 lock을 해제할 때까지 대기해야 함 |


<br>
<br>

# References

[https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html)  