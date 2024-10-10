---
title: "빌더 패턴 (Builder pattern)"
date: 2020-08-14
categories: knowledge
tags:
  - programming
  - design pattern
---
<br>

```java
Post post = Post.builder()
    .title("제목입니다.")
    .content("내용입니다.")
    .build();
```

객체를 생성할 때 흔하게 사용하는 패턴이다.
전 직장의 코드에서는 보지 못했지만, 2022년에 우아한 테크 캠프 Pro에 참가하면서 제공된 코드나 다른 교육생들의 코드를 보며 정말 많이 쓴다는 것을 알게 되었다.

빌더 패턴에 대해 알게된 후 한동안 전부 생성자에 lombok의 @Builder를 붙여서 객체를 생성하였다. 스프링에서 클래스 선언부 위에 @Builder를 선언하게 되면 파라미터 필요 유무에 따른 생성자 오버로딩을 별도로 구현할 필요가 없고, 매우 쉽게 적용할 수 있어 개발자 입장에서 매우 편리함을 느껴서 그랬던 것 같다.  

캡슐화에 대한 고민으로 현재는 도메인 객체 생성 시 클래스 선언부에 @Builder를 사용하지 않고, **정적 팩토리 메서드**를 주로 사용하고 있다.

<br>

# Description

빌더 패턴은 복합 객체의 생성 과정과 표현 방법을 분리하여 동일한 생성 절차에서 서로 다른 표현 결과를 만들 수 있게 하는 패턴이다.   

이펙티브 자바에 따르면, 생성자의 파라미터가 많아질수록 코드 가독성이 떨어지고 수정하기 힘들어지는 점층적 생성자 패턴의 대안으로 자바 빈 패턴이 나왔지만, 자바 빈 패턴은 setter를 public으로 정의하여 immutable한 객체를 생성할 수 없게되어 빌더 패턴이 그 대안이 되었다고 한다.

<br>

# Example

```java
public class Post {
    private final String title;
    private final String content;
    private final String author;

    private Post(PostBuilder postBuilder) {
        title = postBuilder.title;
        content = postBuilder.content;
        author = postBuilder.author;
    }

    public static PostBuilder builder() {
        return new PostBuilder();
    }

    public static class PostBuilder {
        private String title;
        private String content;
        private String author;

        private PostBuilder() {
        }

        public PostBuilder title(String title) {
            this.title = title;
            return this;
        }

        public PostBuilder content(String content) {
            this.content = content;
            return this;
        }

        public PostBuilder author(String author) {
            this.author = author;
            return this;
        }

        public Post build() {
            return new Post(this);
        }
    }
}
```
<br>

객체 생성
```java
Post post = Post.builder()
    .title("제목")
    .content("내용")
    .author("작성자")
    .build(); 
```

<br>
<br>

# References
[https://github.com/greekZorba/java-design-patterns/tree/master/builder](https://github.com/greekZorba/java-design-patterns/tree/master/builder)
[https://en.wikipedia.org/wiki/Builder_pattern](https://en.wikipedia.org/wiki/Builder_pattern)
[https://johngrib.github.io/wiki/pattern/builder/](https://johngrib.github.io/wiki/pattern/builder/)