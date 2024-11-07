---
title: "READ COMMITTED와 UPDATE"
date: 2021-09-10
categories: knowledge
tags:
  - database
  - mysql
---

{% include_relative series/MySQL_8.0_Reference_Manual.md %}

# READ COMMITTED
하나의 트랜잭션 안에서 각각의 consistent read는 자체적인 새 스냅샷(Snapshot)을 생성하고 읽는다.

locking read(`SELECT` with `FOR UPDATE`/`FOR SHARE`), `UPDATE`, `DELETE` 문의 경우,
InnoDB는 인덱스 레코드만 잠그고(record lock) 그 앞의 gap은 잠그지(gap lock) 않으므로 잠긴 레코드 옆에 새 레코드를 자유롭게 삽입할 수 있다.

- `gap lock`은 외래 키 제약 조건 체크, 중복 키 체크에만 사용된다.
- 인덱스 레코드(Index record): 인덱스 레코드는 인덱스를 구성하는 레코드 를 의미한다. InnoDB는 디스크 I/O를 최소화하기 위해 특별히 최적화된 `B+tree` 자료구조를 사용하여 인덱스를 저장하기 때문에,
  데이터의 효율적인 조회와 삽입을 지원한다.

gap locking이 비활성화되어 있으므로 다른 세션에서 새 row를 gap에 삽입할 수 있으므로, phantom row 문제가 발생할 수 있다. 

- Phantom
  - 쿼리의 result set에 존재하는데 동일한 이전 쿼리의 result set에 존재하지 않는 row를 뜻한다.  
  - 예를 들어, `SELECT`가 두 번 실행되었지만 두 번째 실행 시 처음에 반환되지 않았던 row가 반환되는 경우 해당 row를 `phantom row`라고 한다. 

<br>

## Additional effects

### Row Lock과 Deadlock

1. UPDATE나 DELETE 명령어에 대한 Row Lock
- `READ COMMITTED` 격리 수준에서는 `UPDATE`나 `DELETE` 명령어를 사용할 때, **InnoDB 엔진은 실제로 업데이트되거나 삭제되는 행에 대해서만 lock을 잡는다.(row lock)**
- Row Lock
  - 데이터베이스 내의 특정 행(row)에 대한 동시 액세스를 제한하는 메커니즘이다.
  - 여러 트랜잭션이 동일한 행을 변경하려고 할 때 중요하다.
  - row lock을 사용하면, 한 트랜잭션에서 해당 행을 잠글 수 있어, 다른 트랜잭션은 그 행에 접근할 수 없게 된다.
  - 이로 인해 데이터 무결성을 보장할 수 있다.

2. 비일치 행에 대한 Record Lock 해제
- `WHERE` 조건을 사용하여 `UPDATE`나 `DELETE` 작업을 수행할 때, InnoDB는 해당 **조건과 일치하는 행에 대해서만 lock을 유지**한다.
- 조건에 일치하지 않는 행들에 대한 record lock은 MySQL이 **WHERE 조건을 평가한 후 해제**된다.
   - 예를 들어, 100개의 행 중 10개의 행만 UPDATE 명령어의 WHERE 조건과 일치한다면, 나머지 90개의 행에 대한 잠금은 즉시 해제된다.

3. Deadlock 확률 감소
- 이러한 방식은 **여러 트랜잭션이 동시에 데이터에 액세스할 때 deadlock의 확률을 크게 줄인다.** 
- 왜냐하면 트랜잭션은 실제로 필요한 행에 대해서만 락을 유지하기 때문이다.
- 그러나, **여전히 특정 상황에서는 데드락이 발생**할 수 있다.
- Deadlock (교착 상태)
  - 서로 다른 트랜잭션이 상대방에게 필요한 lock을 소유하고 있어 진행할 수 없는 상황이다.
  - 두 트랜잭션 모두 리소스를 사용할 수 있게 되기를 기다리고 있기 때문에, 어느 쪽도 소유하고 있는 lock을 해제하지 않는다.

결국, READ COMMITTED 격리 수준에서는 트랜잭션의 동시성과 효율성을 향상시키기 위해 필요한 행에 대해서만 락을 잡는 방식을 채택하고 있는데,
이는 동시에 여러 트랜잭션이 데이터베이스에 액세스할 때의 성능을 최적화하기 위함이다.

<br>

### Semi-consistent read

`UPDATE` 문을 실행할 때, `READ COMMITTED` 격리 수준에서는 `semi-consistent read` 방식을 사용하는데,
이로 인해 아래와 같은 상황이 발생한다.

1. 먼저, `UPDATE` 문의 `WHERE` 조건을 평가하기 위해 해당 행을 읽을 때, 
그 행이 이미 **다른 트랜잭션에 의해 잠겨 있을 경우,
InnoDB는 잠금을 시도하지 않고 대신 해당 행의 최신 커밋된 버전을 읽어오는데**, 이를 `semi-consistent read`라 한다.

2. 이제 MySQL은 그 읽어온 데이터를 사용하여 **해당 행이 WHERE 조건과 일치하는지 검사**한다.
만약 그 행이 UPDATE의 조건에 일치하면(즉, 그 행이 실제로 업데이트되어야 하는 상황이면), MySQL은 그 행을 실제로 **업데이트하기 전에 다시 한 번 해당 행을 읽는다.**

3. 이때, InnoDB는 다시 그 행에 접근하려고 시도한다.
만약 그 행이 여전히 다른 트랜잭션에 의해 잠겨 있다면, 이번에는 InnoDB가 그 행의 잠금을 기다리게 된다.
잠금이 해제되면 업데이트를 진행한다.

이러한 방식은 첫 번째 `semi-consistent read` 읽기를 통해 **빠르게 WHERE 조건을 검사**할 수 있고, 
**필요한 행만 실제로 잠금을 시도**하게 되어 **성능과 동시성에 유리**하다는 이점이 있다. 

하지만 이로 인해 **동일한 행을 두 번 읽게 되는** 상황이 발생하게 된다.

<br>
<br>

# Example: UPDATE

아래 `CREATE`문을 실행하여 생성할 테이블 t에는 인덱스가 없다.

이와 같이 별도로 인덱스를 생성해주지 않으면, 
검색 및 인덱스 스캔 시 인덱싱된 컬럼이 아니라 **숨겨진 클러스터된 인덱스(Clustered Index)를 사용하여 Record Lock을 수행**한다.

```sql
CREATE TABLE t (
    a INT NOT NULL, 
    b INT
) ENGINE = InnoDB;

INSERT INTO t VALUES (1,2),(2,3),(3,2),(4,3),(5,2);
COMMIT;
```

세션 A가 아래와 같이 업데이트를 수행한다고 가정해 보자.
```sql
# Session A
START TRANSACTION;
UPDATE t 
SET b = 5 
WHERE b = 3;
```

그리고 세션 B가 세션 A에 이어 업데이트를 수행한다고 가정한다:
```sql
# Session B
UPDATE t 
SET b = 4 
WHERE b = 2;
```

InnoDB는 각 UPDATE를 실행할 때 먼저 각 행에 대한 **Exclusive Lock을 획득한 다음 수정할지 여부를 결정**한다.

- `Exclusive Lock` (X-Lock, 배타 잠금)
  - 특정 행에 대한 Lock 방식으로, 한 번 Exclusive Lock이 설정되면, **해당 행에 다른 어떠한 Lock도 설정될 수 없다.**
  - 이로 인해 해당 행에 대한 동시성 액세스가 방지된다.
  - 트랜잭션 격리 수준에 따라 어떤 경우에는 쓰기만 차단되고, 어떤 경우에는 읽기와 쓰기 모두 차단될 수 있다.
  - InnoDB의 기본 격리 수준인 `REPEATABLE READ`에서는 `Consistent read`를 사용하여 Exclusive Lock이 걸린 행들도 트랜잭션 내에서 읽을 수 있게 해준다.
     이로 인해 다른 트랜잭션들과의 동시성이 높아지게 되지만, **트랜잭션 시작 시점**의 데이터를 읽어온다는 점을 잊지 말자.

InnoDB가 행을 수정하지 않으면 lock을 해제하고, 행을 수정하면 트랜잭션이 끝날 때까지 lock을 유지한다.

이는 다음과 같이 트랜잭션 처리에 영향을 준다.

<br>

## Case1. REPEATABLE READ

기본 `REPEATABLE READ` 격리 수준을 사용하는 경우,  
첫 번째 UPDATE는 읽는 각 행에 대해 x-lock을 획득하고 그 중 **어떤 것도 해제하지 않는다**:
```
x-lock(1,2); retain x-lock
x-lock(2,3); update(2,3) to (2,5); retain x-lock
x-lock(3,2); retain x-lock
x-lock(4,3); update(4,3) to (4,5); retain x-lock
x-lock(5,2); retain x-lock
```
<br>

**두 번째 UPDATE는 lock을 획득하려고 시도하는 즉시 차단되며(첫 번째 업데이트가 모든 행에 잠금을 유지했기 때문에),
첫 번째 UPDATE가 커밋되거나 롤백될 때까지 진행되지 않는다**:
```
x-lock(1,2); block and wait for first UPDATE to commit or roll back
```

<br>

## Case2. READ COMMITTED

대신 `READ COMMITTED` 격리 수준을 사용하면,  
첫 번째 UPDATE는 읽은 각 행에 대해 x-lock을 획득하고 **수정하지 않는 행에 대해서는 lock을 해제한다**:
```
x-lock(1,2); unlock(1,2)
x-lock(2,3); update(2,3) to (2,5); retain x-lock
x-lock(3,2); unlock(3,2)
x-lock(4,3); update(4,3) to (4,5); retain x-lock
x-lock(5,2); unlock(5,2)
```
<br>

두 번째 UPDATE의 경우, 
InnoDB는 `semi-consistent read`를 수행하여 읽은 **각 행의 최신 커밋된 버전을 MySQL에 반환**하므로 
MySQL은 해당 행이 업데이트의 WHERE 조건과 일치하는지 확인할 수 있다:
```
x-lock(1,2); update(1,2) to (1,4); retain x-lock
x-lock(2,3); unlock(2,3)
x-lock(3,2); update(3,2) to (3,4); retain x-lock
x-lock(4,3); unlock(4,3)
x-lock(5,2); update(5,2) to (5,4); retain x-lock
```
<br>

## Case3. WHERE 조건에 Indexed Column이 있다면

단, WHERE 조건에 인덱싱된 컬럼이 포함되어 있고 InnoDB가 해당 인덱스를 사용하는 경우, 
record lock을 얻고 유지할 때 인덱싱된 컬럼만 고려하게 된다.

아래 예제에서 
첫 번째 UPDATE(세션 A)는 `b = 2`인 각 행에 대해 x-lock을 얻고 유지한다.
- 인덱스는 `b`에만 설정되었기 때문에 `c = 3` 조건은 고려하지 않는다.

두 번째 UPDATE(세션 B)가 동일한 행(`b = 2`)에 대해 x-lock을 얻으려고 시도할 때, 
**b에 정의된 인덱스도 사용하므로 차단**되며 첫 번째 UPDATE가 커밋되거나 롤백될 때까지 진행되지 않는다.

```sql
CREATE TABLE t (
    a INT NOT NULL, 
    b INT, 
    c INT, 
    INDEX (b)
) ENGINE = InnoDB;

INSERT INTO t VALUES (1,2,3),(2,2,4);
COMMIT;

# Session A
START TRANSACTION;
UPDATE t 
SET b = 3 
WHERE b = 2 AND c = 3;

# Session B
UPDATE t 
SET b = 4 
WHERE b = 2 AND c = 4;
```

<br>

# References

[https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html){:target="_blank"}<br>