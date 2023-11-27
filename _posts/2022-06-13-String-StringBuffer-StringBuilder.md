---
title: "String, StringBuffer, StringBuilder"
date: 2022-06-13
categories: 
  - Programming
tags:
  - java
---

Java에서는 문자열을 표현하는 3가지 클래스인 String, StringBuffer, StringBuilder를 제공한다.  
문자열 생성을 위해서 언제 어떤 클래스를 사용해야하는지 알아보자.

<br>

# 간단 비교

| Index        | String               | StringBuffer | StringBuilder |
|--------------|----------------------|--------------|---------------|
| Storage Area | Constant String Pool | Heap         | Heap          |
| Modifiable   | Immutable            | Mutable      | Mutable       |
| Thread Safe  | Yes                  | Yes          | No            |
| Performance  | Fast                 | Very Slow    | Fast          |

<br>

# String

- `immutable`

String은 `immutable`인 반면 StringBuffer와 StringBuilder는 `mutable`이다.   
따라서, 콘텐츠가 고정되어 있거나 자주 변경되지 않는다면, String을 사용하는 것이 좋다.  

콘텐츠가 자주 변경되지 않는다면, String이 다른 두 클래스보다 메모리 효율이 더 높다.

<br>

# StringBuffer, StringBuilder

- `mutable`

StringBuffer와 StringBuilder는 String과 반대로 `mutable`하다.  
따라서, 콘텐츠가 고정되어 있지 않고 자주 변경된다면, StringBuffer나 StringBuilder를 사용하는 것이 좋다.

그럼 두 클래스의 차이는 무엇일까?

## StringBuffer

- `thread-safe`

StringBuffer는 `thread-safe`하여 여러 thread가 동시에 접근할 수 없다.  
따라서, `thread-safe`한 특성이 필요하면, StringBuffer를 사용하는 것이 좋다.

다만, StringBuffer는 동기화 관련 처리가 필요하여, StringBuilder보다 성능이 떨어진다.

## StringBuilder

- `non-thread-safe`

StringBuilder는 `non-thread-safe`하다.  
따라서, `thread-safe`한 특성이 필요하지 않다면, StringBuilder를 사용해야 한다.

StringBuilder는 동기화 관련 처리가 필요하지 않아서, StringBuffer보다 성능이 좋다. 

<br>
<br>

# 성능 테스트: StringBuffer vs StringBuilder

StringBuffer와 StringBuilder 객체에 문자 하나를 반복문으로 append하는 테스트를 해보았다.  
아래 측정결과를 보면 예상보다 성능차이가 꽤 나는 것을 볼 수 있다.  
성능 및 동기화 필요여부를 고려하여 상황에 맞는 클래스를 사용하자.

## 소요시간 측정결과
- 1만번 append
  - StringBuffer : 2ms
  - StringBuilder : 0ms
- 100만번 append
  - StringBuffer : 12ms
  - StringBuilder : 5ms

## Test Code
```java
public class Test {
    public static void main(String[] args) {

        //StringBuffer
        long startAt = System.currentTimeMillis();
        StringBuffer b = new StringBuffer("init");
        for (int i = 0; i < 1000000; i++) {
            b.append("a");
        }
        long endAt = System.currentTimeMillis();
        System.out.println("StringBuffer 소요시간(ms) = " + (endAt - startAt));

        //StringBuilder
        startAt = System.currentTimeMillis();
        StringBuilder c = new StringBuilder("init");
        for (int i = 0; i < 1000000; i++) {
            c.append("a");
        }
        endAt = System.currentTimeMillis();
        System.out.println("StringBuilder 소요시간(ms) = " + (endAt - startAt));
    }
}
```

## Output
```
StringBuffer 소요시간(ms) = 12
StringBuilder 소요시간(ms) = 5
```

<br>
<br>

# References.
[https://medium.com/@257ramanrb/string-vs-stringbuilder-vs-stringbuffer-935d170141a9](https://medium.com/@257ramanrb/string-vs-stringbuilder-vs-stringbuffer-935d170141a9)
[https://incheol-jung.gitbook.io/docs/q-and-a/java/string-stringbuffer-stringbuilder](https://incheol-jung.gitbook.io/docs/q-and-a/java/string-stringbuffer-stringbuilder)