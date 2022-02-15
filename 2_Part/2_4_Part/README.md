<!-- -
![](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_4_Part/django.png)
-->
# 질문 목록과 질문 상세 기능 구현하기

이제 파이보의 핵심 기능을 구현할 것이다. 바로 질문 목록과 질문 상세 기능이다. 우선은 /pybo/에 접속하면 질문을 모두 조회할 수 있는 기능을 구현해 볼것이다.
> 지금은 localhost:8000/pybo/에 접속하면 '안녕하세요 pybo에 오신것을 환영합니다.'가 출력되는 상태이다.

## 질문 목록 조회 구현하기

질문 목록 조회를 위해 pybo/views.py 파일을 열어 코드를 조금씩 수정해 보자.

1. Question 모델 데이터 작성일시 역순으로 조회하기
Question 모델을 임포트해 Question 모델 데이터를 작성한 날짜의 역순으로 조회하기 위해 order_by 함수를 사용했다. 조회한 Question 모델 데이터는 context 변수에 저장했다. context 변수는 조금 후에 설명할 render 함수가 템플릿을 HTML로 변환하는 과정에서 사용되는 데이터이다.
```
from django.http import HttpResponse
from .models import Question


def index(request):
    """
    pybo 목록 출력
    """
    question_list = Question.objects.order_by('-create_date')
    context = {'question_list': question_list}
    return HttpResponse("안녕하세요 pybo에 오신것을 환영합니다.")
```

order_by 함수는 조회한 데이터를 특정 속성으로 정렬하며, '-create_date'는 - 기호가 앞에 붙어 있으므로 작성일시의 역순을 의미한다.

2. render로 화면 출력하기
이제 조회한 Question 모델 데이터를 템플릿 파일을 사용하여 화면에 출력할 수 있는 render 함수를 사용해 보자.
```
# from django.http import HttpResponse  # 삭제
from django.shortcuts import render
from .models import Question


def index(request):
    """
    pybo 목록 출력
    """
    question_list = Question.objects.order_by('-create_date')
    context = {'question_list': question_list}
    return render(request, 'pybo/question_list.html', context)
```
render 함수는 context에 있는 Question 모델 데이터 questioN_list를 pybo/question_list.html 파일에 적용하여 HTML 코드로 변환한다. 그리고 장고에서는 이런 파일(pybo/question_list.html)을 템플릿이라 부른다. 템플릿은 장고의 태그를 추가로 사용할 수 있는 HTML파일이라 생각하면 된다. 템플릿에 대해서는 바로 다음 실습 과정을 통해 자연스럽게 알아보겠다.

3. 템플릿을 모아 저장할 디렉터리 만들기
템플릿을 만들기 전에 템플릿을 저장할 디렉터리를 루트 디렉터리 바로 밑에 만들자.
```
(mysite) c:\projects\mysite>mkdir templates
```

4. 템플릿 디렉터리 위치 config/setttings.py에 등록하기
3단계에서 만든 템플릿 디렉터리를 장고 config/settings.py 파일에 등록하자. config/settings.py 파일을 열어 **TEMPLATES**항목을 같이 수정하자.
```
(... 생략 ...)
TEMPLATES = [
    {
        (... 생략 ...)

        'DIRS': [BASE_DIR / 'templates'],

        (... 생략 ...)
    },
]

```

**DIRS**에는 템플릿 디렉터리를 여러 개 등록할 수 있다. 다만 현재 우리가 개발하는 파이보는 1개의 템플릿 디렉터리를 쓸 것이므로 BASE_DIR / 'templates'와 같이 1개의 디렉터리만 등록하자. 현재 BASE_DIR은 C;\projects\mysite 이므로 templates만 더 붙여 C:\projects\mysite\templates를 반환한다.

---

**장고는 앱 하위에 있는 templates 디렉터리를 자동으로 템플릿 디렉터리로 인식한다.**

장고는 DIRS에 설정한 디렉터리 외에도 특정 앱(예를 들어 pybo 앱) 디렉터리 하위에 있는 templates라는 이름의 디렉터리를 자동으로 템플릿 디렉터리로 인식한다. 예를 들어 다음과 같은 pybo 앱 디렉터리 밑의 templates 디렉터리는 별다른 설정을 하지 않아도 템플릿 디렉터리로 인식된다.
```
c:\projects\mysite\templates
```

하지만 필자는 이 방법을 권장하지 않는다. 왜냐하면 하나의 사이트에서 여러 앱을 사용할 때 여러 앱의 화면을 구성하는 템플릿은 한 디렉터리에 모아 관리하는 편이 여러모로 좋기 때문이다. 예를 들어 여러 앱이 공통으로 사용하는 공통 템플릿을 어디에 저장해야 할지 생각해 보면 왜 이런 방법을 선호하는지 쉽게 이해될것이다. 그래서 파이보는 템플릿 디렉터리를 mysite\pybo\templates와 같은 방식이 아니라 mysite\templates\pybo 같은 방식으로 관리하며, 공통으로 사용하는 템플릿은 C:\prjects\mysite\templates에 저장한다.


| 디렉터리 | 경로 |
| :--- | :--- |
| 공통 템플릿 디렉터리 | C:\projects\mysite\templates |
| pybo 앱 템플릿 디렉터리 | C:\projects\mysite\templates\pybo |

---

5. 템플릿 파일 만들기
3단계에서 만든 템플릿 디렉터리를 참고하여 question_list.html 템플릿 파일을 mysite\templates\pybo\ 디렉터리에 생성하자. pybo 디렉터리를 templates 안에 새로 만들어서 파일을 추가해야 한다.

![2-04_tree1.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_4_Part/2-04_tree1.png)

그리고 템플릿 파일을 다음과 같이 작성하자.
```
{% if question_list %}
    <ul>
    {% for question in question_list %}
        <li><a href="/pybo/{{ question.id }}/">{{ question.subject }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>질문이 없습니다.</p>
{% endif %}
```

**{% if question_list %}**라는 문장이 눈에 띌 것이다. 바로 이것이 템플릿 태그이다. 템플릿 태그는 **{% 와 %}**로 둘러싸인 문장을 말한다. 다음 표에 정리한 템플릿 태그의 의미를 살펴보면 파이썬 문법과 크게 다르지 않음을 알 수 있다. 템플릿 태그는 따로 문법을 설명하지 않고 그때그때 필요할때마다 설명하겠다.
> 템플릿 태그에서 사용된 question_list가 바로 2단계의 render 함수에서 템플릿으로 전달한 Question 모데 데이터이다.
> 만약 템플릿 태그의 자세한 내용이 궁금하다면 장고 공식 문서를 참고하자.
> 템플릿 장고 공식 문서 주소: https://docs.djangoproject.com/en/3.0/topics/templates

| 템플릿 태그 | 의미 |
| :--- | :--- |
| {% if question_list %} | question_list가 있다면 |
| {% for question in question_list %} | question_list를 반복하며 순차적으로 question에 대입 |
| {{ question.id }} | for 문에 의해 대입된 question 객체의 id 출력 |
| {{ question.subject }} | for 문에 의해 대입된 question 객체의 subject 출력 |

---

**템플릿 태그! 3가지 유형만 정리하면 끝!**
장고의 템플릿 태그는 분기, 반복, 객체 출력이라는 3가지 유형만 알면 된다. 분기 템플릿태그는 다음과 같다. 문법을 보면 알겠지만 파이썬의 if 문과 다르지 않다. 다만 if 문이 끝나는 부분에 **{% endif %}**를 사용하는 점만 다르다.
```
{% if 조건문1 %}
    <p>조건문1에 해당하는 경우</p>
{% if 조건문2 %}
    <p>조건문2에 해당하는 경우</p>
{% else %}
    <p>조건문1, 2에 해당하는 경우</p>
{% endfor %}
```
또한 반복 템플릿 안에서는 forloop 객체를 사용할 수도 있다. forloop 객체는 반복 중 유용한 값을 제공한다.

| forloop 객체 속성 | 설명 |
| :--- | :--- |
| forloop.counter | for 문의 순서로 1부터 표시 |
| forloop.counter0 | for 문의 순서로 0부터 표시 |
| forloop.first | for 문의 첫 번째 순서인 경우 True |
| forloop.last | for 문의 마지막 순서인 경우 True |

객체 출력 템플릿 태그는 다음과 같다. 객체에 속성이 있으면 파이썬과 동일한 방법으로 점(.) 연산자를 사용한다.
```
{{ question }}
{{ question.id }}
{{ question.subject }}
```

6. 질문 목록이 잘 출력되는지 확인해 보기
템플릿 디렉터리를 추가한 상태이므로 장고 개발 서버를 다시 시작해 /pybo/에 접속하자. 장고 개발 서버를 다시 시작하지 않으면 장고가 템플릿 디렉터리를 인식하지 못해 오류가 발생한다.

![2-04_br1.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_4_Part/2-04_br1.png)

화면을 보면 이전에 등록한 질문 2건이 보인다. 직접 장고 셸이나 장고 ADmin에서 다른 질문도 추가해 보면서 질문 목록이 잘 만들어지는지 테스트해 보기 바란다.

## 질문 상세 기능 구현하기

1. 질문 목록에서 아무 질문이나 눌러 보기
질문 목록 화면에서 아무 질문(예를 들면 Django Model Question)이나 눌러 보자. 그러면 다음과 같은 오류 화면이 나타난다.
> 질문을 눌렀을 때 /pybo2/와 같은 주소로 이동한 이유는 템플릿에서 href 엘리먼트에 link 속성을 <a href="/pybo/{{ question.id }}>으로 지정했기 때문이다.

![2-04_br2.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_4_Part/2-04_br2.png)

오류 화면이 나타난 이유는 /pybo/2/에 대한 URL 매핑을 추가하지 않았기 때문이다.

2. pybo/urls.py 열어 URL 매핑 추가하기
/pybo/2/의 숨은 의도는 'Question 모델 데이터 중 id값이 2인 데이터를 조회하라'이다. 이 의도에 맞게 결과 화면을 보여줄 수 있도록 pybo/urls.py 파일에서 path('<int:question_id>/', views.detail) URL 매핑을 추가하자.
> 장고 pybo/까지는 config/urls.py의 URL 매핑을, pybo/에 이은 pybo/urls.py의 URL 매핑을 참고할 것이다.
> int:는 question_id에 숫자가 매핑되었음을 의미한다.
```
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index),
    path('<int:question_id>/', views.detail),
]
```
/pybo/2/가 요청되면 이 매핑 규칙에 의해 /pybo/<int:question_id>/가 적용되어 question_id에 2라는 값이 저장되고 views.detail 함수가 실행된다.

3. pybo/views.py 열어 화면 추가하기
pybo/views.py 파일을 열어 detail 함수를 추가하자.
```
(... 생략 ...)

def detail(request, question_id):
    """
    pybo 내용 출력
    """
    question = Question.objects.get(id=question_id)
    context = {'question': question}
    return render(request, 'pybo/question_detail.html', context)
```

추가된 내용은 앞에서 만든 index 함수와 크게 다르지 않다. 다만 detail 함수의 매개변수 question_id가 추가된 점이 다르다. 바로 이것이 URL 매핑에 있던 question_id이다. 즉,/pybo/2/ 페이지가 호출되면 최종으로 detail 함수의 매개변수 question_id에 2가 전달된다.

4. pybo/question_detail.html 작성하기
3단계에서 render 함수가 question_detail.html 파일을 사용하고 있으므로 이에 대한 작업도 해야 한다. templates/pybo 디렉터리에 question_detail.html 파일을 만든 후 다음처럼 코드를 작성하자.
```
<h1>{{ question.subject }}</h1>

<div>
    {{ question.content }}
</div>
```
{{ question.subject }}, {{ question.content }}의 question 객체는 detail 함수에서 render 함수에 전달한 context에 저장한 데이터이다.

5. 질문 상세 페이지에 접속해 보기
/pybo/2/에 접속해 보자. 그러면 질문 상세 화면이 나타난다! 축하한다!

![2-04_br3.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_4_Part/2-04_br3.png)

## 오류 화면 구현하기
지금까지 질문 목록, 질문 상세 기능을 구현했다. 그런데 사용자가 잘못된 주소에 접속하면 어떻게 처리해야 할까?

1. 잘못된 주소에 접속해 보기
/pybo/30/에 접속해 보자. 그러면 다음과 같은 DoesNotExist 오류 화면이 나온다.

![2-04_br4.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_4_Part/2-04_br4.png)

당연한 오류이다. question_id가 30인 데이터를 조회하는 Question.object.get(id=30)에서 오류가 발생했기 때문이다.
>참고로 현재는 config/setting.py의 DEBUG 항목이 True로 설정되어 있어 개발자에게 여러 정보를 알려 주는 오류 화면이 나타난다. 그런데 실제 서비스 화면에는 그런 중요한 정보가 표현되면 안 된다. 그래서 서비르를 할 떄는 DEBUG 항목을 False로 설정한다.
> config/settings.py의 DEUBG 항목을 False로 설정하는 부분은 4장에서 자세히 다룬다.

2. 페이지가 존재하지 않음(404 페이지) 출력하기
존재하지 않는 페이지에 접속하면 오류 대신 404 페이지를 출력하도록 detail 함수를 수정하자. Question.object.get(id=question_id)를 get_object_or404(Question, pk=question_id)로 수정하면 된다.

![2-04_br5.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_4_Part/2-04_br5.png)

---

**404 페이지가 무엇인가요?**
웹 브라우저는 HTTP 요청을 하고, 장고는 그 요청에 응답을 한다. 보통의 경우에는 성공을 의미하는 200 응답 코드가 자동으로 반환된다. 하지만 요청하는 페이지가 없거나 서버에서 오류가 발생하면 다음과 같은 응답 코드가 반환된다.

| 오류 코드 | 설명 |
| :--- | :--- |
| 200 | 성공 |
| 500 | 서버 오류(Iternal Server Error) |
| 404 | 페이지 존재하지 않음(Not Found) |

---

---

**장고 제네릭뷰 간단 소개**
제네릭뷰는 목록 조회나 상세 조회처럼 특정 패턴이 있는 뷰를 작성할 때 사용하는 매우 편리한 기능이다. 하지만 장고 입문자에게 제네릭뷰의 실행 방식이 무척 이해하기 어렵다. 여러분에게는 제네릭뷰가 오히려 장고 학습에 혼란을 줄 수 있으므로 코너로 소개한다. 눈으로 코드를 살펴보기만 하자. 만약 우리가 views.py 파일에 작성한 index 함수나 detail 함수를 제네릭뷰로 변경하면 다음처럼 작성할 수 있다.

```
class IndexView(generic.ListView):
    """
    pybo 목록 출력
    """
    def get_queryset(self):
        return Question.objects.order_by('-create_date')


class DetailView(generic.DetailView):
    """
    pybo 내용 출력
    """
    model = Question
```
IndexView 클래스가 index 함수를 대체하고 DetailView 클래스가 detail 함수를 대체했다고 생각하면 된다. IndexView 클래스는 템플릿명이 명시적으로 지정되지 않으면 자동으로 모델명_list.html를 템플릿명으로 사용한다. 정리하자면 Question 모델을 사용하는 IndexView 클래스는 question_list.html, DetailView 클래스는 question_detail.html을 템플릿명으로 사용한다. 이어서 urls.py 파일은 다음과 같이 변경해야 한다.

```
# 제네릭뷰 예시: urls.py
from django.urls import path

from . import views

app_name = 'pybo'
urlpatterns = [
    path('', views.IndexView.as_view()),
    path('<int:pk>/', views.DetailView.as_view()),
]
```

이렇듯 단순 모델의 목록 조회나 상세 조회는 제네릭뷰를 사용하면 매우 간편하다. 단 제네릭뷰는 복잡한 문제를 해결할 때 오히려 개발 난이도를 높이는 경우도 있으니 주의해야 한다.

---