---
title: "MySQL이 Index를 사용하는 방법"
date: 2022-08-01
categories: knowledge
tags:
  - database
  - mysql
---

{% include_relative series/MySQL_8.0_Reference_Manual.md %}

# Database에서 Index란?

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/2022/0801/index.png"/>  

인덱스(Index)는 테이블에서 검색 작업의 속도를 향상시키는 자료 구조(data structure)로,
테이블에 접근(access)할 때마다 해당 테이블의 모든 행(row)을 검색할 필요 없이 데이터를 빠르게 찾는 데 사용된다.

- 인덱스는 테이블에서 선택한 데이터 열(column)의 복사본이다.
  - Index를 유지하기 위해 쓰기(write) 작업이 추가되며 추가 저장 공간도 필요하다.
  - 데이터의 원본 행의 key 또는 위치(주소)가 포함된다.
- 인덱스는 테이블 내의 1개 이상의 컬럼을 이용하여 생성할 수 있다.
- 빠른 무작위 조회(rapid random lookups)와 정렬된 레코드의 효율적인 접근을 제공한다.

<br>
<br>

# MySQL이 Index를 사용하는 방법

인덱스는 특정 열 값을 가진 행을 빠르게 찾는 데 사용된다.
인덱스가 없으면 MySQL은 첫 번째 행부터 시작하여 전체 테이블을 읽어 관련 행을 찾아야 한다. 
테이블이 클수록 이 작업은 더 많은 비용이 들어간다.
테이블에 해당 열에 대한 인덱스가 있으면 MySQL은 모든 데이터를 볼 필요 없이 데이터 파일 중간에서 찾아야 할 위치를 빠르게 결정할 수 있다. 
이는 **모든 행을 순차적으로 읽는 것보다 훨씬 빠르다.**

SELECT 성능을 향상시키는 가장 좋은 방법은 쿼리에서 검사하는 1개 이상의 열에 인덱스를 생성하는 것이다.
인덱스 항목(index entries)은 테이블 행에 대한 포인터처럼 작동하므로,
쿼리에서 WHERE 절의 조건과 일치하는 행을 빠르게 확인하고 해당 행의 다른 열 값을 조회할 수 있다.

쿼리에 사용되는 모든 가능한 열에 대해 인덱스를 만들고 싶을 수 있지만,
불필요한 인덱스는 자원을 낭비하고 MySQL이 사용할 인덱스를 결정하는 데 시간을 낭비하게 한다.
또한, 각 인덱스를 업데이트해야 하므로 인덱스는 삽입, 업데이트, 삭제 연산에 대한 비용이 추가된다.
결국 최적의 인덱스 집합을 사용하여 빠른 쿼리를 달성하려면 적절한 균형을 찾아야 한다.

## Index는 어떻게 저장될까?
**대부분의 MySQL 인덱스(PRIMARY KEY, UNIQUE, INDEX, FULLTEXT)는 `B-tree` 자료 구조로 저장된다.**

- 예외: 
  - 공간(spatial) 데이터 유형의 인덱스는 `R-tree`를 사용한다.
  - MEMORY 테이블은 `Hash Index`도 지원한다.
  - InnoDB는 FULLTEXT index에 `Inverted Index`를 사용한다.

### B-tree
정렬된 데이터를 유지하고 $O(log n)$ 시간 내에 검색, 순차적 액세스, 삽입 및 삭제가 가능한 자체 균형(self-balancing) 트리 자료 구조.
  - B-tree는 이진 검색 트리를 확장하여 두 개 이상의 자식을 가질 수 있다.

다른 자체 균형 이진 검색 트리와 달리 B-tree는 데이터베이스 및 파일 시스템과 같이 비교적 큰 데이터 블록을 읽고 쓰는 스토리지 시스템에 적합하다.
이 구조는 **항상 정렬된 상태로 유지되므로 정확히 일치하는 항목(equals 연산자)과 범위(range, 예: greater than, less then 또는 BETWEEN 연산자)를 빠르게 조회할 수 있다.**

- B-tree 인덱스는 InnoDB, MyISAM 등 대부분의 스토리지 엔진에서 사용할 수 있다.
  - MEMORY 스토리지 엔진에서만 사용할 수 있는 hash index와 대조된다.
    - MEMORY 스토리지 엔진은 B-tree 인덱스도 사용할 수 있다. 
    - 일부 쿼리가 범위 연산자를 사용하는 경우 MEMORY 테이블에 대해 B-tree 인덱스를 선택해야 한다.
- B-tree라는 용어의 사용은 인덱스 설계의 일반적인 클래스를 참조하기 위한 것으로,
MySQL 스토리지 엔진에서 사용하는 B-트리 구조는 고전적인 B-트리 디자인에는 없는 정교함(sophistications)으로 인해 변형으로 볼 수 있다.

### B+tree
B-tree를 확장 및 개선한 자료 구조.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/2022/0801/Bplustree.png"/>

1. **모든 레코드들이 트리의 가장 하위 레벨(리프 노드)에 정렬되어 있다.**
  - 리프 노드에는 인덱스 키와 그 키에 해당하는 데이터 레코드가 저장된 위치의 포인터가 함께 저장된다.
    - 이 포인터는 데이터가 실제로 저장된 테이블의 행(row)를 가리킨다.
  - 이 구조는 전체 key를 순차적으로 검색하는데 유리하다.
  - 리프 노드(Leaf Node): 자식 노드가 없는 노드.
2. **내부 노드에는 자식 노드를 가리키는 포인터와 각 서브 트리에서의 최소값 또는 최대값을 나타내는 키(key)가 저장된다.**
  - 이로 인해 더 많은 key를 한 페이지에 저장할 수 있어, 디스크 I/O를 줄이고 Cache 효율을 높일 수 있다.
  - 내부 노드 (Internal Node): 자식이 하나 이상 있는 노드
3. **리프 노드들은 서로 연결되어 있다.**
  - 범위 쿼리에 효율적이다.
  - LinkedList 구조로 서로를 참조하고 있다.
  - 부등호(<, >)를 이요한 순차 검색 연산을 하는 경우, 많은 노드를 방문해야 하는 B-tree에 비해,
    B+tree는 말단의 리프 노드를 저장한 LinkedList를 한 번만 탐색하는 등 속도에 이점이 있다.

### R-tree
N차원의 공간 데이터를 효과적으로 저장하고 지리정보와 관련된 질의를 빠르게 수행 할 수 있는 자료구조.
이를테면, 지리학에서 R-tree는 "현재 위치에서 200km 이내의 모든 도시를 찾아라"와 같은 질의에 대해 빠르게 답을 줄 수 있다.

### Hash Index
Greater-than 또는 BETWEEN과 같은 범위 연산자 대신 등호 연산자(=)를 사용하는 쿼리를 위한 인덱스 유형이다.
MEMORY 테이블에서 사용할 수 있다.
Hash Index가 MEMORY 테이블의 기본값이지만, MEMORY 스토리지 엔진은 범용 쿼리에 더 적합한 B-트리 인덱스도 지원한다.
MySQL에는 이 인덱스의 변형인 adaptive(적응형) Hash Index가 포함되어 있으며, 
이 인덱스는 런타임 조건에 따라 필요한 경우 InnoDB 테이블에 대해 자동으로 생성된다.

### Inverted Index
문서 검색 시스템에 최적화된 데이터 구조로 **InnoDB에서 full-text 검색(search)를 구현하는 데 사용**된다.
inverted index로 구현된 InnoDB FULLTEXT index는 테이블 행의 위치가 아닌 문서(document) 내 각 단어(word)의 위치를 기록한다. 
하나의 열 값(텍스트 문자열로 저장된 문서)은 inverted index의 여러 항목(entry)으로 표현된다.

<br>
<br>

# MySQL은 이러한 연산에 인덱스를 사용한다


## 1. WHERE 절에 맞는 행 빠르게 찾기
MySQL은 WHERE 절의 조건과 일치하는 행을 빠르게 찾기 위해 인덱스를 사용한다.


## 2. 행 제거 고려
MySQL은 **쿼리가 처리해야 할 데이터의 양을 줄이기 위해** 인덱스를 사용한다.  
여러 인덱스가 있는 경우, MySQL은 **가장 적은 수의 행을 찾는 인덱스(가장 선택적(selectivity)인 인덱스)를 사용**한다.

- selectivity
  - 데이터 분포의 특성으로, 열의 고유 값 수(카디널리티, cardinality)를 테이블의 레코드 수로 나눈 값.
  - selectivity가 높다는 것은 열 값이 상대적으로 고유하며 인덱스를 통해 효율적으로 검색할 수 있음을 의미한다.
- **cardinality**
  - **특정 데이터 집합의 고유(unique)한 값의 개수**.
  - 남/여의 값만 가지는 성별 컬럼은 중복도가 높으며 카디널리티가 낮다.
  - 주민번호 컬럼은 중복도가 낮으며 카디널리티가 높다.

예를 들어, 인덱스 A가 쿼리 조건과 일치하는 행을 100개 찾고, 다른 인덱스 B가 같은 조건으로 10개의 행만 찾는다면, MySQL은 후자의 인덱스를 선택한다.
이는 검색해야 할 데이터의 양이 적기 때문에 더 효율적이기 때문이다.


## 3. 다중 컬럼 인덱스 사용
테이블에 다중 컬럼 인덱스(Multiple-Column Index)가 있다면,
옵티마이저(Optimizer)는 이러한 컬럼들의 조합 중 **가장 왼쪽에서 시작하는 연속된 부분 집합**을 행을 조회하는 데 사용할 수 있다.

예를 들어, (col1, col2, col3)라는 다중 컬럼 인덱스가 있을 때, 이 인덱스는 아래 3가지 조합으로 사용될 수 있다.
- (col1)
- (col1, col2)
- (col1, col2, col3)

이는 쿼리에서 col1만 사용하거나, col1과 col2를 함께 사용하는 경우에도 이 인덱스를 활용할 수 있다는 것을 의미하지만,  
**col2만 사용하는 경우에는 이 인덱스를 활용할 수 없다.**

## 4. 조인 시 인덱스 사용
조인(JOIN)을 수행할 때, 다른 테이블의 행을 검색하는 데 인덱스를 사용한다.  
만약 컬럼이 같은 타입과 크기로 선언되어 있다면, 인덱스를 더 효율적으로 사용할 수 ss있다.

- VARCHAR와 CHAR는 같은 크기로 선언되었다면, 동일한 것으로 간주된다.
  - (ex) VARCHAR(10)과 CHAR(10)은 같은 크기이지만, VARCHAR(10)과 CHAR(15)는 다르다.
- 이진이 아닌(nonbinary) 문자열 컬럼을 비교하려면, 두 열 모두 동일한 문자 집합(character set)을 사용해야 한다.
  - (ex) utf8mb4 컬럼과 latin1 컬럼을 비교하는 것은 인덱스 사용을 배제한다.
- 서로 다른 타입의 컬럼(예: 문자열과 숫자)을 비교할 때, 값이 직접적으로(변환없이) 비교할 수 없다면 인덱스를 사용하지 못할 수 있다.
  - 이는 데이터 형 변환 과정에서 발생하는 비효율 때문이다.
  - (ex) 숫자 컬럼에서의 값 1은 문자열 컬럼에서 '1', ' 1', '00001', '01.e1' 등 다양한 값과 비교할 수 있지만, 문자열 컬럼에 대한 인덱스 사용을 배제한다.

## 5. MIN() 또는 MAX() 값 찾기
MySQL에서 특정 인덱스된 컬럼 key_col에 대한 MIN() 또는 MAX() 값을 찾는 작업은 인덱스를 활용하여 최적화된다.

MySQL에서 MIN() 또는 MAX() 함수를 최적화(optimized)하기 위한 전처리(Preprocessing) 과정에서, 
쿼리가 특정 인덱스를 사용하고 있고, 그 인덱스에서 key_col 이전에 나오는 모든 컬럼에 대해 상수 값을 가지는 WHERE 조건(`WHERE key_part_N = 상수`)을 사용하는지 확인한다.

- key_part1, key_part2, key_part3 순으로 컬럼이 정렬된 인덱스가 있다고 하자.
이제 key_part3에 대한 MIN() 또는 MAX() 값을 찾고 싶다면,
쿼리는 key_part1과 key_part2에 대해 상수 값을 가지는 WHERE 조건을 포함해야 한다.
즉, key_part3를 찾기 전에 key_part1과 key_part2에 대한 조건이 **모두 충족**되어야 한다.

이후, MySQL은 각각의 MIN() 또는 MAX() 표현식에 대해 단일 키 조회를 수행하고, 이를 상수로 대체한다.
모든 표현식이 상수로 대체되면, 쿼리는 즉시 결과를 반환한다.

예를 들어, key_part1과 key_part2를 모두 포함하는 다중 컬럼 인덱스가 있다고 하자.  
아래 쿼리는 `WHERE` 절에서 key_part1에 대해 상수 값(10)을 사용하고 있으므로,
MySQL은 key_part2의 최소값과 최대값을 찾기 위해 해당 인덱스를 효율적으로 사용할 수 있다.

```sql
SELECT MIN(key_part2),MAX(key_part2)
FROM tbl_name 
WHERE key_part1 = 10;
```

## 6. 정렬 또는 그룹화 최적화
테이블의 정렬 또는 그룹화가 사용 가능한 인덱스의 가장 왼쪽에서 시작하는 연속된 부분 집합을 기준으로 수행되면 인덱스를 사용해 최적화할 수 있다.
- (ex) ORDER BY key_part1, key_part2
- 모든 키 부분 뒤에 DESC가 오면 키를 역순으로 읽는다.
  - 또는 인덱스가 내림차순 인덱스인 경우 키를 정방향으로 읽는다.

## 7. Covering Index 사용
경우에 따라 데이터 행을 참조하지 않고도 값을 검색하도록 쿼리를 최적화할 수 있다. 
- 특정 쿼리에서 데이터 행을 직접 조회하지 않고 필요한 모든 결과를 제공하는 인덱스를 `Covering Index`라고 한다. 

예를 들어, key_part1과 key_part3를 모두 포함하는 다중 컬럼 인덱스가 있을 때, 아래 쿼리는 선택한 값을 인덱스 트리에서 검색하여 속도를 높일 수 있다.
```sql
SELECT key_part3 
FROM tbl_name
WHERE key_part1 = 1
```

### Covering Index
쿼리에서 검색한 모든 컬럼을 포함하는 인덱스.
쿼리가 인덱스 값을 사용하여 전체 테이블 행을 찾는 대신 인덱스 구조에서 값을 반환하여 디스크 I/O를 절약할 수 있으며, 이는 성능 향상에 크게 기여한다.

InnoDB와 MyISAM의 차이점 중 하나는 InnoDB가 이 최적화 기술을 더 많은 인덱스에 적용할 수 있다는 점이다.
InnoDB의 보조 인덱스(`Secondary Index`)는 기본 키(Primary Key) 컬럼을 포함하기 때문에, 기본 키를 포함한 쿼리에서도 커버링 인덱스를 활용할 수 있다.
하지만, 트랜잭션에 의해 수정된 테이블에 대한 쿼리에서는 해당 트랜잭션이 끝날 때까지 이 기술을 적용할 수 없다.

커버링 인덱스는 단일 컬럼 인덱스 또는 복합 인덱스일 수 있으며, 적절한 쿼리가 주어진 경우 어떤 인덱스도 커버링 인덱스로 작동할 수 있다.
따라서 인덱스와 쿼리를 설계할 때 이 최적화 기법을 가능한 활용하도록 설계하는 것이 좋다.

### Secondary Index
InnoDB에서 보조 인덱스(Secondary Index)는 테이블 컬럼의 일부를 나타내는 인덱스 유형이다.

InnoDB 테이블은 보조 인덱스를 하나도 가지지 않거나, 하나 또는 여러 개를 가질 수 있다.
이는 클러스터형 인덱스와 대비되는데, 클러스터형 인덱스는 각 InnoDB 테이블에 필요하며 테이블의 모든 컬럼 데이터를 저장한다.

보조 인덱스는 인덱스된 컬럼에서만 값을 요구하는 쿼리를 만족시키는 데 사용될 수 있다. 
더 복잡한 쿼리의 경우, 보조 인덱스는 관련 행을 식별하는 데 사용되며, 이후 클러스터형 인덱스를 사용하여 조회한다.

전통적으로 보조 인덱스를 생성하고 제거하는 것은 InnoDB 테이블의 모든 데이터를 복사하는 데 상당한 오버헤드가 있었다.
그러나 빠른 인덱스 생성 기능(The fast index creation)으로 인해, 보조 인덱스의 `CREATE INDEX`, `DROP INDEX` 처리가 훨씬 빨라졌다.

<br>
<br>

# Index를 사용할 필요가 없는 경우

## 작은 테이블에서의 쿼리
테이블이 작으면 데이터베이스는 전체 테이블을 빠르게 스캔할 수 있어, 인덱스를 사용하는 것보다 더 효율적일 수 있다.

## 대규모 테이블에서 대부분 또는 모든 행을 처리하는 쿼리
이 경우에는 인덱스를 사용하여 특정 행을 찾는 것보다 테이블을 순차적으로 읽는 것이 더 빠를 수 있다.  
순차적 읽기(Sequential read)는 쿼리에 모든 행이 필요하지 않더라도 디스크 탐색을 최소화하여 성능을 향상시킬 수 있기 때문이다.

<br>
<br>

# References
[https://dev.mysql.com/doc/refman/8.0/en/mysql-indexes.html](https://dev.mysql.com/doc/refman/8.0/en/mysql-indexes.html){:target="_blank"}<br>
[https://ko.wikipedia.org/wiki/인덱스_(데이터베이스)](https://ko.wikipedia.org/wiki/%EC%9D%B8%EB%8D%B1%EC%8A%A4_(%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4)){:target="_blank"}<br>
[https://tecoble.techcourse.co.kr/post/2021-09-18-db-index/](https://tecoble.techcourse.co.kr/post/2021-09-18-db-index/){:target="_blank"}<br>

