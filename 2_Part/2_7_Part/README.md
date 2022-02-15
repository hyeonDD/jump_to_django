<!-- -
![](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_7_Part/django.png)
-->
# 화면 예쁘게 꾸미기

지금까지 질문과 답변을 등록하고 조회하는 기능을 만들었다. 그런데 그럴싸한 화면이 아니라서 아쉽다. 여기서는 스타일시트를 이용해 웹 페이지에 디자인을 적용하는 방법을 알아본다.

## 웹 페이지에 스타일시트 적용하기
웹 페이지에 디자인을 적용하려면 스타일 시트(CSS)를 사용해야 하며, 스타일시트를 파이보에 적용하려면 CSS 파일이 스태틱 디렉터리에 있어야 한다.
> 이떄 CSS 파일은 장고에서 정적파일로 분류한다. 정적 파일은 주로 이미지(.png, .jpg)나 자바스크립트(.js), 스타일시트(.css)같은 파일을 의미한다.

1. 설정 파일에 스태틱 디렉터리 위치 추가하기
config/settings.py 파일을 열어 STATICFILES_DIRS에 스태틱 디렉터리 경로를 추가하자.
BASE_DIR / 'static'은 C:\projects\mysite\static을 의미한다.
```
STATIC_URL = '/static/'
STATICFILES_DIRS = [
    BASE_DIR / 'static',
]
```

2. 스태틱 디렉터리 만들고 스타일시트 작성하기
프로젝트 루트 디렉터리에 static이라는 이름의 디렉터를 생성하자. 루트 디렉터리는 C:\projects\mystie를 의미한다.
```
(mysite) c:\projects\mysite>mkdir static
```

---

**스태틱 디렉터리를 관리하는 방법**
pybo 앱 디렉터리 바로 아래에 static이라는 디렉터리를 만들어도 장고에서 스태틱 디렉터리로 인식된다. 즉, 다음과 같이 static 디렉터리를 만들어도 된다.
```
C:\projects\mysite\pybo\static
```
하지만 이 방법 역시 템플릿 디렉터리를 구성할 때 설명했듯 프로젝트 관리를 불편하게 만든다. 이 책에서는 스태틱 디렉터리도 템플릿 디렉터리처럼 한곳으로 모아서 관리할 것이다.

---

static 디렉터리를 만들었으면 그곳에 style.css 파일을 만들어 다음 코드를 작성하자. 여기서는 답변을 등록할 때 사용하는 textarea를 100%로 넓히고, <답변등록> 버튼 위에 margin을 10px 추가했다.
> CSS를 활용하면 이보다 더 예쁘게 만들 수 있다. 그것은 필자보다 디자인 감각이 뛰어난 독자 여러분 몫으로 남겨 둔다. 그 대신 다음 절에서 부트스랩을 이용해 좀 더 예쁘게 만드는 방법을 소개한다.
```
textarea {
    width:100%;
}

input[type=submit] {
    margin-top:10px;
}
```

3. 질문 상세 템플릿에 스타일 적용하기
pybo/question_detail.html 파일에 style.css 파일을 적용해 보자. 스태틱 파일을 사용하기 위해 템플릿 파일 맨 위에 {% load statidc %}태그를 삽입하고, link 엘리먼트 href 속성에 {% static 'style.css' %}를 적자.
```
{% load static %}
<link rel="stylesheet" type="text/css" href="{% static 'style.css' %}">
<h1>{{ question.subject }}</h1>
(... 생략 ...)
```
추가한 코드는 static 디렉터리의 style.css 파일을 연결한다는 의미다. 질문 상세 화면은 다음과 같이 바뀔 것이다. 만약 아무런 변화가 없다면 개발 서버를 종료했다가 다시 실행해 보자.

![2-07_br1.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_7_Part/2-07_br1.png)

축하한다. 이제 조금은 그럴싸한 화면을 출력할 수 있게 되었다.