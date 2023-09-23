---
title: "try-with-resources"
date: 2020-07-12
categories: 
  - Programming
tags:
  - Java
toc: true
toc_sticky: true
#use_math: true
---

# try-with-resources (vs try-catch-finally)

try-catch-finally의 문제점을 보완하기 위해 try-with-resources가 나왔다.  
try(…) 안에 자원 객체를 전달하면, try 블록이 끝나고 **자동으로 자원을 해제**해주는 기능이 핵심이다.  

## 기존의 try-catch-finally를 사용한 경우

```java
FileOutputStream fos = null;
try {
    fos = new FileOutputStream("example.txt");
    String data = "Hello, World!";
    fos.write(data.getBytes());
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if (fos != null) {
        try {
            fos.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## try-with-resources를 사용한 경우

```java
try (FileOutputStream fos = new FileOutputStream("example.txt")) {
    String data = "Hello, World!";
    fos.write(data.getBytes());
} catch (IOException e) {
    e.printStackTrace();
}
```

## 비교
- `try-with-resources` : 자원을 닫는 코드를 명시적으로 작성할 필요가 없다. Java가 알아서 처리해준다.
- `try-catch-finally` : 명시적으로 자원을 닫고, 이 과정에서 필요한 exception 처리를 추가로 해줘야 한다.

<br>

# Interface AutoCloseable

`try-with-resources`을 사용하여 자원을 자동으로 해제하기 위해서는 해당 자원에 AutoCloseable 인터페이스를 Implements 해야 한다.  

- AutoClosable 인터페이스 내에는 close() 메소드만 존재한다. [(Document)](https://docs.oracle.com/javase/8/docs/api/java/lang/AutoCloseable.html)

`try-with-resources`의 `try` 블록이 종료될 때, 오버라이딩한 close() 메소드가 자동으로 호출되어 자원이 해제된다. 

## Sample Code
```java
public class TryWithResources {
    public static void main(String[] args) {
        
        try (Resource resource = new Resource()) {
            resource.doSomething();
        } catch (Exception e) {
            e.printStackTrace();
        }
        
    }
}

class Resource implements AutoCloseable {

    public void doSomething() {
        System.out.println("call: MyResource.doSomething()");
    }

    @Override
    public void close() throws Exception {
        System.out.println("call: MyResource.close()");
    }
}
```

## Result
```bash
call: MyResource.doSomething()
call: MyResource.close()
```

<br>

# 결론
가능하면 `try-with-resources`를 사용하자


<br>
<br>

# References.
[https://docs.oracle.com/javase/8/docs/api/java/lang/AutoCloseable.html](https://docs.oracle.com/javase/8/docs/api/java/lang/AutoCloseable.html)  