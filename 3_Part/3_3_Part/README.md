<!-- -
![](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_3_Part/django.png)
-->
# 템플릿 필터 직접 만들어 보기

여기서는 템플릿 필터를 직접 만드는 방법을 알아본다. 템프릿 필터란 템플릿 태그에서 | 문자 뒤에 사용하는 필터를 말한다. 예를 들어 None 대신 공백 문자열로 보여주기 위해 사용했던 default_if_none과 같은 것을 템플릿 필터라고 한다.
```
{{ form.subject.avlue|default_if_none:'' }}
```

## 항상 1로 시작하는 게시물 번호 문제 해결하기

파이보 질문 목록 화면을 유심히 보면 페이지마다 게시물 번호가 항상 1부터 시작되는 문제가 있다. 페이지를 이리저리 이동해 봐도 게시물 번호는 1부터 시작한다. 이 문제를 템플릿 필터를 사용하여 해결해 보자.

![3-03_1.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_3_Part/3-03_1.png)

![3-03_2.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_3_Part/3-03_2.png)

1. 게시물 번호 공식 만들기
만약 질문 게시물이 12개라면 1페이지에는 12번째~3번째 게시물이, 2페이지에는 2번째~1번째 게시물이 역순으로 표시되어야 한다. 질문 게시물의 번호를 역순으로 정렬하려면 다음과 같은 공식을 적용해야 한다.

```
일련번호 = 전체 게시물 개수 - 시작 인덱스 - 현재 인덱스 + 1
```

시작 인덱스는 페이지당 시작되는 게시물의 시작 번호를 의미한다. 예를 들어 페이지당 게시물을 10건씩 보여준다면 1페이지의 시작 인덱스는 1, 2페이지의 시작 인덱스는 11이 된다. 현재 인덱스는 페이지에 보여지는 게시물 개수만큼 0부터 1씩 증가되는 번호이다. 따라서 전체 게시물 개수가 12개이고 페이지당 10건씩 게시물을 보여 준다면 1페이지의 일련번호는 12 - 1 -(0~9 반복) + 1 이 되어 12~3까지 표시되고 2페이지의 경우에는 12 - 11 - (0~1 반복) + 1 이 되어 2~1이 표시될 것이다. 템플릿에서 이 공식을 적용하려면 빼기 기능이 필요하다. 앞에서 더하기 필터(|add:5)를 사용한 것처럼 빼기 필터(|sub:3)가 있으면 좋을 것 같다. 하지만 장고에는 빼기 필터가 없다. 그래서 우리는 빼기 필터를 만들 것이다.
|add:-3와 같이 숫자를 직접 입력하면 더하기 필터를 이용하여 원하는 값을 뺀 결과를 화면에 보여줄 수는 있다. 하지만 이 방법은 이곳에는 사용할 수 없다. 왜냐하면 더하기 필터에는 변수를 적용할 수 없기 때문이다.
> add 필터는 인수로 숫자만 가능하다.

2. 템플릿 필터 디렉터리 만들기
템플릿 필터 함수는 템플릿 필터 파일을 작성한 다음에 정의해야 한다. 템플릿 필터 파일은 템플릿 파일이나 스태틱 파일과 마찬가지로 pybo\templatetags 디렉터리를 새로 만들어서 저장해야 한다.
```
C:\projects\mysite\pybo\templatetags
```
pybo\templatetags 디렉터리는 반드시 앱 디렉터리 아래에 생성해야 한다. mystie 디렉터리 아래에 만들면 안 되므로 위치에 주의하자.
```
(mysite) D:\code\vscode\github\jump_to_django\src\projects\mysite\pybo>mkdir templatetags
```

3. 템플릿 필터 작성하기
pybo_filter.py 파일을 만든 후 템플릿 필터 함수를 다음과 같이 작성하자.
```
from django import template

register = template.Library()


@register.filter
def sub(value, arg):
    return value - arg
```
템플릿 필터 함수를 만드는 방법은 무척 간단하다. sub 함수에 @register.filter라는 에너테이션(데코레이터)을 적용하면 템플릿에서 해당 함수를 필터로 사용할 수 있게 된다. 템플릿 필터 함수 sub는 기존 값 value에서 입력으로 받은 값 arg를 빼서 반환할 것이다.

4. 템플릿 필터 사용하기
템플릿 필터 함수를 템플릿에 사용해 보자. 여러분이 직접 만든 템플릿 필터를 사용하려면 템플릿 파일 맨 위에 {% load pybo_filter %}와 같이 템플릿 필터 파일을 로드해야 한다. 만약 템플릿 파일 맨 위에 extends 문이 있으면 load 문은 extends 문 다음에 위치해야 한다.
```
{% load pybo_filter %}
```
다음은 템플릿 필터 로드 후 일련번호를 적용한 것이다.
```
{% extends 'base.html' %}
{% load pybo_filter %}
{% block content %}
<div class="container my-3">
    <table class="table">
        <thead>
        <tr class="thead-dark">
            <th>번호</th>
            <th>제목</th>
            <th>작성일시</th>
        </tr>
        </thead>
        <tbody>
        {% if question_list %}
        {% for question in question_list %}
        <tr>
            <td>
                <!-- 번호 = 전체건수 - 시작인덱스 - 현재인덱스 + 1 -->
                {{ question_list.paginator.count|sub:question_list.start_index|sub:forloop.counter0|add:1 }}
            </td>
            <td>
                <a href="{% url 'pybo:detail' question.id %}">{{ question.subject }}</a>
            </td>
            <td>{{ question.create_date }}</td>
        </tr>
(... 생략 ...)
```
<td>{{ forloop.counter }}</td>에서 {{ forloop.counter }}를 변경했다. 다음은 게시물 일련번호 공식에 사용된 코드를 정리한 표이다.
> 일련번호 공식은 전체 개시물 개수 - 시작 인덱스 - 현재 인덱스 + 1이다.

| 공식 요소 | 설명 |
| :--- | :--- |
| question_list.paginator.count | 전체 게시물 개수 |
| question_list.start_index | 시작 인덱스(1부터 시작) |
| forloop.counter0 | 루프 내의 현재 인덱스(foorloop.counter0는 0부터, forloop.counter는 1부터 시작) |

코드 수정 후 일련번호를 확인해 보면 항상 1부터 시작했던 문제가 사라졌음을 확인할 수 있다.

![3-03_3.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_3_Part/3-03_3.png)

![3-03_4.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_3_Part/3-03_4.png)