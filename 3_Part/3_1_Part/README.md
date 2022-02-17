<!-- -
![](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_1_Part/django.png)
-->
# 내비게이션 기능 추가하기

지금까지 만든 파이보의 기능(질문 등록,조회, 답변 등록,조회)을 사용해 봤다면 편의 기능이 없어서 이런저런 불편함을 느꼇을 것이다. 그중에서 메인 페이지로 돌아갈 수 있는 장치가 없다는 것이 가장 불편하다. 여기서는 이런 불편함을 해소할 수 있는 기능을 추가하기 위해 내비게이션바를 만들어 볼것이다.
> 내비게이션바는 모든 화면이 위쪽에 고정되어 있는 부트스트랩 컴포넌트이다.
> 부트스트랩 내비게이션바 공식 문서: https://getbootstrap.com/docs/4.5/components/navbar

## 내비게이션바 추가하기

1. 로고, 로그인 링크 추가하기
내비게이션바는 모든 페이지에서 보여야 하므로 base.html 템플릿 파일을 열어 <body> 태그 바로 아래에 추가하자. 내비게이션바에는 메인 페이지로 이동해 주는 'Pybo' 로고(클래스값 navbar-brand)를 가장 왼쪽에 배치하고, 오른쪽에는 '로그인' 링크를 추가하자.

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
    <link rel="stylesheet" type="text/css" href="{% static 'pybo/style.css' %}">
    <title>Hello, pybo!</title>
</head>
<body>
<!-- 네비게이션바 -->
<nav class="navbar navbar-expand-lg navbar-light bg-light border-bottom">
    <a class="navbar-brand" href="{% url 'pybo:index' %}">Pybo</a>
    <button class="navbar-toggler ml-auto" type="button" data-toggle="collapse" data-target="#navbarNav"
        aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
        <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse flex-grow-0" id="navbarNav">
        <ul class="navbar-nav">
            <li class="nav-item ">
                <a class="nav-link" href="#">로그인</a>
            </li>
        </ul>
    </div>
</nav>
<!-- 기본 템플릿 안에 삽입될 내용 Start -->
{% block content %}
{% endblock %}
<!-- 기본 템플릿 안에 삽입될 내용 End -->
</body>
</html>
```

2. 질문 목록 화면에서 상단 내비게이션바 확인하기
1단계 작업을 마친 뒤 질문 목록 페이지를 요청하면 맨 위에 멋진 내비게이션바가 보인다. 또한 내비게이션바의 'Pybo'로고를 누르면 다른 페이지에서 메인 페이지로 돌아갈 수 있다. 'Pybo' 로고를 눌러서 잘 작동하는지 확인해 보자.

![3-01_1.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_1_Part/3-01_1.png)

내비게이션바는 모든 화면이 상속하는 base.html 파일에 추가되어서 질문 목록, 질문 상세, 질문 등록 화면 모두에 나타난다. 한번 확인해 보자.

3. 부트스랩이 제공하는 햄버거 메뉴 버튼 확인하기
그런데 부트스랩 내비게이션바에는 재미있는 기능이 하나 숨어 있다. 아무 페이지나 접속해서(여기서는 질문 목록에 접속했다) 웹 브라우저의 너비를 줄여 보자. 그러면 어느 순간 햄버거 메뉴 버튼이 생긴다. 그리고 '로그인' 링크는 사라진다.
> 햄버거 메뉴 버튼을 눌렀는데 아무 변화가 없더라도 당황하지 말자. 아직은 제대로 작동하지 않는 것이 정상이다.

![3-01_2.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_1_Part/3-01_2.png)

이렇게 부트스트랩은 화면 크기자 작은 기기를 고려한 '반응형 웹'까지 적용되어 있다. 그런데 햄버거 메뉴 버튼을 클릭해도 아무런 변화가 없을 것이다. 그 이유는 부트스랩 자바스크립트 파일(bootstrap.min.js)이 base.html 파일에 포함되지 않았기 때문이다. 또한 부트스랩 자바스크립트 파일은 제이쿼리를 기반으로 해서 만들어졌다. 결국 햄버거 메뉴 버튼을 제대로 사용하려면 부트스트랩 자바스크립트 파일과 제이쿼리 파일이 필요하다.

4. 부트스랩에 필요한 파일 추가하기 - 부트스랩 자바스크립트 파일
부트스트랩 자바스크립트 파일은 bootstrap-4.5.3.-dist.zip 압축 파일에 있다. 이 파일을 찾아 다음과 같은 위치에 복사해 붙여 넣자.

---

* 부트스트랩 자바스크립트 파일 위치: bootstrap-4.5.3-dist.zip/bootstrap-4.5.3-dist/js/bootstrap.min.js

* 붙여 넣을 위치: C:\projects\mysite\static\bootstrap.min.js

---

5. 부트스랩에 필요한 파일 추가하기 - 제이쿼리
제이쿼리는 https://jquery.com/download/에 접속하여 'Download the compressed, production jQuery 3.4.1' 링크를 마우스 오른쪽 버튼으로 누른 다음 '다른 이름으로 저장'을 하면 'jquery-3.4.1.min.js' 파일이 다운로드된다. 이 파일을 다음 위치에 붙여 넣자.

---

* 붙여 넣을 위치: C:\projects\mystie\static\jquery-3.4.1.min.js

---

![3-01_3.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_1_Part/3-01_3.png)

6. templates/base.html에 파일 추가하기
4, 5단계를 마치면 다음과 같은 경로에 각 파일이 위치해야 한다. 파일 위치를 확인하자.

![3-01_5.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_1_Part/3-01_5.png)

파일 위치를 확인하고 base.html 파일을 열어 base.html 파일 </body> 태그 바로 위에 코드를 추가하자. 수정한 다음 햄버거 메뉴 버튼을 누르면 숨어 있는 링크가 표시된다.
```
(... 생략 ...)
<!-- 기본 템플릿 안에 삽입될 내용 Start -->
{% block content %}
{% endblock %}
<!-- 기본 템플릿 안에 삽입될 내용 End -->
<!-- jQuery JS -->
<script src="{% static 'jquery-3.4.1.min.js' %}"></script>
<!-- Bootstrap JS -->
<script src="{% static 'bootstrap.min.js' %}"></script>
</body>
</html>
```

![3-01_6.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_1_Part/3-01_6.png)

> 혹시 실행이 제대로 되지 않으면 앞에서 다운로드한 제이쿼리 버전이 다르게 입력된 것은 아닌지 확인해 보자.

## incldue 기능으로 내비게이션바 추가해 보기
이번에는 조금 더 나은 방법으로 내비게이션바를 템플릿에 추가해 보자. 장고에는 템플릿 특정 위치에 템플릿 파일을 삽입하는 incldue라는 기능이 있다. 이번에는 incldue 기능으로 내비게이션바를 base.html 파일에 추가해 보자.

1. templates/navbar.html 생성하고 코드 작성하기
templates 디렉터리에 navbar.html 파일을 생성하고 다음과 같은 코드를 작성하자.
```
<!-- 네비게이션바 -->
<nav class="navbar navbar-expand-lg navbar-light bg-light border-bottom">
    <a class="navbar-brand" href="{% url 'pybo:index' %}">Pybo</a>
    <button class="navbar-toggler ml-auto" type="button" data-toggle="collapse" data-target="#navbarNav"
        aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
        <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse flex-grow-0" id="navbarNav">
        <ul class="navbar-nav">
            <li class="nav-item ">
                <a class="nav-link" href="#">로그인</a>
            </li>
        </ul>
    </div>
</nav>
```

navbar.html 파일의 코드는 base.html 파일에 작성했던 내비게이션바를 위한 HTML과 같다. 내비게이션바와 관련된 코드를 분리했다고 생각하면 된다

2. templates/base.html에 incldue 적용하기
이제 include 기능을 이용해 1단계에서 만든 navbar.html 파일을 base.html 파일에 삽입해보자.
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
<!-- 네비게이션바 -->
<nav class="navbar navbar-expand-lg navbar-light bg-light border-bottom">
    <a class="navbar-brand" href="{% url 'pybo:index' %}">Pybo</a>
    <button class="navbar-toggler ml-auto" type="button" data-toggle="collapse" data-target="#navbarNav"
        aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
        <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse flex-grow-0" id="navbarNav">
        <ul class="navbar-nav">
            <li class="nav-item ">
                <a class="nav-link" href="#">로그인</a>
            </li>
        </ul>
    </div>
</nav>
{% include "navbar.html" %}
<!-- 기본 템플릿 안에 삽입될 내용 Start -->
{% block content %}
{% endblock %}
<!-- 기본 템플릿 안에 삽입될 내용 End -->
<!-- jQuery JS -->
<script src="{% static 'jquery-3.4.1.min.js' %}"></script>
<!-- Bootstrap JS -->
<script src="{% static 'bootstrap.min.js' %}"></script>
</body>
</html>
```
이렇게 include 기능은 템플릿의 특정 영역을 중복, 반복해서 사용할 경우에 유용하다. 즉, 중복, 반복되는 템플릿의 특정 영역은 따로 템플릿 파일로 만들고, incldue 기능으로 그 템플릿을 포함한다. navbar.html 파일은 base.html 파일에서 1번만 사용되지만 따로 파일로 관리해야 이후 유지,보수하는 데 유리하므로 분리했다.