<!-- -
![](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_14_Part/django.png)
-->
# 마크다운 기능 적용하기
파이보는 게시판 서비스이므로 질문 또는 답변을 작성할 때 일반 텍스트 형식으로 글을 작성하면 매력이 떨어진다. 예를 들어 글자를 진하게 표시(볼드)하거나 링크를 추가하고 싶을 수도 있다. 이런 경우 사용하면 좋은 도구가 바로 '마크다운'이다. 마크다운을 이용하면 간단한 문법으로 문서를 여러 형태로 표시할 수 있다. 여기서는 마크다운 문법을 간단히 설명하고, 파이보에 마크다운 기능을 적용하는 방법까지 알아보겠다.

## 마크다운이 뭐죠? 문법 알아보기!
마크다운 문법은 아주 간단하다. 여기서는 마크다운의 주요 문법을 간단히 알아보려고 한다. 참고로 마크다운 문법이 원하는 형태로 표시되려면 그 문법을 해석하고 표현하는 해석기를 설치해야 한다. 지금은 단순히 마크다운 문법만 설명하고 파이보에서는 해석기까지 설치하여 실제 표시되는 과정까지 실습해 보자.
> 마크다운 문법은 프로그래밍이 아니라 아주 간단한 텍스트 입력 규칙이다. 실습하면서 알아보자.
> 마크다운은 블로그, 깃허브 등 많은 사이트에서 사용하는 글쓰기 도구이다. 배워 두면 크게 쓸모가 있다.

마크다운 문법은 이미 알고있으므로

...생략...

## 파이보에 마크다운 기능 추가히기

1. markdown 설치하기
파이보에 마크다운 기능을 추가하려면 pip install markdown으로 마크다운을 설치해야한다.
```
(mysite) D:\code\vscode\github\jump_to_django\src\projects\mysite>pip install markdown
Collecting markdown
  Downloading Markdown-3.3.6-py3-none-any.whl (97 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 97.8/97.8 KB 5.5 MB/s eta 0:00:00
Collecting importlib-metadata>=4.4
  Downloading importlib_metadata-4.11.1-py3-none-any.whl (17 kB)
Collecting zipp>=0.5
  Using cached zipp-3.7.0-py3-none-any.whl (5.3 kB)
Installing collected packages: zipp, importlib-metadata, markdown
Successfully installed importlib-metadata-4.11.1 markdown-3.3.6 zipp-3.7.0
```

2. 마크다운 필터 등록하기
마크다운으로 작성한 문서를 HTML 문서로 변환하려면 템플릿에서 사용할 마크다운 필터를 작성해야 한다. 이전에 sub 필터를 작생했던 pybo_filter.py 파일에 mark 함수를 추가하자.
```
import markdown
from django import template
from django.utils.safestring import mark_safe

register = template.Library()

@register.filter
def sub(value, arg):
    return value - arg

@register.filter()
def mark(value):
    extensions = ["nl2br", "fenced_code"]
    return mark_safe(markdown.markdown(value, extensions=extensions))
```
mark 함수는 markdown 모듈과 mark_safe 함수를 이용하여 문자열을 HTML 코드로 변환하여 반환한다. 이 과정을 거치면 마크다운 문법에 맞도록 HTML이 만들어진다. 그리고 markdown 모듈에 "nl2br", "fenced_code"확장 도구를 설정했다. "nl2br"은 줄바꿈 문자를 <br> 태그로 바꿔 주므로 Enter를 한 번만 눌러도 줄바꿈으로 인식한다. 만약 이 확장 도구를 사용하지 않으면 줄바꿈을 위해 줄 끝에 마크다운 문법인 스페이스를 2개를 연속으로 입력해야 할 것이다. "fenced_code"는 마크다운의 소스 코드 표현을 위해 적용했다.
> 마크다운 확장 기능은 다음 문서를 참고하자.
> 마크다운 확장 기능 문서: https://python-markdown.github.io/extensions

3. 질문 상세 템플릿에 마크다운 적용해 보기
질문 상세 템플릿에 {% load pybo_filter %}을 추가하여 마크다운 필터를 적용하자.
```
{% extends 'base.html' %}
{% load pybo_filter %}
{% block content %}
<div class="container my-3">
(... 생략 ...)
<div class="card-text" style="white-space: pre-line;">{{ question.content|mark }}</div>
(... 생략 ...)
```
기존의 style="white-space: pre-line;"은 삭제하고 {{ question.content|mark }}와 같이 마크다운 필터 mark를 적용했다.

4. 답변 내용에 마크다운 적용해 보기
답변 내용에도 마크다운을 적용하자.
```
(... 생략 ...)
<div class="card-text" style="white-space: pre-line;">{{ answer.content|mark }}</div>
(... 생략 ...)
```
이제 질문 또는 답변을 마크다운 문법으로 작성하면 웹 브라우저에서 어떻게 보이는지 확인해 보자. 다음 내용으로 글을 작성해 보자.

```
**마크다운 문법으로 작성해 봅니다.**

* 리스트1
* 리스트2
* 리스트3

파이썬 홈페이지는 [http://www.python.org](http://www.python.org) 입니다.
```

![3-14_1.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_14_Part/3-14_1.png)

> 마크다운 문법을 몰라도 simplemde와 같은 마크다운 UI 도구를 설치하면 마크다운을 쉽게 사용할 수 있다.