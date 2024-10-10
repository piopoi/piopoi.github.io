---
title: "Java의 static 키워드에 대하여"
date: 2022-12-08
categories: knowledge
tags:
  - programming
  - java
  - study
---

자바에서 `static` 키워드는 다양한 상황에서 사용되며, 그 사용법과 의미는 크게 4가지로 분류할 수 있다.  
- static 변수
- static 메서드
- static 초기화 블록
- static 클래스

이 글에서는 위 4가지 static 키워드의 사용법과 의미에 대해 알아보자.

<br>

# 1. Static Variable

`static`으로 선언된 변수는 클래스의 모든 인스턴스가 공유하는 변수로, **클래스 변수**, **정적 변수**라고도 한다.    

static 변수는 JVM의 런타임 데이터 영역(Runtime Data Areas) 내 **메서드 영역(Method Area)**에 저장된다.
이 영역은 Class Area, Static Area라고도 불리며, 클래스 파일의 바이트 코드가 로드되는 곳이다.
static 변수는 클래스 레벨에 속하기 때문에 메서드 영역에 위치한다.

클래스가 로드되면, 메서드 영역에 있는 static 변수는 해당 클래스의 모든 인스턴스에 의해 공유되는 것이 아니라, 클래스 자체에 의해 공유된다.
즉, **static 변수는 개별 인스턴스에 종속되지 않고 클래스 전체에 걸쳐 하나의 공통된 상태를 유지**한다.
이러한 특성 때문에, **한 인스턴스에서 static 변수의 값을 변경하면, 그 변경사항이 클래스의 모든 인스턴스에 영향을 미친다.**

예를 들어, 아래와 같은 `TestClass`가 있다고 하면, `TestClass`의 어떤 인스턴스에서도 `count` 변수에 접근할 수 있다.  
```java
class TestClass {
    static int count = 0;    
}
```
하지만, 이 `count` 변수는 `TestClass`의 모든 인스턴스에 의해 공유되는 것이 아니라,
`TestClass`라는 하나의 클래스에 속해 있으며, 모든 인스턴스가 이 하나의 `count` 변수를 참조한다.  

<br>
<br>

# 2. Static Method

static 메서드는 **인스턴스 생성 없이 호출**할 수 있는 메서드다.  
**오직 static 변수나 다른 static 메서드에만 접근할 수 있으며, 인스턴스 변수에는 접근할 수 없다.**  
이러한 특성으로 인해, static 메서드는 **유틸리티 함수를 작성할 때 유용**하다.

`java.lang.Math` 클래스를 예로 들 수 있다.  
`Math` 클래스의 모든 메서드는 static 메서드로, `Math`의 인스턴스를 생성하지 않고도 바로 사용할 수 있다.
```java
int randomNumber = (int)(Math.random() * 6); // 0 ~ 5
```

static 메서드 역시 JVM의 메서드 영역에 저장된다.  
이 메서드들은 인스턴스 생성 없이 호출되며, JVM은 클래스가 로딩될 때 이 메서드들을 로드하고 관리한다.  
이러한 특징은 위에서 말한 것 처럼 공유 가능한 유틸리티 함수를 만드는 데 이상적이다.

<br>
<br>

# 3. Static Initialization Block

**클래스가 JVM에 로드될 때 단 한 번만 실행**되는 static 블록은 **주로 static 변수를 초기화하는 데 사용**된다.  
이 블록은 **클래스의 생성자보다 먼저 실행**되며, 클래스 로딩 시에 필요한 준비 작업을 수행하는 데 유용하다.

## Initialization Block

초기화 블록(Initialization Block)은 `static` 여부에 따라 2가지로 나뉜다.
- Instance Block (Non-static Initialization Block)
- Static Block (Static Initialization Block)

```java
public class Example {
    {
        // Instance Block
    }
  
    static {
        // Static Block
    }
}
```

### Instance Block
- = `Non-static` Initialization Block
- 인스턴스 블록은 클래스의 각 인스턴스가 생성될 때마다 실행된다.
- 객체 생성 과정에서 객체의 필드나 상태를 초기화하는 데 사용된다.
- 생성자보다 먼저 실행되지만, 생성자와 마찬가지로 객체당 한 번만 실행된다.
- `{ }`(중괄호)로 정의되며, `static` 키워드 없이 작성된다.

### Static Block 
- = `Static` Initialization Block
- 클래스가 JVM에 로드될 때 단 한 번만 실행된다.
- 주로 클래스 레벨의 데이터 또는 정적 필드(static fields)를 초기화하는 데 사용된다.
- 클래스의 모든 인스턴스에 공통적인 초기 설정을 제공한다.
- `static` 키워드와 `{ }`(중괄호)로 정의된다.

<br>
<br>

# 4. Static Inner Class

정적 내부 클래스(Static Inner Class)라고도 하는 static으로 선언된 내부 클래스는 외부 클래스의 인스턴스 없이도 사용할 수 있다.    

- 외부 클래스의 인스턴스 멤버에 직접 접근할 수 없다.
- 외부 클래스와 무관하게 로드되고 인스턴스화 된다.

JVM은 정적 내부 클래스를 **외부 클래스와 독립적으로 취급**하며, 이는 메모리 사용을 최적화하고 클래스 간의 결합도를 낮추는 데 유용하다.

## 내부 클래스를 static으로 선언해야 하는 이유

> 참조: [내부 클래스는 static 으로 선언 안하면 큰일 난다](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EC%9E%90%EB%B0%94%EC%9D%98-%EB%82%B4%EB%B6%80-%ED%81%B4%EB%9E%98%EC%8A%A4%EB%8A%94-static-%EC%9C%BC%EB%A1%9C-%EC%84%A0%EC%96%B8%ED%95%98%EC%9E%90#inner_%ED%81%B4%EB%9E%98%EC%8A%A4%EC%9D%98_%EB%A9%94%EB%AA%A8%EB%A6%AC_%EB%88%84%EC%88%98_%ED%98%84%EC%83%81)

### 비 정적 내부 클래스는 외부 클래스를 참조한다.

```java
public class OuterClass {
    int outerField = 1;

    class InnerClass {
        int innerField = 2;
    }
}
```

위와 같이 내부 클래스(Inner Class)를 가진 외부 클래스(Outer Class)가 있다고 하자.

이 코드를 컴파일 후 바이트코드 파일(OuterClass$InnerClass.class)을 Decompile 하면,  
아래와 같이 **내부 클래스의 생성자에서 외부 클래스를 매개변수로 받아 내부 클래스의 인스턴스 변수로 저장**하는 것을 볼 수 있다.

```java
class OuterClass$InnerClass {
    int innerField;
    final OuterClass this$0; //외부 클래스를 저장할 인스턴스 변수
  
    OuterClass$InnerClass(OuterClass this$0) { //외부 클래스를 매개변수로 받는다.
        this.this$0 = this$0;
        this.innerField = 2;
    }
}
```

결과적으로, Non-static Inner Class의 인스턴스는 Outer Class의 인스턴스를 참조하게 되어,  
`this` 키워드를 이용하여 Outer Class의 인스턴스를 사용할 수 있다.

```java
public class OuterClass {
    int outerField = 1;
    
    int getField() {
        return outerField;
    }

    class InnerClass {
        int innerField = 2;
        
        int doSomething() {
            return OuterClass.this.getField(); //this 사용 예
        }
    }
}
```

### 외부 참조로 인한 메모리 누수

하지만, 이 **외부 참조로 인해 메모리 누수라는 치명적인 문제가 발생**한다.

Java에서 GC(가비지 컬렉션)의 대상이 되는 객체를 선정할 때,
Heap Area 안의 객체들 중 해당 객체가 다른 객체에 의해 참조되고 있는지의 여부를 기준으로 삼는다.

Non-static Inner Class의 경우, 외부 클래스가 필요없더라도 내부 클래스의 인스턴스를 생성하려면 외부 클래스의 인스턴스 생성이 필요하다.
이 경우 **외부 클래스의 인스턴스를 내부 클래스가 참조하고 있기 때문에, 외부 클래스가 필요 없더라도 GC의 대상이 되지 못한다.**  

### 결론: 정적 내부 클래스를 사용하자

정적 내부 클래스(Static Inner Class)의 경우 비 정적 내부 클래스와 같은 외부 참조가 없기 때문에,
내부 클래스의 인스턴스만 독립적으로 생성하여 사용할 수 있다.

결론적으로, **외부 클래스의 인스턴스 변수나 메서드를 내부 클래스에서 사용해야 하는 경우가 아니면, 정적 내부 클래스를 사용하자.**

<br>
<br>

# References.

[[Java] static 변수, static 메서드 그리고 static 클래스](https://velog.io/@mooh2jj/Java-static-%EB%B3%80%EC%88%98-static-%EB%A9%94%EC%84%9C%EB%93%9C-%EA%B7%B8%EB%A6%AC%EA%B3%A0-static-%ED%81%B4%EB%9E%98%EC%8A%A4)  
[내부 클래스는 static 으로 선언 안하면 큰일 난다](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EC%9E%90%EB%B0%94%EC%9D%98-%EB%82%B4%EB%B6%80-%ED%81%B4%EB%9E%98%EC%8A%A4%EB%8A%94-static-%EC%9C%BC%EB%A1%9C-%EC%84%A0%EC%96%B8%ED%95%98%EC%9E%90)  
[JVM 내부 구조 & 메모리 영역 💯 총정리](https://inpa.tistory.com/entry/JAVA-%E2%98%95-JVM-%EB%82%B4%EB%B6%80-%EA%B5%AC%EC%A1%B0-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EC%98%81%EC%97%AD-%EC%8B%AC%ED%99%94%ED%8E%B8#%EC%9E%90%EB%B0%94_%EA%B0%80%EC%83%81_%EB%A8%B8%EC%8B%A0jvm%EC%9D%98_%EB%8F%99%EC%9E%91_%EB%B0%A9%EC%8B%9D)