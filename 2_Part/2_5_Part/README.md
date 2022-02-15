<!-- -
![](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_5_Part/django.png)
-->
# URL 더 똑똑하게 사용하기

이번에는 템플릿에서 사용한 URL 하드 코딩을 없애는 방법에 알아보자. 그나저나 URL 하드 코딩이란 무엇일까? 잠시 question_list.html 템플릿에 사용된 href값을 보자.
```
<li><a href="/pybo/{{ question.id }}/">{{ question.subject }}</a></li>
```
"/pybo{{ question.id }}"는 질문 상세를 위한 URL 규칙이다. 하지만 이런 URL 규칙은 프로그램을 수정하면서 '/pybo/question/2/' 또는 '/pybo/2/question/'으로 수정될 가능성도 있다. 이런 식으로 URL 규칙이 자주 변경된다면 템플릿에 사용된 모든 href값들을 일일이 찾아 수정해야 한다. URL 하드 코딩의 한계인 셈이다. 이런 문제를 해결하려면 해당 URL에 대한 실제 주소가 아닌 주소가 매핑된 URL 별칭을 사용해야 한다.

## URL 별칭으로 URL 하드 코딩 문제 해결하기
그러면 URL 별칭을 파이보에 적용해 보자.

1. pybo/urls.py 수정하여 URL 별칭 사용하기
템플릿의 href에 실제 주소가 아니라 URL 별칭을 사용하려면 우선 pybo/urls.py 파일을 수정해야 한다. path 함수에 있는 URL 매핑에 name속성을 부여하자.
```
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
    path('<int:question_id>/', views.detail, name='detail'),
]
```
이렇게 수정하면 실제 주소 /pybo/는 index라는 URL 별칭이, /pybo/2/는 detail이라는 URL 별칭이 생긴다.

2. pybo/question_list.html 템플릿에서 URL 별칭 사용하기
1단계에서 만든 별칭을 템플릿에서 사용하기 위해 /pybo/{{ question.id }}를 {% url 'detail' question.id %}로 변경하자.
```
(... 생략 ...)
    {% for question in question_list %}
        <li><a href="{% url 'detail' question.id %}">{{ question.subject }}</a></li>
    {% endfor %}
(... 생략 ...)
```

## URL 네임스페이스 알아보기
여기서 한 가지 더 생각할 문제가 있다. 현재의 프로젝트에서는 pybo 앱 하나만 사용하지만, 이후 pybo 앱 이외의 다른 앱이 프로젝트에 추가될 수도 있다. 이때 서로 다른 앱에서 같은 URL 별칭을 사용하면 중복 문제가 생긴다.

이 문제를 해결하며녀 pybo/urls.py 파일에 네임스페이스라는 개념을 도입해야 한다. 네임스페이스는 쉽게 말해 각각의 앱이 관리하는 독립된 이름 공간을 말한다.

1. pybo/urls.py에 네임스페이스 추가하기
pybo/urls.py 파일에 네임스페이스를 추가하려면 간단히 app_name 변수에 네임스페이스 이름을 저장하면 된다.
```
from django.urls import path

from . import views

app_name = 'pybo'

urlpatterns = [
    path('', views.index, name='index'),
    path('<int:question_id>/', views.detail, name='detail'),
]
```

네임스페이스 이름으로 'pybo'를 저장했다.

2. 네임스페이스 테스트하기 - 오류 발생!
/pybo/에 접속해 보자. 그러면 다음과 같은 오류가 발생한다.

![2-05_br1.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_5_Part/2-05_br1.png)

3. pybo/question_list.html 수정하기
오류가 발생한 이유는 템플릿에서 아직 네임스페이스를 사용하고 있지 않기 때문이다. {% url 'detail' question.id %}을 {% url 'pybo:detail' question.id %)} 으로 바꾸자.
```
(... 생략 ...)
    {% for question in question_list %}
        <li><a href="{% url 'pybo:detail' question.id %}">{{ question.subject }}</a></li>
    {% endfor %}
(... 생략 ...)
```
detail에 pybo라는 네임스페이스를 붙여준 것이다.

---

**URL 별칭은 여러 곳에서 사용된다**
URL 별칭은 템플릿 외에도 여러 곳에서 사용된다. 예를 들어 URL 별칭은 redirect 함수에도 사용된다. 다음은 redirect 함수에서 pybo:detail을 사용한 예이다. 지금은 눈으로 살펴보고 넘어가자.
```
redirect('pybo:detail', question_id=question.id)
```