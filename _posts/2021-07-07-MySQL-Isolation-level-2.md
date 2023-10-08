---
title: "Consistent Nonlocking Reads와 Isolation level"
date: 2021-07-07
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
> Transaction Isolation levels 시리즈 - 2

Consistent Nonlocking Reads(이하 "consistent read")는 
InnoDB에서 Isolation level에 따라 트랜잭션 내에서 `SELECT` 문의 조회 결과가 달라지는 부분을 이해하기 위해 꼭 짚고 넘어가야 하는 기술이다.

앞서 "Isolation level과 REPEATABLE READ" 글을 작성하며 이해가 어려웠던 부분이 있어, 
다른 Isolation level을 공부하기 전에 먼저 consistent read를 포스팅한다.

<br>
<br>

# Consistent read

동시에 실행되는 다른 트랜잭션의 변경 사항에 관계없이, 스냅샷을 사용하여 특정 시점을 기준으로 쿼리 결과를 출력하는 read 작업이다.
- 'consistent read을 실행한다'는 것은 일반적인 `SELECT` 문을 실행하는 것이라고 보면 된다.
- 쿼리된 데이터가 다른 트랜잭션에 의해 변경된 경우, **원래 데이터는 undo log를 기반으로 재구성**된다.
- 이 기술은 **동시성을 저하시킬 수 있는 locking issue를 방지**한다.
  - 액세스하는 테이블에 lock을 하지 않기 때문에 다른 세션은 테이블에서 consistent read가 수행되는 동시에 해당 테이블을 자유롭게 수정할 수 있다.

쿼리는 해당 시점 이전에 커밋된 트랜잭션이 변경한 내용만 보고, 이후 또는 커밋되지 않은 트랜잭션이 변경한 내용은 보지 않는다.
이 규칙의 예외는 쿼리가 **동일한 트랜잭션 내에서 이전 명령문에 의해 수행된 변경 사항을 볼 수 있다**는 것이다. (예제의 step4)

이 예외로 인해 테이블의 일부 row를 업데이트하는 경우, `SELECT`에 업데이트된 row의 최신 버전이 표시되지만 이전 버전의 row도 표시될 수 있게 되었다.
이는 서로 다른 세션이 동시에 동일한 테이블을 업데이트하는 경우, 데이터베이스에 존재하지 않았던 상태의 테이블이 표시될 수 있음을 의미한다.

## 예제
스냅샷은 반드시 DML 문에 적용되는 것은 아니지만, 트랜잭션 내의 `SELECT` 문에는 적용된다.  
격리 수준이 `REPEATABLE READ`인 경우를 예로 들어본다.

트랜잭션 B에서 row를 삽입/수정한 다음 커밋하는 경우,
- 동시 진행 중인 트랜잭션 A에서 B가 커밋한 row를 조회할 수 없었더라도
  - 이미 A에서 첫 `SELECT`가 실행되어 스냅샷 생성되었다면 해당 row를 조회할 수 없다.
- 해당 row에 대해 `DELETE` 또는 `UPDATE`를 실행하면 해당 row를 수정/삭제할 수 있다.

트랜잭션이 다른 트랜잭션에 의해 커밋된 row를 수정/삭제하는 경우, 이러한 변경 사항이 현재 트랜잭션에 표시된다.
- A가 step3에서 수정한 row를 step4에서 조회할 수 있었다.

<br>

코드를 보고 이해하자.

1. step1에서 트랜잭션 A가 `c = 'aaa'`인 row를 조회했지만 결과는 0건이었다.
2. A가 커밋되기 전에 step2에서 동시에 다른 트랜잭션 B가 `c = 'aaa'`인 row 10건을 삽입 또는 수정했다.
3. A는 step1에서 어떠한 row도 조회되지 않았지만, step3에서 `UPDATE`을 실행하니 10건의 row가 수정되었다.
4. step4에서 A가 자신이 `c = 'ccc'`로 `UPDATE`한 row를 조회하니 10건이 조회되었다.

UPDATE
```sql
time     Transaction A           Transaction B
v       (REPEATABLE READ)

        # step1
        SELECT COUNT(c)
        FROM t
        WHERE c = 'aaa';
        -- 결과: 0
                                # step2
                                INSERT/UPDATE t
                                10 rows with 'aaa' values;

                                COMMIT;
        # step3                        
        UPDATE t
        SET c = 'ccc' 
        WHERE c = 'aaa';
        -- 트랜잭션 A의 첫 SELECT에서 조회된 row는 없었지만, 트랜잭션 B에서 커밋한 10개 row가 수정되었다.

        # step4
        SELECT COUNT(c)
        FROM t
        WHERE c = 'ccc';
        -- 결과: 10
```

DELETE
```sql
time     Transaction A           Transaction B
v
        SELECT COUNT(c) 
        FROM t 
        WHERE c = 'aaa';
        -- 결과: 0
                                INSERT/UPDATE t
                                10 rows with 'aaa' values;

                                COMMIT;
        DELETE FROM t 
        WHERE c = 'aaa';
        -- 역시 트랜잭션 A의 첫 SELECT에서 조회된 row는 없었지만, 트랜잭션 B가 최근에 커밋한 10개 row가 삭제되었다.
```

<br>
<br>

# Consistent read가 작동하지 않는 경우

## 특정 DDL 문
특정 DDL 문에서는 consistent read가 작동하지 않는다.

### DROP TABLE
MySQL은 삭제된 테이블을 사용할 수 없고 InnoDB가 테이블을 삭제하기 때문에 consistent read는 DROP TABLE에서 작동하지 않는다.

### ALTER TABLE
원본 테이블의 임시 복사본을 만들고 임시 복사본이 작성될 때 원본 테이블을 삭제하는 ALTER TABLE 작업에서는 일관된 읽기가 작동하지 않는다.
- 새 테이블의 row는 트랜잭션의 스냅샷을 생성할 때 존재하지 않았기 때문에 표시되지 않는다.
- 이 경우 트랜잭션은 오류를 반환한다
  ```
  ER_TABLE_DEF_CHANGED, “Table definition has changed, please retry transaction”.
  ```

## 특정 SELECT 유형
기본적으로 InnoDB는 아래와 같은 `SELECT` 문장에 더 강력한 `lock`을 사용하며,
해당 문장의 `SELECT` 부분은 READ COMMITTED처럼 작동한다.
여기서 각 consistent read는 동일한 트랜잭션 내에서도 자체적인 스냅샷을 생성한다.

- `INSERT INTO ... SELECT`
- `UPDATE ... (SELECT)`
- `CREATE TABLE ... SELECT`

이러한 경우에 비잠금 읽기(nonlocking read)를 수행하려면,
`SELECT`된 테이블에서 읽은 row에 `lock`을 설정하지 않도록 트랜잭션의 격리 수준을 READ UNCOMMITTED 또는 READ COMMITTED로 설정해야 한다.

<br>
<br>

# with Transaction isolation level
consistent read는 InnoDB가 `READ COMMITTED` 및 `REPEATABLE READ` isolation level에서 `SELECT` 문을 처리하는 기본 모드다.

## REPEATABLE READ
트랜잭션 내의 모든 consistent read는 해당 트랜잭션의 **첫 번째 read 시점에 생성**된 스냅샷을 조회한다.

1. 트랜잭션 A: `DELETE` row
2. 트랜잭션 B: `SELECT` 실행 (=시점 할당 =스냅샷 생성)
3. 트랜잭션 A: `COMMIT`
4. 트랜잭션 B: `SELECT` 실행 (=스냅샷 조회: A에서 삭제된 row가 삭제된 것으로 표시되지 않는다) 

INSERT/UPDATE도 비슷하게 처리된다.

## READ COMMITTED
트랜잭션 내 **각각의 consistent read가 자체적인 새로운 스냅샷을 생성**하고 조회한다.

<br>
<br>

# MVCC (Multi-Versioned Concurrency Control)
트랜잭션을 커밋한 다음, 다른 SELECT 또는 START TRANSACTION WITH CONSISTENT SNAPSHOT을 수행하여 시점을 앞당길 수 있는데, 
이를 **다중 버전 동시성 제어**(Multi-Versioned Concurrency Control)라고 한다.

아래 예제에서 세션 A는 B가 `INSERT`를 커밋하고 A도 커밋한 경우에만, 
B가 `INSERT`한 row를 조회할 수 있으므로 시점도 B의 커밋보다 이전에 있다.

```sql
             Session A              Session B

           SET autocommit=0;      SET autocommit=0;
time
|          SELECT * FROM t;
|          empty set
|                                 INSERT INTO t VALUES (1, 2);
|
v          SELECT * FROM t;
           empty set
                                  COMMIT;

           SELECT * FROM t;
           empty set

           COMMIT;

           SELECT * FROM t;
           ---------------------
           |    1    |    2    |
           ---------------------
```

<br>
<br>

# DB 최신 상태 조회 방법
데이터베이스의 가장 최신 상태를 조회하려면, 아래 2가지 중 하나를 사용하면 된다. 
- isolation level = READ COMMITTED
- locking read

## READ COMMITTED
READ COMMITTED 격리 수준을 사용하면 트랜잭션 내의 각 consistent read가 자체적으로 새로운 스냅샷을 생성하고 읽는다.

## locking read
```sql
SELECT * FROM t FOR SHARE;
```
FOR SHARE를 사용하면 locking read가 발생한다.
locking read는 InnoDB 테이블에 대한 lock도 함께 수행하는 `SELECT` 문이다.

- `SELECT ... FOR UPDATE` 또는 `SELECT ... FOR SHARE` 중 하나
- 트랜잭션 isolation level에 따라 교착 상태(deadlock)가 발생할 수 있다.
- read-only transaction의 global table에는 허용되지 않는다.

<br>
<br>

# References

[https://dev.mysql.com/doc/refman/8.0/en/innodb-consistent-read.html](https://dev.mysql.com/doc/refman/8.0/en/innodb-consistent-read.html)