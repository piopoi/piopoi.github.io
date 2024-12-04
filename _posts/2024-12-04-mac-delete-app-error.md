---
title: "Mac OS에서 앱 삭제 시 \"일부 항목을 건너뛰어야 하기 때문에 작업을 완료할 수 없습니다.\" 문제 해결 방법"
date: 2024-12-04
categories: knowledge
tags:
  - java
  - concurrency
  - study
---

맥북에서 앱을 삭제하고 휴지통 비우기를 진행했더니 아래와 같은 메시지가 뜨면서 삭제가 되지 않았다.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/2024/1204/241204_01.png" width="70%"/><br><br>

[출처](https://apple.stackexchange.com/questions/412996/folder-wont-delete-from-trash-for-cant-delete-imageitems-because-its-pathn){:target="_blank"} 
페이지 중간을 보면 아래와 같은 해결방법이 나와 있다.

1. 터미널을 연다.
2. `sudo rm -rfv ` 를 입력한다. (**마지막 공백** 필수!!)
3. 휴지통을 연다.
4. 휴지통에서 삭제하려는 항목들을 모두 선택하여 터미널로 드래그한다.
5. 명령어를 실행하고 비밀번호를 입력한다.

순서대로 실행하니 앱이 잘 삭제되었다.

아래는 [출처](https://apple.stackexchange.com/questions/412996/folder-wont-delete-from-trash-for-cant-delete-imageitems-because-its-pathn){:target="_blank"} 
페이지에 소개된 `-rvf` 옵션에 대한 설명을 `claude`를 통해 정리한 내용이다.

```
-f (force)
  - 파일 권한에 상관없이 확인 절차 없이 강제로 삭제를 시도한다.
  - 파일이 존재하지 않아도 오류 메시지를 표시하지 않는다.
  - 이전의 -i 옵션(삭제 전 확인)을 무시한다.

-R 또는 -r (recursive)
  - 지정된 디렉토리와 그 하위의 모든 파일과 디렉토리를 삭제한다.
  - -d 옵션을 포함한다.
  - -i 옵션과 함께 사용하면 각 디렉토리의 내용을 처리하기 전에 사용자 확인을 요청한다.
  - 사용자가 확인에 동의하지 않으면 해당 디렉토리와 그 하위 항목들은 건너뛴다.

-v (verbose)
  - 삭제되는 파일들의 목록을 상세히 보여준다.
```

<br>

Mac OS는 뭔가 문제가 생기면 터미널을 통해 해결되는 경우가 많다.  
개발자 입장에서는 편하지만, 터미널 사용에 익숙하지 않은 사용자에게는 불친절한 OS라는 생각이 든다.

## References.

[https://apple.stackexchange.com/questions/412996/folder-wont-delete-from-trash-for-cant-delete-imageitems-because-its-pathn](https://apple.stackexchange.com/questions/412996/folder-wont-delete-from-trash-for-cant-delete-imageitems-because-its-pathn){:target="_blank"}<br>
[claude.ai](https://claude.ai){:target="_blank"}<br>