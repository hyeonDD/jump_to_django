<!-- -
![](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_1_Part/django.png)
-->
# 주소와 화면을 연결하는 URL과 뷰

이제부터 파이보를 만들면서 장고의 기능을 살펴볼 것이다. 가장 먼저 요청 URL을 어떻게 처리하고 또 어떻게 파이썬 프로그램을 호출하는지 알아보자.

## 앱 생성하고 확인하기

1장에서 mysite 프로젝트를 생성했다. 프로젝트에는 장고가 제공하는 기본 앱과 개발자(여러분)가 직접 만든 앱이 포함될 수 있으며, 장고에서 말하는 '앱'은 안드로이드 앱과 성격이 다르다고 언급했다. 그러면 장고의 앱이란 정말로 무엇일까? 우리의 파이보 서비스에 필요한 pybo 앱을 만들어 보며 알아보자.

1. pybo 앱 생성하기
명령 프롬프트에서 django-admin의 startapp 명령을 이용하여 pybo 앱을 생성하자.
```
(mysite) C:\projects\mysite>django-admin startapp pybo
(mysite) C:\projects\mysite>
```

2. 생성된 앱 확인하기
1단계를 진행하면 아무런 메시지가 나타나지 않을 것이다. 하지만 디렉터리 목록을 살펴보면 pybo라는 이름의 디렉터리가 생성되었음을 확인할 수 있다.
__init__.py, admin.py, apps.py 같은 파일이 바로 pybo 앱을 위한 것이다. 이 파일들은 실습을 진행하며 작성 또는 수정할 것이다.

![pybo.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_1_Part/pybo.png)

## 안녕하세요 파이보?

지금부터 실습을 시작해 보자. 실습을 진행하면 파이보 서비스가 조금씩 완성될 것이다. 여기서 필자가 여러분에게 한 가지 부탁하고 싶은 것이 있다. **결과를 보기 위해 실습을 무작정 따라 하지 않길 바란다. 결과를 보고 싶은 조급한 마음은 이해하지만, 결과가 나온 이유를 명확하게 이해하면서 따라 해야 좋은 개발자가 될 수 있다.**

1. 개발 서버 구동하기
개발 서버를 구동하자.
```
(mysite) C:\projects\mysite> python manage.py runserver
```

2. localhost:8000/pybo에 접속하기
우리는 아직 아무런 작업도 하지 않았다. 하지만 그냥 웹 브라우저 주소창에 다음을 입력하여 접속해 보자. 앞으로 이 과정을 '페이지를 요청한다' 또는 localhost:8000을 생략하여 '/pybo/를 요청한다'라고 할 것이다.
```
localhost:8000/pybo
```

3. 오류 메시지 확인하기
그러면 'Page not found (404)'오류 페이지가 보인다. 명령 프롬프트에도 같은 오류가 보인다. 404는 HTTP 오류 코드 중 하나로, 사용자가 요청한 페이지를 찾을 수 없는 경우 발생하는 오류이다. 장고는 오류 발생 시 오류 원인을 웹브라우저 또는 멸령 프롬프트에 자세히 보여 준다.

![2-01_2.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_1_Part/2-01_2.png)

![err_404png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_1_Part/err_404png)

이 오류는 왜 발생했을까? 장고는 사용자가 웹 브라우저에서 /pybo/라는 페이지를 요청하면 해당 페이지를 가져오는 URL 매핑이 있는지 cofig/urls.py 파일을 뒤져 찾아본다. 그런데 아직 /pybo/ 페이지에 해당하는 URL 매핑을 장고에 등록하지 않았다. 그래서 장고는 /pybo/ 페이지를 찾을 수 없다고 메시지를 보낸 것이다.

4. config/urls.py 수정하기
그럼 이제 할 일은 정해졌다. 장고가 사용자의 페이지 요청을 이해할 수 있도록 config/urls.py 파일을 수정하자. 앞으로 이를 'URL 매핑을 추가한다'고 말할 것이다. URL 매핑을 추가하기 위해 config/urls.py 파일을 수정하자.
> config/urls.py은 페이지 요청 시 가장 먼저 호출되며, 요청 URL과 뷰 함수를 1:1로 연결해 준다.
> 뷰 함수는 아직 살펴보지 않았다. 뷰 함수는 화면을 보여 주기 위한 함수로, view.py에 있는 함수다. 이 함수도 곧 공부할 것이다.
```
from django.urls import path
from pybo import views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('pybo/', views.index),
]
```
코드의 urlpatterns 변수를 보면 path 함수를 사용하여 pybo/ URL과 views.index를 매피앻ㅆ다. views.index는 views.py 파일의 index 함수를 의미한다. 장고는 이런 식으로 URL과 뷰함수를 매핑했다.

5. config/urls.py 다시 살펴보기
그런데 여러분이 urlpatterns에 입력하 ㄴURL은 웹 브라우저에 입력한 localhost:8000/pybo에서 호스트명 localhost와 포트 번호:8000이 생략된 pybo/이다. 호스트명과 포트는 장고가 실행되는 환경에 따라 변하는 값이며 장고가 이미 알고 있는 값이다. 그러므로 urlpatterns에는 호스트명과 포트를 입력하지 않는다.
```
urlpatterns = [
    path('admin/', admin.site.urls),
    path('pybo/', views.index),
]
```
그리고 pybo에 슬래시 /를 붙여 입력한 점에도 주목하자! 슬래시를 붙이면 사용자가 슬래시없이 주소를 입력해도 장고가 자동으로 슬래시를 붙여 준다. 이는 URL을 정규화하는 장고의 기능 덕분이다. 아무튼! 특별한 경우가 아니라면 URL 매핑에는 호스트명과 포트를 생략하고 끝에는 슬래시를 붙이자.
> 웹 브라우저 주소창에 localhost:8000/pybo라고 입력하면 장고가 자동으로 localhost:8000/pybo/와 같이 슬래시를 붙여 준다.

---

**프로젝트 디렉터리는 BASE_DIR 변수에 저장되어 있다.**

여러분의 파이보 프로젝트 디렉터리는 C:\projects\mysite일 것이다. 장고는 이 경로를 settings.py 파일의 **BASE_DIR** 변수에 저장한다. 이 책에서는 파일 경로를 언급할 때 BASE__DIR을 생략한다. 예를 들어 config/urls.py라 언급하면 BASE_DIR의 값인 C:\projects\mysite가 생략된 것이므로 실제 언급하는 파일 위치는 C:\projects\mysite\config\urls.py이다.

---

6. 오류 메시지 확인하기
다시 /pybo/에 접속해 보자. 그러면 웹 브라우저에 '사이트에 연결할 수 없음' 오류가 표시되고, 개발 서버에는 다음과 같은 오류가 표시된다.
```
# (... 생략 ...)
  File "D:\code\vscode\github\jump_to_django\src\projects\mysite\config\urls.py", line 28, in <module>
    path('pybo/', views.index),
AttributeError: module 'pybo.views' has no attribute 'index'
```
config/urls.py 파일을 수정했음에도 이런 오류가 발생한 이유는 URL 매핑에 추가한 뷰 함수 views.index가 없기 때문이다.

7. pybo/views.py 작성하기
pybo/views.py 파일에 index 함수를 추가하자.
```
from django.http import HttpResponse


def index(request):
    return HttpResponse("안녕하세요 pybo에 오신것을 환영합니다.")
```
return 문에 사용된 HttpResponse는 페이지 요청에 대한 응답을 할 때 사용하는 장고 클래스이다. 여기서는 HTTPResponse에 "안녕하세요 pybo에 오신것을 환영합니다."라는 문자열을 전달하여 이 문자열이 웹 브라우저에 그대로 출력되도록 만들었다.
> index 함수의 매개변수 request는 장고에 의해 자동으로 전달되는 HTTP 요청 객체이다.
> request는 사용자가 전달한 데이터를 확인할 때 사용된다.

8. 첫 번째 프로그램 완성!
이제 /pybo/에 접속하면 웹 브라우저에 "안녕하세요 pybo에 오신것을 환영합니다."라는 문자열이 출력된다. 축하한다. 여러분의 첫 번째 장고 프로그램이 완성되었다!

![2-01_5.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_1_Part/2-01_5.png)

## 장고 개발 흐름 정리하기
지금까지 여러분이 경험한 개발 과정은 모든 실습 과정에서 여러 번 반복될 것이다. 이 과정은 중요하다! 다음 그림으로 개발 과정을 정리해 보자.

![2-01_6.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_1_Part/2-01_6.png)

1. 웹 브라우저 주소창에 localhost:8000/pybo 입력(장고 개발 서버에 /pybo/ 페이지 요청).
2. config/urls.py 파일에서 URL을 해석해 pybo/views.py 파일의 **index** 함수 호출.
3. pybo/views.py 파일의 **index** 함수를 실행해 실행 결과를 웹 브라우저에 전달.

사용자가 /pybo/ 페이지를 요청하면 장고 개발 서버가 URL을 분석해, URL에 매핑된 함수를 호출하고, 함수 실행 결과를 웹 브라우저 화면에 전달한다. 이 과정을 기억하며 실습을 진행하자.

## URL 분리하기
1. config/urls.py 다시 살펴보기
config/urls.py 파일을 다시 한번 살펴보자. 아까 얘기했듯이 pybo 앱 관련 파일은 대부분 pybo 디렉터리에 있다. 하지만 config/urls.py 팡리은 pybo 디렉터리에 없다. 그러므로 pybo 앱에 URL 매핑을 추가하려면 pybo 디렉터리가 아닌 config 디렉터리에 있는 urls.py 파일을 수정해야 한다.
```
from django.urls import path
from pybo import views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('pybo/', views.index),
]
```
이 방식은 프로젝트의 짜임새를 전혀 고려하지 않은 것이다. pybo 앱 관련 urls.py 파일을 따로 구성할 방법은 없을까? 물론 있다.

2. config/urls.py 수정하기
include 함수를 임포트해 pybo/의 URL 매핑을 path('pybo/', view.index)에서 path('pybo/', incldue('pybo.urls'))로 수정하자.
```
from django.contrib import admin
from django.urls import path, include
from pybo import views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('pybo/', include('pybo.urls')),
]
```
**path('pybo/', include('pybo.urls'))는 pybo/로 시작되는 페이지 요청은  pybo/urls.py 파일에 있는 URL 매핑을 참고하여 처리하라는 의미이다.** 다시 말해 pybo 앱과 관련된 URL 요청은 앞으로 pybo/urls.py 파일에서 관리하라는 뜻이다. 다음 예와 같이 pybo/로 시작하는 요청은 config/urls.py 파일이 아닌 pybo/urls.py파일을 통해 처리하게 된다.

| URL 요청 | 요청 처리 파일 |
| :--- | :--- |
| pybo/question/create/ | pybo/urls.py |
| pybo/answer/create/ | pybo/urls.py |
| 그외 | config/urls.py |

3. pybo/urls.py 생성하기
pybo 앱 디렉터리에 urls.py 파일을 생성하자. 

![pybo_urls.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_1_Part/pybo_urls.png)

4. pybo/urls.py 수정하기
pybo/urls.py 파일을 다음과 같이 수정하자. 코드 내용은 기존 config/urls.py 파일과 다르지 않다. 다만 path('', views.index) 입력 부분만 다르다. config/urls.py 파일에서 pybo/에 대한 처리를 한 상태에서 pybo/urls.py 파일이 실행되므로 첫 번째 매개변수에 pybo/가 아닌 빈 문자열(")을 인자로 넘겨준 것이다.
```
# pybo/urls.py
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index),
]
```


다시 /pybo/에 접속해 보자. 다음 화면이 나오면 잘 접속된 것이다.

![2-01_9.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_1_Part/2-01_9.png)