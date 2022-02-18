<!-- -
![](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_5_Part/django.png)
-->
# 로그인,로그아웃 구현하기

이번에는 파이보에 로그인,로그아웃을 구현해 보자. 장고는 로그인,로그아웃을 쉽게 구현할 수 있도록 django.contrib.auth 앱을 제공한다. 이 앱은 장고 프로젝트 생성 시 mysite\config\settings.py에 자동으로 추가된다.
```
INSTALLED_APPS = [
    (... 생략 ...)
    'django.contrib.auth',
    (... 생략 ...)
]
```

## common 앱 생성 후 초기 설정 작업하기
당장 로그인,로그아웃을 구현하고 싶겠지만, 잠시 생각해 볼 문제가 있다. 로그인,로그아웃은 pybo 앱에 구현해야 옳을까? 필자는 그렇지 않다고 이야기하고 싶다. 왜냐하면 하나의 웹사이트에는 파이보와 같은 게시판 서비스 외에도 블로그나 쇼핑몰과 같은 굵직한 단위의 앱들이 함께 있을 수 있기 때문에 공통으로 사용되는 기능인 로그인이나 로그아웃을 이 중의 하나의 앱에 종송시키는 것은 좋지 않기 때문이다. 이러한 이유로 여기서는 로그인,로그아웃을 '공통 기능을 가진 앱'이라는 의미의 common 앱에 구현할 것이다.

1. common 앱 생성하기
common 앱을 생성하려면, 새로운 앱을 생성하는 것이므로 맨 처음 pybo 앱을 생성했던 것과 동일한 방법으로 생성하면 된다. projects\mysite 디렉터리 위치에서 다음 명령으로 common 앱을 생성하자.
```
(mysite) c:\projects\mysite>django-admin startapp common
```
common 앱을 생성하고 디렉터리를 열어 보면 pybo 앱의 디렉터리와 같은 구조임을 알 수 있다.

![3-05_1.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_5_Part/3-05_1.png)

2. 설정 파일에 common 앱 등록하기
config\settings.py 파일에 common 앱을 등록하자.
```

INSTALLED_APPS = [
    'common.apps.CommonConfig',  
    (... 생략 ...)   
]
```
INSTALLED_APPS에 'common.apps.CommonConfig',을 추가해 등록했다. 이어서 common 앱의 urls.py 파일을 사용하기 위해 config\urls.py 파일을 다음과 같이 수정하자.
```
from django.contrib import admin
from django.urls import path, include


urlpatterns = [
    (... 생략 ...)
    path('common/', include('common.urls')),
]
```
이 수정으로 \common\으로 시작하는 URL은 모두 common\urls.py 파일을 참조할 것이다.

3. common\urls.py 생성하기
common\urls.py 파일을 생성하고 다음과 같이 작성하자.
```
app_name = 'common'

urlpatterns = [
]
```
아직 common 앱에 어떤 기능도 구현하지 않았으므로 urlpatterns는 빈 상태로 놔두자.

## 로그인 구현하기
이제 common 앱에 로그인 기능을 구현하자.

1. 내비게이션바 수정하고 URL 매핑 추가하기
templates\navbar.html 파일을 열어 '로그인' 링크를 수정하자.
```
(... 생략 ...)
<ul class="navbar-nav">
    <li class="nav-item ">
        <a class="nav-link" href="{% url 'common:login' %}">로그인</a>
    </li>
</ul>
(... 생략 ...)
```
템플릿 태그로 {% url 'common:login' %}를 사용했으므로 common\urls.py 파일에 URL 매핑을 추가하자.
```
from django.urls import path
from django.contrib.auth import views as auth_views

app_name = 'common'

urlpatterns = [
    path('login/', auth_views.LoginView.as_view(), name='login'),
]
```
앞에서 언급했듯 로그인 기능은 django.contrib.auth 앱을 사용할 것이므로 common\views.py 파일은 수정할 필요가 없다. 여기서는 django.contrib.auth 앱의 LoginView 클래스를 사용할 것이다.

2. 로그인 템플릿 만들기
\pybo\ 페이지에서 내비게이션바의 '로그인' 링크를 눌러 보자. 그러면 아마도 다음과 같은 오류메시지가 나타날 것이다.

![3-05_2.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_5_Part/3-05_2.png)

위 오류는 registration 디렉터리에 login.html 파일이 없음을 의미한다. 1단계에서 사용한 LoginView는 registration이라는 템플릿 디렉터리에서 login.html 파일을 찾는다. 그런데 이 파일을 찾지 못해 오류가 발생한 것이다. 이 오류를 해결하려면 registration\login.html 템플릿 파일을 작성해야만 한다.
다만! 로그인은 common 앱에 구현할 것이므로 오류 메시지에 표시한 것처럼 registration 디렉터리에 템플릿 파일을 생성하기보다는 common 디렉터리에 템플릿을 생성하는 것이 좋다. 이를 위해 LoginView가 common 디렉터리의 템플릿을 참조할 수 있도록 common\urls.py 파일을 다음과 같이 수정하자.
```
(... 생략 ...)
urlpatterns = [
    path('login/', auth_views.LoginView.as_view(
        template_name='common/login.html'), name='login'),
]
```
as_view 함수에 template_name으로 'common\login.html'을 설정하면 registration 디렉터리가 아닌 common 디렉터리에서 login.html 파일을 참조하게 된다. 코드 수정 후 로그인 링크를 누르면 여전히 오류 메시지가 나타난다. 하지만 조금만 더 자세히 보면 메시지가 변경되었음을 알 수 있다.

![3-05_3.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_5_Part/3-05_3.png)

우리가 설정한 common 디렉터리에서 login.html 파일을 찾으려고 했으나 common 디렉터리에 login.html 파일이 없어서 오류가 발생하였다.

3. common 디렉터리 생성 후 login.html 생성하기
common 디렉터리를 생성하고 login.html 파일을 생성한 후 다음처럼 코드를 작성하자.
```
(mysite) D:\code\vscode\github\jump_to_django\src\projects\mysite>cd templates
(mysite) D:\code\vscode\github\jump_to_django\src\projects\mysite\templates>mkdir common
```
그리고 common 템플릿 디렉터리에 login.html 템플릿을 다음과 같이 작성하자.
```
{% extends "base.html" %}
{% block content %}
<div class="container my-3">
    <form method="post" class="post-form" action="{% url 'common:login' %}">
        {% csrf_token %}
        {% include "form_errors.html" %}
        <div class="form-group">
            <label for="username">사용자ID</label>
            <input type="text" class="form-control" name="username" id="username"
                   value="{{ form.username.value|default_if_none:'' }}">
        </div>
        <div class="form-group">
            <label for="password">비밀번호</label>
            <input type="password" class="form-control" name="password" id="password"
                   value="{{ form.password.value|default_if_none:'' }}">
        </div>
        <button type="submit" class="btn btn-primary">로그인</button>
    </form>
</div>
{% endblock %}
```
아이디와 비밀번호를 입력받아 로그인하는 간단한 HTML 코드이다. 입력 항목 username과 password는 모두 django.contrib.auth 앱에서 요구하는 필수 항목이다.

4. form_erros.html 만들어 작성하기
3단계에서 작성한 {% includ "form_errors.html" %}를 위해 templates\form_errors.html 파일을 생성한 후에 다음과 같이 코드를 작성하자.
```
{% if form.errors %}
    {% for field in form %}
        {% for error in field.errors %}  <!-- 필드 오류를 출력한다. -->
            <div class="alert alert-danger">
                <strong>{{ field.label }}</strong>
                {{ error }}
            </div>
        {% endfor %}
    {% endfor %}
    {% for error in form.non_field_errors %}   <!-- 넌필드 오류를 출력한다. -->
        <div class="alert alert-danger">
            <strong>{{ error }}</strong>
        </div>
    {% endfor %}
{% endif %}
```
form_erros.html 팡리은 로그인 실패 시 로그인이 실패한 원인을 알려 준다. 위에서 보듯 폼오류에는 다음과 같은 두 가지 오류가 있다.

---

폼 오류 2가지
- 필드 오류(입력값이 누락되었거나 형식에 맞지 않음)
- 넌필드 오류(입력값과 관계없이 발생한 오류)

---

5. 엉뚱한 값으로 로그인해 보기
코드를 수정한 다음 내비게이션바의 로그인 링크를 눌러 로그인 화면으로 이동해보자. 그러면 다음 화면을 볼 수 있다.

![3-05_4.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_5_Part/3-05_4.png)

form_erros.html 파일이 잘 작동하는지 시험해 보기 위해 입력값을 누락하거나 엉뚱한 값을 입력한 다음 <로그인>을 눌러 보자. 그러면 적절한 오류 메시지가 표시될 것이다.

![3-05_5.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_5_Part/3-05_5.png)

6. 제대로 로그인해 보기
현재 로그인할 수 있는 계정은 슈퍼 유저로 생성한 admin뿐이므로 admin 계정으로 로그인해 보자. 그러면 아마도 다음 오류 메시지를 볼 수 있을 것이다.
> 회원가입은 03-6에서 자세히 알아볼 것이다.

![3-05_6.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_5_Part/3-05_6.png)

이 오류는 왜 발생했을가? 로그인은 성공한 것이 맞다. 오류가 발생한 이유는 django.contrib.auth 앱이 로그인 성공 후 /accounts/profile/ 이라는 URL로 페이지를 리다이렉트했기 때문인데, 우리는 아직 이 URL에 대한 준비를 하지 않았다.

7. 로그인 성공 시 이동할 페이지 등록하기
다만 /account/profile/ URL은 현재 우리가 파이보에 구성한 것과 맞지 않으므로 로그인 성공 시 / 페이지로 이동할 수 있도록 config\setttings.py 파일을 수정하자. 마지막 줄에 LOGIN_REDIRECT_URL을 추가하면 된다.
> /페이지는 기본 URL인 localhost:8000/를 의미하며 보통 index 페이지라 부른다.
```
(... 생략 ...)

# 로그인 성공후 이동하는 URL
LOGIN_REDIRECT_URL = '/'
```
> 계속해서 오류가 발생하고 있지만 한 걸음씩 나아가고 있으니 두려워 말자.

![3-05_7.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_5_Part/3-05_7.png)

이 오류 메시지는 / 페이지에 대한 매핑이 없음을 의미한다.

8. config/urls.py 수정하기
/ 페이지에 대응하는 urlpatterns를 추가하자.
```
from django.contrib import admin
from django.urls import path, include
from pybo import views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('pybo/', include('pybo.urls')),
    path('common/', include('common.urls')),
    path('', views.index, name='index'),  # '/' 에 해당되는 path
]
```
그러면 / 페이지 요청에 대해 path('', views.index, name='index')가 작동하여 pybo/views.py 파일의 index 함수가 실행된다. 여기까지 수정한 다음 로그인이 잘 되는지 직접 확인해 보자.

## 로그아웃 구현하기

로그인에 성공했지만, 내비게이션바에는 여전히 로그인 링크가 보인다. 로그인 후에는 로그인 링크가 로그아웃 링크로 바뀌도록 navbar.html 파일을 수정하자.

1. 내비게이션바 수정하기
navbar.html 템플릿 파일에서 로그인 링크 부분을 다음과 같이 수정하자.
```
(... 생략 ...)
<li class="nav-item">
    {% if user.is_authenticated %}
    <a class="nav-link" href="{% url 'common:logout' %}">{{ user.username }} (로그아웃)</a>
    {% else %}
    <a class="nav-link" href="{% url 'common:login' %}">로그인</a>
    {% endif %}
</li>
(... 생략 ...)
```
{% if user.is_authenticated %}은 현재 로그인 상태를 판별하여 로그인 상태라면 로그아웃 링크를, 로그아웃 상태라면 로그인 링크를 보여준다. 로그인 상태에서는 로그인한 사용자명 {{ user.name }}도 표시했다.

2. 로그아웃 URL 매핑하기
로그아웃 링크가 추가되었으므로 {% url 'common:logout' %}에 대응하는 URL 매핑을 common\urls.py 파일에 추가하자.
```
from django.urls import path
from django.contrib.auth import views as auth_views

app_name = 'common'

urlpatterns = [
    path('login/', auth_views.LoginView.as_view(template_name='common/login.html'), name='login'),
    path('logout/', auth_views.LogoutView.as_view(), name='logout'),
]
```
이제 슈퍼 유저로 로그인하면 로그인 링크가 다음과 같이 변경된다.

![3-05_8.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_5_Part/3-05_8.png)

3. 로그아웃 성공 시 이동할 페이지 등록하기
로그인 성공 시 리다이렉트할 위치인 LOGIN_REDIRECT_URL을 등록했던 것과 마찬가지로 로그아웃 성공 시 리다이렉트할 위치도 config\settings.py파일에 추가하자.
```
(... 생략 ...)

# 로그인 성공후 이동하는 URL
LOGIN_REDIRECT_URL = '/'

# 로그아웃시 이동하는 URL
LOGOUT_REDIRECT_URL = '/'
```
로그아웃 성공 시 / 페이지로 이동하기 위한 LOGOUT_REDIRECT_URL = '/'을 설정했다. 코드 수정 후 로그인,로그아웃을 해보면 잘 작동할 것이다. 여러분이 직접 확인해 보자.