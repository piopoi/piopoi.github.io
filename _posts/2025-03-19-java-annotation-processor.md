---
title: "Java Compiler의 Annotation Processor"
date: 2025-03-19
categories: knowledge
tags:
  - java
---

개발 커뮤니티에 Annotation Processor 관련 질문이 올라왔는데, QueryDSL을 설정할 때 관련 설정을 추가했던 것이 기억이 났다.

```
dependencies {
    ...

    //QueryDSL
   implementation 'com.querydsl:querydsl-jpa:5.0.0'
   annotationProcessor "com.querydsl:querydsl-apt:5.0.0:jpa"
   annotationProcessor "jakarta.annotation:jakarta.annotation-api"
   annotationProcessor "jakarta.persistence:jakarta.persistence-api"

    ...
}
```

QueryDSL이 QClass를 생성할 때 annotationProcessor가 필요하다는 것만 알고 어떻게 동작하는지는 몰라서 이 포스트를 통해 간단하게 정리해본다.

# Annotation Processor

Annotation Processor는 자바 컴파일 단계에서 `@Entity`, `@Getter`, `@Autowired` 같은 Annotation을 읽고 처리해주는 도구다.
필요한 경우, 새로운 소스 파일(.java)이나 리소스를 자동 생성하기도 한다. 쉽게 말해, "Annotation 보고 필요한 코드를 만들어주는 자동 코드 생성기"라고 보면 된다.

- Annotation Processor는 컴파일 시 애너테이션을 읽고 필요한 코드를 자동으로 만들어주는 컴파일러 도구다.
- 자바 컴파일러가 코드를 컴파일하는 동안 어노테이션 프로세서를 실행한다.
- gradle은 `annotationProcessor` 의존성 설정을 통해 쉽게 이 기능을 사용할 수 있게 도와준다.

## 주요 역할

- Annotation 읽기: 소스 코드에서 특정 Annotation을 찾아서 처리.
- 검증 수행: 잘못된 Annotation 사용을 컴파일 단계에서 검증.
- 새 소스 생성: QUser.java와 같은 소스 코드 생성(QueryDSL).
- 리소스 생성: 설정 파일, JSON 파일 등을 생성.

## 대표적인 예시

- Lombok: `@Getter`를 달면, 자동으로 getter 메서드 생성.
- QueryDSL: `@Entity`를 보고 QUser.java 같은 클래스 생성.

## 왜 사용할까?

- 반복적인 코드 작성을 최소화한다.
- 사람이 작성하면 실수할 수 있는 부분을 자동으로 생성해준다.
- 결국 코드 가독성이 좋아지고, 생산성이 올라간다.

# Annotation Processing 동작 과정

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/2025/20250319_01.png" width="100%"/><br><br>

1. 소스 코드 파싱
   - 컴파일러가 .java 파일을 읽어 **구문 트리(Syntax Tree)**로 변환한다.
   - @Entity, @Getter, @Autowired 같은 **애너테이션도 함께 읽어서 기억**한다.
2. 애너테이션 프로세서 실행
   - **등록된 Annotation Processor**들이 실행된다.
   - 각 프로세서는 자신이 처리할 애너테이션이 붙은 클래스, 메서드, 필드를 찾아 작업한다.
   - 예시: `@Entity` → QueryDSL Processor 동작
   - 예시: `@Getter` → Lombok Processor 동작
3. 소스 코드 또는 파일 생성
   - 필요한 경우 **새로운 .java 파일이나 리소스 파일을 생성**한다.
   - QueryDSL: QUser.java
   - Lombok: User.getName() 메서드 생성.
   - 새로 생성된 파일이 있다면 **컴파일 대상 목록에 추가**.
4. 재컴파일 여부 판단
   - **새로 생성된 파일이 있다면 처음부터 다시 파싱 단계로 돌아가서 반복**한다.
   - 더 이상 생성할 파일이 없으면 다음 단계로 넘어감.
5. 최종 컴파일 및 클래스 파일 생성
   - 최종적으로 모든 .java 소스를 .class 파일로 변환한다.
   - 이때 생성된 **Q 클래스나 Lombok 메서드도 포함되어 컴파일**됨.
