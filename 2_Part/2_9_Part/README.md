<!-- -
![](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_9_Part/django.png)
-->
# 표준 HTML과 템플릿 상속 사용해 보기

혹시 눈치챘는지 모르겠지만, 지금까지 작성한 질문 목록과 질문 상세 템플릿 파일은 표준 HTML 구조가 아니다. 어떤 운영체제나 웹 브라우저를 사용하더라도 웹 페이지가 동일하게 보이고 정상적으로 작동 하게 하려면 반드시 웹 표준을 지키는 HTML 문서를 작성해야 한다.

## 표준 HTML 구조는 어떻게 생겼을까?
표준 HTML 문서의 구조는 다음과 같이 html, head, body 엘리먼트가 있어야 하며, CSS 파일은 head 엘리먼트 안에 있어야 한다. 또한 head 엘리먼트 안에는 meta, title 엘리먼트 등이 포함되어야 한다.
```
{% load static %}
<!doctype html>
<html lang="ko">
<head>
    <!-- Required meta tags -->
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <!-- Bootstrap CSS -->
    <link rel="stylesheet" type="text/css" href="{% static 'bootstrap.min.css' %}">
    <!-- pybo CSS -->
    <link rel="stylesheet" type="text/css" href="{% static 'style.css' %}">
    <title>Hello, pybo!</title>
</head>
<body>

(... 생략 ...)

</body>
</html>
```

## 템플릿을 표준 HTML 구조로 바꾸기

앞에서 작성한 템플릿 파일을 표준 HTML 구조로 수정해 보자. 그런데 모든 템플릿 파일을 표준 HTML 구조로 변경하면 body 엘리먼트 바깥 부분은 모두 같은 내용으로 중복된다. 그리고 CSS 파일 이름이 변경되거나 새로운 CSS 파일이 추가되면 head 엘리먼트의 내용을 수정하려고 템플릿 파일을 해소하기 위한 템플리 상속(extends) 기능을 제공한다. 여기서는 단순히 템플릿을 표준 HTML 구조로 바꾸는 것이 아니라 템플릿 상속 기능까지 사용할 것이다. 그러면 파일을 하나씩 수정해 보자.

1. 템플릿 파일의 기본 틀 작성하기
우선 템플릿 파일의 기본 틀인 base.html 템플릿을 작성하자. 모든 템플릿에서 공통으로 입력할 내용을 여기에 포함한다고 생각하면 된다.
```
{% load static %}
<!doctype html>
<html lang="ko">
<head>
    <!-- Required meta tags -->
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <!-- Bootstrap CSS -->
    <link rel="stylesheet" type="text/css" href="{% static 'bootstrap.min.css' %}">
    <!-- pybo CSS -->
    <link rel="stylesheet" type="text/css" href="{% static 'style.css' %}">
    <title>Hello, pybo!</title>
</head>
<body>
<!-- 기본 템플릿 안에 삽입될 내용 Start -->
{% block content %}
{% endblock %}
<!-- 기본 템플릿 안에 삽입될 내용 End -->
</body>
</html>
```
body 엘리먼트에 {% block content %}와 {% endblock %} 템플릿 태그가 있다. 바로 이 부분이 이후 base.html 템플릿 파일을 상속한 파일에서 구현해야 하는 영역이 된다. 이제 question_list.html 템플릿을 다음과 같이 변경하자.

2. 질문 목록 템플릿 수정하기
질문 목록을 나타내는 question_list.html파일을 다음과 같이 수정하자.
```
# {% load static %} # 삭제
# <link rel="stylesheet" type="text/css" href="{% static 'bootstrap.min.css' %}">  # 삭제
{% extends 'base.html' %}
{% block content %}
<div class="container my-3">
    <table class="table">

        (... 생략 ...)

    </table>
</div>
{% endblock %}
```
base.html 템플릿 파일을 상속받고자 {% extends 'base.html' %} 템플릿 태그를 사용했다. 그리고 {% block content %}와 {% endblock %} 사이에 question_list.html 파일에서만 사용할 내용을 작성했다. 이제 question_list.html은 base.html을 상속받았으므로 표준 HTML 구조를 갖추게 되었따.

3. 질문 상세 템플릿 수정하기
질문 상세를 나타내는 question_detail.html 파일도 같은 방법으로 수정하자.
```
# {% load static %} #삭제
# <link rel="stylesheet" type="text/css" href="{% static 'bootstrap.min.css' %}"> #삭제
{% extends 'base.html' %}
{% block content %}
<div class="container my-3">
    <h2 class="border-bottom py-2">{{ question.subject }}</h2>

    (... 생략 ...)

    </form>
</div>
{% endblock %}
```
{% extends 'base.html' %}템플릿 태그를 맨 위에 추가하고 기존 내용 위 아래로 {% block content %}와 {% endblock %}를 작성했다.

4. 기존 스타일 파일 내용 비우기
부트스트랩을 사용하게 되었으니 style.css 파일의 내용을 비우자. 이 파일은 이후 부트스트랩으로 표현할 수 없는 스타일을 위해 사용할 것이므로 파일 자체를 삭제하지는 말고 내용만 삭제하자.
```
/* 내용을 전부 삭제하자. */
```
