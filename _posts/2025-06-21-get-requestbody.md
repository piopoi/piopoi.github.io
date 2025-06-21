---
title: "SI 프로젝트 회고 01: HTTP GET Method는 Request Body를 사용하지 않는다."
date: 2025-06-21
categories: experience
tags:
  - web
  - project
---

최근 B2B 백오피스 지불시스템 구축 프로젝트(SI)에 개발자로 참여하고 있다.
이번 주에 있었던 명확한 규칙을 잊어버리고 실수를 했던 경험을 기록해본다.

이번에 관리자 페이지의 기능을 개발하면서,
GET 메서드의 컨트롤러를 작성하고, 파라미터를 DTO 객체 형태로 받기 위해 `@RequestBody`를 사용했다.
프론트엔드 쪽에서도 jQuery의 Ajax를 이용하여 요청을 보낼 때 파라미터를 Body에 담아 보냈는데, 400 Bad Request 에러가 발생했다.
이상하게도 컨트롤러에서 디버깅을 위해 설정한 브레이크 포인트가 전혀 걸리지 않았다.

처음엔 프론트 쪽 Ajax 코드나 URL 문제를 의심했지만, 빠르게 확인해보니 요청 자체는 정상적으로 서버에 도달하고 있었다.
문제는 역시 Spring 컨트롤러의 파라미터 매핑 단계에서 발생하고 있었다.

문제의 원인은 간단했지만 잠시 잊고 있었던 HTTP 스펙 때문이었다.

GET 요청은 Request Body를 포함하지 않는 것이 HTTP 스펙의 표준이다.
요청 메시지의 body가 비어 있었기 때문에, Spring에서는 `@RequestBody`가 붙은 메서드에 진입하기도 전에 매핑 오류가 발생하고, 이로 인해 컨트롤러 내부 로직은 실행되지 않는다.
다시 말해, Spring의 `@RequestBody`는 기본적으로 POST, PUT, PATCH와 같은 메서드에서만 사용하는 것이 원칙이고,
GET 방식에서는 무시되거나 매핑 오류를 일으킨다.

더 자세히 얘기하면 아래와 같다.

- HTTP/1.1 표준(RFC 7231) 상에서는 GET 요청에 body가 없다고 가정하며, 서버는 이를 무시하거나 지원하지 않아도 되는 것으로 본다.
- jQuery의 `$.ajax()`는 내부적으로 **XMLHttpRequest**를 사용한다.
- XMLHttpRequest의 스펙을 보면 아래와 같은 내용이 있다.
  - "If this’s request method is `GET` or `HEAD`, then set body to null."
  - 즉, **GET이나 HEAD 요청에서는 `send()` 메서드에 어떤 데이터를 전달해도 무시되고 body가 null로 설정된다**는 내용이다.
  - 출처: [XMLHttpRequest Living Standard — Last Updated 11 March 2025](https://xhr.spec.whatwg.org/)
- jQuery의 `$.ajax()`를 사용하면 XMLHttpRequest 객체를 직접 다루지 않기 때문에 `send()`를 명시적으로 호출하지는 않지만, 내부적으로는 XMLHttpRequest의 `send()`를 호출하게 된다.
- 결론: **jQuery의 `$.ajax()`로 GET 요청을 하면 Request Body가 null로 설정된다.**

이런 HTTP Method의 표준 스펙에 대해서는 2024년에 스터디에서 주제로 다뤄진 적이 있었음에도 잊고 있었다.
다행히도 디버깅 도중에 이 내용이 생각나서, 컨트롤러를 수정하여 파라미터를 `@RequestParam`으로 바꾸고,
Ajax 요청에서도 데이터를 Query String 형태로 전달하도록 수정하니 정상적으로 작동했다.

돌이켜 보면 프로젝트 초반의 속도를 내기 위해 서둘렀던 것이 이런 사소한 문제를 놓치는 원인이 된 것 같다.
HTTP 메서드의 특성과 Spring Framework의 기본 동작 원칙을 잊지 않도록 더 꼼꼼히 신경 써야겠다고 다짐하게 되는 계기였다.
