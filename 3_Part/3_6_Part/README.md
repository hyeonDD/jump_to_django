<!-- -
![](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_6_Part/django.png)
-->
# 회원가입 구현하기
이번에는 파이보에 회원가입 기능을 구현해 보자. 회원가입 기능을 만들어 보았다면 웹 프로그래밍은 거의 마스터했다고 할 수 있다. 그만큼 회원가입 기능은 웹 사이트에서 핵심 중의 핵심이라 할 수 있따. 회원가입 역시 로그인,로그아웃과 마찬가지로 장고의 django.contrib.auth앱을 이용하여 쉽게 구현할 것이다.
> django.contrib.auth 앱을 이용하여 쉽게 구현한다는 뜻은 models.py 파일이나 urls.py, views.py 파일을 많이 수정하지 않고도 구현할 수 있다는 뜻이다.

## 회원가입 구현하기
1. 회원가입 링크 추가하기
회원가입 링크는 login.html 파일에 추가하자.
```
(... 생략 ...)
<div class="container my-3">
    <div class="row">
        <div class="col-4">
            <h4>로그인</h4>
        </div>
        <div class="col-8 text-right">
            <span>또는 <a href="{% url 'common:signup' %}">계정을 만드세요.</a></span>
        </div>
    </div>
    <form method="post" class="post-form" action="{% url 'common:login' %}">
(... 생략 ...)
```
form 엘리먼트 위에 <div class="row">...</div>를 입력하여 회원가입 링크를 추가했다.

2. 회원가입 URL 매핑 추가하기
1단계에서 login.html 파일에 {% url 'common:signup' %}를 추가했으므로 여기에 대응하는 URL 매핑을 추가해야 한다. common\urls.py 파일에 회원가입을 위한 URL 매핑을 추가하자.
```
from django.urls import path
from django.contrib.auth import views as auth_views
from . import views

app_name = 'common'

urlpatterns = [
    path('login/', auth_views.LoginView.as_view(template_name='common/login.html'), name='login'),
    path('logout/', auth_views.LogoutView.as_view(), name='logout'),
    path('signup/', views.signup, name='signup'),
]
```
이제 로그인 화면에서 회원가입 링크를 누르면 views.singup 함수가 실행될 것이다.

3. 회원가입에 사용할 폼 만들기
회원가입에 사용할 UserForm을 commons\forms.py 파일에 작성하자.
```
from django import forms
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth.models import User


class UserForm(UserCreationForm):
    email = forms.EmailField(label="이메일")

    class Meta:
        model = User
        fields = ("username", "password1", "password2", "email")
```
UserForm은 django.contrib.auth.forms 패키지의 UserCreationForm 클래스를 상속하고 email 속성을 추가했다.
상속한 UserCreationForm은 다음 속성을 가지고 있다.

| 속성명 | 설명 |
| :--- | :--- |
| username | 사용자 이름 |
| password1 | 비밀번호1 |
| password2 | 비밀번호2(비밀번호1을 제대로 입력했는지 대조하기 위한 값) |

즉 UsercreationForm이 기본적으로 가지고 있는 username, password1, password2 속성에 부가 정보인 email 속성을 추가하기 위해 UserCreationForm을 상속한 UserForm을 만든 것이다.
UserCreationForm의 is_valid 함수는 회원가입 화면의 필드값 3개가 모두 입력되었는지, 비밀번호1과 비밀번호2가 같은지, 비밀번호의 값이 비밀번호 생성 규칙에 맞는지 등을 검사한다. is_valid 함수는 다음 단계에서 사용할 것이다.

4. 회원가입을 위한 signup 함수 정의하기
common\views.py 파일에 signup 함수를 다음과 같이 정의하자.
> common\views.py 파일에는 기본값 외에는 아무 것도 없는 상태일 것이다.
```
from django.contrib.auth import authenticate, login
from django.shortcuts import render, redirect
from common.forms import UserForm


def signup(request):
    """
    계정생성
    """
    if request.method == "POST":
        form = UserForm(request.POST)
        if form.is_valid():
            form.save()
            username = form.cleaned_data.get('username')
            raw_password = form.cleaned_data.get('password1')
            user = authenticate(username=username, password=raw_password)  # 사용자 인증
            login(request, user)  # 로그인
            return redirect('index')
    else:
        form = UserForm()
    return render(request, 'common/signup.html', {'form': form})
```
signup 함수는 POST 요청인 경우 화면에서 입력한 데이터로 새로운 사용자를 생성하고, GET 요청인 경우 common\signup.html 화면을 반환한다. POST 요청에서 form.cleaned_data.get 함수는 회원가입 화면에서 입력한 값을 얻기 위해 사용하는 함수이다. 여기서는 로그인 시 필요한 아이디, 비밀번호를 얻기 위해 사용되었다. 그리고 회원가입이 완료된 이후에 자동으로 로그인되도록 authenticate 함수와 login 함수를 사용했다.
> authenticate, login 함수는 django.contirb.auth 패키지에 있는 함수로 사용자인증과 로그인을 담당한다.

5. 회원가입 템플릿 만들기
common\signup.html 파일을 생성한 다음 아래와 같이 작성하자.
```
{% extends "base.html" %}
{% block content %}
<div class="container my-3">
    <div class="row my-3">
        <div class="col-4">
            <h4>계정생성</h4>
        </div>
        <div class="col-8 text-right">
            <span>또는 <a href="{% url 'common:login' %}">로그인 하세요.</a></span>
        </div>
    </div>
    <form method="post" class="post-form" action="{% url 'common:signup' %}">
        {% csrf_token %}
        {% include "form_errors.html" %}
        <div class="form-group">
            <label for="username">사용자 이름</label>
            <input type="text" class="form-control" name="username" id="username"
                   value="{{ form.username.value|default_if_none:'' }}">
        </div>
        <div class="form-group">
            <label for="password1">비밀번호</label>
            <input type="password" class="form-control" name="password1" id="password1"
                   value="{{ form.password1.value|default_if_none:'' }}">
        </div>
        <div class="form-group">
            <label for="password2">비밀번호 확인</label>
            <input type="password" class="form-control" name="password2" id="password2"
                   value="{{ form.password2.value|default_if_none:'' }}">
        </div>
        <div class="form-group">
            <label for="email">이메일</label>
            <input type="text" class="form-control" name="email" id="email"
                   value="{{ form.email.value|default_if_none:'' }}">
        </div>
        <button type="submit" class="btn btn-primary">생성하기</button>
    </form>
</div>
{% endblock %}
```
맨 위에 로그인 페이지로 이동할 수 있는 링크를 추가하고, 오류 표시를 위해 form_erros.html 파일을 include로 포함했다. 그리고 회원가입 시 필요한 UserForm 클래스의 username, password1, password2, email 필드를 추가했다. 코드 수정 후 회원가입을 시도해 보자. 로그인 페이지에서 '계정을 만드세요.' 링크를 눌러 회원가입 페이지로 이동하면 된다.

![3-06_1.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_6_Part/3-06_1.png)

![3-06_2.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_6_Part/3-06_2.png)

만약 비밀번호와 비밀번호 확인에서 다르게 입력하면 다음 오류 메시지를 볼 수 있을 것이다.

![3-06_3.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_6_Part/3-06_3.png)

필수값, 비밀번호 생성 규칙, 이메일 규칙도 모두 잘 작동할 것이다. 여러분이 직접 확인해보자.

6. 회원가입 후 장고 admin 접속해 계정 확인하기
회원가입을 완료한 다음 장고 admin에 접속하면 슈퍼 유저가 아닌 계정으로 로그인되어 있으므로 다음과 같은 경고 메시지가 나타날 것이다.

![3-06_4.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_6_Part/3-06_4.png)

장고 admin은 슈퍼 유저로 접속해야 한다. 슈퍼 유저로 로그인하자. 이어허 [홈-> 사용자(들)] 화면을 보면 새로 회원가입한 계정을 확인할 수 있을 것이다.

![3-06_5.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_6_Part/3-06_5.png)

축하한다. 파이보는 이제 회원가입을 할 수 있게 되었다.