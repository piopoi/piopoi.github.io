---
title: "템플릿 메소드 패턴 (Template method pattern)"
date: 2021-08-25
categories: 
  - Programming
tags:
  - Design pattern
toc: true
toc_sticky: true
#use_math: true
---
<br>

# Description

알고리즘의 구조를 변경하지 않고 알고리즘의 특정 단계들을 서브 클래스에서 다시 정의할 수 있게 하는 디자인 패턴이다.

- 템플릿 메소드(template method)
  - 필수 처리 절차를 정의한 메소드
  - 서브클래스가 오버라이드하는 추상 메소드들을 사용하여 알고리즘을 정의한다.
  - 가급적이면 템플릿 메소드의 오버라이딩을 막기 위해 final로 선언하는게 좋다.
- 훅 메소드(hook method)
  - 추상 클래스에서 선언하지만 기본적인 내용만 구현되어 있거나 내용이 비어있는 메소드.
  - 필요한 경우 서브클래스에서 확장(오버라이드)할 수 있다.
- 서브클래스에 오버라이드해야 하는 메소드가 많아지지 않도록 주의하자. 
  - 추상 메소드가 많아질수록 서브클래스를 구현하기 어려워진다.

<br>

# Diagram

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/2021/0825/Template_Method_UML_class_diagram.svg" width="100%"/><br>

<br>

# Example

```java
abstract class Human {
    //템플릿 메소드(template method)
    //오버라이딩을 막기 위한 final.
    final void work() {
        goToWork();
        doWork();
        if (needOvertimeWork()) {
            doOvertimeWork();
        }
        leaveWork();
    }

    protected abstract void doWork();
    
    private void goToWork() {
        System.out.println("출근을 한다.");
    }
    private void doOvertimeWork() {
        System.out.println("야근을 한다.");
    }
    private void leaveWork() {
        System.out.println("퇴근을 한다.");
    }
    
    //훅 메소드(hook method)
    protected boolean needOvertimeWork() {
        return false;
    }
}

class Fireman extends Human {
    @Override
    protected void doWork() {
        System.out.println("불을 끈다.");
    }
}

class PoliceOfficer extends Human {
    @Override
    protected void doWork() {
        System.out.println("사건을 해결한다.");
    }

    //훅 메소드 오버라이딩
    @Override
    protected boolean needOvertimeWork() {
        return true;
    }
}

public class TemplateMethodPattern {
    public static void main(String[] args) {
        Human fireman = new Fireman();
        Human policeOfficer = new PoliceOfficer();

        System.out.println("소방관:");
        fireman.work();

        System.out.println("\n경찰관:");
        policeOfficer.work();
    }
}
```
<br>

실행 결과
```bash
소방관:
출근을 한다.
불을 끈다.
퇴근을 한다.

경찰관:
출근을 한다.
사건을 해결한다.
야근을 한다.
퇴근을 한다.
```
<br>
<br>

# References.
[https://johngrib.github.io/wiki/pattern/template-method/](https://johngrib.github.io/wiki/pattern/template-method/)
[https://en.wikipedia.org/wiki/File:Template_Method_UML_class_diagram.svg](https://en.wikipedia.org/wiki/File:Template_Method_UML_class_diagram.svg)