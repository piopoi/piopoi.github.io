---
title: "싱글톤 패턴 (Singleton pattern)"
date: 2020-06-14
categories: 
  - Programming
tags:
  - Design pattern
toc: true
toc_sticky: true
#use_math: true
---
<br>

클래스의 인스턴스가 1개만 생성되는 것을 보장하는 디자인 패턴.

<a href="https://ko.wikipedia.org/wiki/%EC%8B%B1%EA%B8%80%ED%84%B4_%ED%8C%A8%ED%84%B4" target="_blank">
  <img src="https://upload.wikimedia.org/wikipedia/commons/f/fb/Singleton_UML_class_diagram.svg">
</a>

<br>

# 1. LazyHolder 방식

```java
public class LazyHolderSingleton {

    private LazyHolderSingleton() {
    }

    private static class LazyHolder {
        private static final LazyHolderSingleton INSTANCE = new LazyHolderSingleton();
    }

    public static LazyHolderSingleton getInstance() {
        return LazyHolder.INSTANCE;
    }
}
```

- 가장 많이 사용되는 싱글톤 구현 방식이다.
- volatile이나 synchronized 키워드 없이 동시성 문제를 해결했다.
- 클래스 로더가 LazyHolderSingleton 클래스를 초기화 할 때 LazyHolder 클래스는 초기화되지 않고, getInstance()를 호출할 때 초기화 되어 인스턴스를 생성한다.
- 즉, **동적 바인딩**(Dynimic Binding, 런타임 시 성격이 결정)의 특징을 이용하여 Thread-safe하면서 성능이 뛰어나다.

<br>

# 2. DCL(Double-checked locking) 방식
```java
public class DCLSingleton {

    private volatile static DCLSingleton instance;

    private DCLSingleton() {
    }

    public DCLSingleton getInstance() {
        if (instance == null) {
            synchronized (DCLSingleton.class) {
                if (instance == null) {
                    instance = new DCLSingleton();
                }
            }
        }
        return instance;
    }
}
```

- 인스턴스가 생성되지 않은 경우에만 동기화 블럭 실행 

## volatile 키워드가 필요한 이유
  - 스레드가 변수값을 읽어올 때 CPU Cache에서 읽어오게 되는데, **멀티 스레드 환경에서는 각각의 CPU Cache에 저장된 값이 다르기 때문**이다.
  - 변수를 volatile로 선언 시 CPU Cache가 아닌 **Main Memory에 값을 read/write**하기 때문에 변수 값 불일치가 생기지 않는다.
  - 멀티 스레드 환경에서 여러개의 스레드가 write하는 상황이라면, 동기화 블럭(synchronized)을 지정해서 **원자성(atomic)**을 보장해야 한다.

<br>

# 3. 싱글톤 사용시 주의사항

- 클래스 로더를 2개 이상 사용하는 경우, 인스턴스가 2개 이상 생성될 수 있다. 이런 경우에는 클래스 로더를 지정해야 한다.
- 자바와 스프링의 싱글톤 차이점은, 싱글톤 객체의 생명주기가 다르다는 점이다. 또한, 자바에서 범위는 클래스 로더가 기준이지만, 스프링에서는 어플리케이션 컨텍스트(ApplicationContext)가 기준이 된다.
- 클래스 로더 기준이라는 것은 톰캣이 WAR 파일을 만들게 되면, WAR파일 하나 당 클래 로더 하나가 1:1 식으로 배치가 되기 때문에 다른 WAR 파일은 참조가 불가능하다.
- 반면, ApplicationContext 기준이라는 것은 web.xml에서 root context 하나와 servlet context 여러개를 등록할 수 있는데, 이 각각의 context들이 싱글톤의 범위가 된다.

<br>

# 4. Spring Framework의 싱글톤 패턴

- 스프링은 빈을 등록할 때 범위(scope)를 지정할 수 있는데, default가 싱글톤(Singleton)이다.
- 스프링에서 싱글톤 객체를 저장/관리해주는 녀석이 ApplicationContext이다. 이 녀석의 명칭은 Singleton Registry, IOC 컨테이너, 스프링 컨테이너, 빈 팩토리 등으로 불린다.

<br>
<br>

# References.
[https://en.wikipedia.org/wiki/Initialization-on-demand_holder_idiom](https://en.wikipedia.org/wiki/Initialization-on-demand_holder_idiom)  
[https://www.baeldung.com/java-singleton-double-checked-locking](https://www.baeldung.com/java-singleton-double-checked-locking)  
[https://medium.com/webeveloper/%EC%8B%B1%EA%B8%80%ED%84%B4-%ED%8C%A8%ED%84%B4-singleton-pattern-db75ed29c36](https://medium.com/webeveloper/%EC%8B%B1%EA%B8%80%ED%84%B4-%ED%8C%A8%ED%84%B4-singleton-pattern-db75ed29c36)  
[https://blog.javarouka.me/2018/11/20/no-instance/](https://blog.javarouka.me/2018/11/20/no-instance/)  