---
title: "배열(Array)과 연결리스트(Linked list)"
date: 2022-07-21
categories:
  - Computer Science
tags:
  - data structure
---

<br>

가장 기본적인 자료구조인 배열(Array)과 연결 리스트(Linked List)를 비교해보자.

<br>
<br>

# 차이점 요약

|          | Array                                   | Linked list                      |
|----------|-----------------------------------------|----------------------------------|
| 메모리 저장위치 | 연속된 위치에 저장한다.                           | 인접한 위치에 저장하지 않는다.                |
| 크기       | 정적 (고정된 크기)                             | 동적 (변경 가능한 크기)                   |
| 메모리 할당 시점 | 컴파일 시 할당된다.                             | 런타임 시 할당된다.                      |
| 접근 방식    | 연속된 위치에 저장되므로 인덱스를 사용하여 요소에 쉽게 접근 가능하다. | 특정 요소에 접근하려면 처음부터 순차적으로 탐색해야 한다. |
| 메모리 효율   | 더 효율적이다.                                | 데이터와 다음 노드의 주소까지 저장하므로 더 비효율적이다. |
| 삽입/삭제 효율 | 다른 요소들까지 이동해야 하므로 더 비효율적이다.             | 더 효율적이다.                         |


<br>
<br>

# 배열 (Array)

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/2022/0721/array.png"/>  

1. 배열은 메모리에서 연속적인 위치에 요소들을 저장하므로, 
2. 저장된 요소의 주소를 쉽계 계산할 수 있어 특정 인덱스의 요소에 더 빠르게 접근할 수 있다.
   - 예를 들어, arr[5]는 arr 배열의 6번째 요소에 직접 접근한다.
3. 대신, 배열의 크기가 생성 시 고정되어 나중에 변경하기 어렵다.

## 시간복잡도

| 연산            | 시간복잡도  |
|---------------|--------|
| 접근(Access)    | $O(1)$ |
| 검색(Search)    | $O(n)$ |
| 삽입(Insertion) | $O(n)$ |
| 삭제(Deletion)  | $O(n)$ |

- 접근(Access): 인덱스를 알고 있다면, 한 번의 연산으로 해당 요소에 접근할 수 있으므로 상수 시간이다.
  - 상수 시간:
    - 알고리즘이 처리해야 하는 데이터의 크기와 상관없이, 작업을 완료하는데 걸리는 시간이 일정하다는 것을 의미한다.
    - 보통 $O(1)$을 의미한다.
- 검색(Search): 최악의 경우 배열의 모든 요소를 확인해야 하므로 선형 시간이 걸린다.
  - 선형 시간: 
    - 알고리즘이 처리해야 하는 데이터의 양에 비례하여 수행시간이 증가한다는 것으로
    - 단순 반복 작업으로 볼 수 있으며
    - 보통 $O(n)$을 의미한다.
- 삽입(Insertion): 새 요소 삽입 시 기존의 요소들을 이동시켜야 할 수 있으므로, 최악의 경우 선형 시간이 필요하다. 
- 삭제(Deletion): 요소를 삭제한 후에는 남은 요소들을 이동시켜야 하므로, 최악의 경우 선형 시간이 걸린다.

<br>
<br>

# 연결 리스트 (Linked List)

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/2022/0721/linkedlist.png"/>  

1. 연결 리스트는 요소들이 메모리의 연속적인 위치에 있지 않고, 각 요소가 다음 요소의 주소를 포함하는 방식으로 구성된다.
   - 다음 요소의 주소를 저장해야 하므로 배열 대비 많은 메모리를 사용한다.
2. 위 특징으로 인해 리스트의 크기를 동적으로 조정할 수 있으며,
3. 요소의 삽입/삭제 시, 배열 대비 효율적이다.
   - 특히, 리스트의 시작/끝에서의 작업이 매우 효율적이다.

## 시간복잡도

| 연산            | 시간복잡도            |
|---------------|------------------|
| 접근(Access)    | $O(n)$           |
| 검색(Search)    | $O(n)$           |
| 삽입(Insertion) | $O(1)$ or $O(n)$ |
| 삭제(Deletion)  | $O(1)$ or $O(n)$ |

- 접근(Access): 특정 인덱스의 요소에 접근하려면, 처음부터 해당 요소까지 순차적으로 탐색해야 하므로 선형 시간이 필요하다.
- 검색(Search): 처음부터 해당 요소까지 순차적으로 탐색해야 하므로 선형 시간이 필요하다.
- 삽입(Insertion):
  - 삽입 위치가 리스트의 시작이나 끝이면 상수 시간이 걸리지만,
  - 중간에 삽입하면 해당 위치까지 도달하는데 선형 시간이 걸린다.
- 삭제(Deletion):
  - 삭제할 요소가 리스트의 시작이나 끝이면 상수 시간에 삭제할 수 있지만,
  - 중간 요소를 삭제할 경우, 해당 위치까지 탐색하는데 선형 시간이 걸린다.

## 연결 리스트의 종류

### 단방향 연결 리스트 (Singly linked list)

<img style="height: 100px;" src="{{ site.url }}{{ site.baseurl }}/assets/images/2022/0721/Singly-linked-list.png"/>

- 모든 노드(요소)가 `다음(next) 노드의 주소`를 가지고 있다.
- 현재 요소에서 `이전 요소로 접근하기 힘들다.`
  - 처음부터 다시 탐색해야 한다.
  - 이 단점을 극복한 것이 아래의 양방향 연결 리스트다.

<br>

### 양방향 연결 리스트 (Doubly linked list)

> **Java에서 Collection Framework의 LinkedList는 Doubly linked list로 구현되어 있다.** 

<img style="height: 140px; padding-top: 10px;" src="{{ site.url }}{{ site.baseurl }}/assets/images/2022/0721/Doubly-linked-list.png"/>

- 모든 노드가 `이전(prev) 및 다음(next) 노드의 주소`를 가지고 있다.
- 단방향 연결 리스트의 이동 방향이 단방향이라는 단점을 보완한 형태다.

<br>

### 양방향 순환 연결 리스트 (Doubly circular linked list)

<img style="height: 180px; padding-top: 10px;" src="{{ site.url }}{{ site.baseurl }}/assets/images/2022/0721/Doubly-circular-linked-list.png"/>  

- 양방향 연결 리스트에서 `첫 번째 노드와 마지막 노드의 연결을 추가`한 형태다.
- 단방향 순환 연결 리스트(Singly circular linked list)도 있지만 잘 사용하지 않아 넘어간다.

<br>
<br>

# References.

[geeksforgeeks: Linked List vs Array](https://www.geeksforgeeks.org/linked-list-vs-array/)  
[Inpa Dev: 자바 LinkedList 구조 & 사용법 - 정복하기](https://inpa.tistory.com/entry/JAVA-%E2%98%95-LinkedList-%EA%B5%AC%EC%A1%B0-%EC%82%AC%EC%9A%A9%EB%B2%95-%EC%99%84%EB%B2%BD-%EC%A0%95%EB%B3%B5%ED%95%98%EA%B8%B0#linkedlist_%EA%B0%9D%EC%B2%B4_%EC%83%9D%EC%84%B1)