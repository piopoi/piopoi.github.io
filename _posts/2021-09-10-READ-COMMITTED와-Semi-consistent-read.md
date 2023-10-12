---
title: "READ COMMITTED와 Semi-consistent read"
date: 2021-09-10
categories:
  - Database
tags:
  - MySQL
  - MySQL 8.0 Reference Manual
toc: true
toc_sticky: true
#published: false
use_math: true
---
> MySQL 8.0 Reference Manual로 공부하기  
> Series: Transaction isolation levels 4

{% include_relative series/transaction_isolation_level.md %}

트랜잭션 격리 수준 시리즈의 세 번째 글이다.
이 글에서는 [첫 번째 글]({{ site.url }}{{ site.baseurl }}/database/Isolation-level과-REPEATABLE-READ)에서 다루지 못한 나머지 isolation level 옵션들에 대해 알아보자.

<br>

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


# READ COMMITTED 사용 시 추가적인 effect

간단히 말하면, READ COMMITTED 격리 수준에서는 최근 커밋된 데이터를 읽을 수 있으며,
`UPDATE`나 `DELETE` 작업을 수행할 때 특정 조건에 맞는 행들만 잠그게 된다.
이로 인해 데드락 확률은 줄어들지만, 완전히 제거되지는 않는다.

### row lock 관련

1. UPDATE나 DELETE 명령어에 대한 row lock:
`READ COMMITTED` 격리 수준에서는 `UPDATE`나 `DELETE` 명령어를 사용할 때, **InnoDB 엔진은 실제로 업데이트되거나 삭제되는 행에 대해서만 lock을 잡는다.(row lock)**
   - row lock
     - 데이터베이스 내의 특정 행(row)에 대한 동시 액세스를 제한하는 메커니즘이다.
     - 여러 트랜잭션이 동일한 행을 변경하려고 할 때 중요하다.
     - row lock을 사용하면, 한 트랜잭션에서 해당 행을 잠글 수 있어, 다른 트랜잭션은 그 행에 접근할 수 없게 된다.
     - 이로 인해 데이터 무결성을 보장할 수 있다.

2. 비일치 행에 대한 record lock 해제:  
`WHERE` **조건을 사용하여** `UPDATE`나 `DELETE` 작업을 수행할 때, InnoDB는 해당 조건과 일치하는 행에 대해서만 lock을 유지한다.
조건에 일치하지 않는 행들에 대한 record lock은 MySQL이 WHERE 조건을 평가한 후 해제된다.
   - 예를 들어, 100개의 행 중 10개의 행만 UPDATE 명령어의 WHERE 조건과 일치한다면, 나머지 90개의 행에 대한 잠금은 즉시 해제된다.

3. Deadlock 확률 감소:  
이러한 방식은 여러 트랜잭션이 동시에 데이터에 액세스할 때 deadlock의 확률을 크게 줄인다. 
왜냐하면 트랜잭션은 실제로 필요한 행에 대해서만 락을 유지하기 때문이다.
그러나, 여전히 특정 상황에서는 데드락이 발생할 수 있다.
   - Deadlock (교착 상태)
     - 서로 다른 트랜잭션이 상대방에게 필요한 lock을 소유하고 있어 진행할 수 없는 상황이다.
     - 두 트랜잭션 모두 리소스를 사용할 수 있게 되기를 기다리고 있기 때문에, 어느 쪽도 소유하고 있는 lock을 해제하지 않는다.

결국, READ COMMITTED 격리 수준에서는 트랜잭션의 동시성과 효율성을 향상시키기 위해 필요한 행에 대해서만 락을 잡는 방식을 채택하고 있는데,
이는 동시에 여러 트랜잭션이 데이터베이스에 액세스할 때의 성능을 최적화하기 위함이다.

<br>

### semi-consistent read 관련
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

# References

[https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html)  
[https://chat.openai.com/?model=gpt-4](https://chat.openai.com/?model=gpt-4)