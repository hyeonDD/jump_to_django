<!-- -
![](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_11_Part/django.png)
-->
# views.py 파일 분리하기

지금까지 실습을 진행한 독자라면 views.py 파일에 함수가 늘어나 파일 관리의 불편함을 느꼇을 것이다. 점점 방대해지는 views.py 파일을 개선할 수 없을까? 당연히 있다. 여기서는 두가지 개선 방법을 알아본다.

## views.py 파일 분리하기
첫 번째 방법은 views.py 파일을 분리하고 나머지 파일을 수정하지 않는 방법이다. 이 방법이 전체 코드의 변화가 가장 적다.

1. views 디렉터리 생성하기
mystie\pybo\views 디렉터리를 생성하자.
```
(mysite) D:\code\vscode\github\jump_to_django\src\projects\mysite>mkdir views
```

2. views.py 파일을 분리하여 views 디렉터리에 각각 저장하기
views.py 팡리에 정의한 함수를 기능별로 분리하여 views 디렉터리에 다음의 파일로 저장하자. 우선 다음 표를 보고 views 디렉터리에 빈 파일을 만들어 놓자.

| 파일명 | 기능 | 함수 |
| :--- | :--- | :--- |
| base_views.py | 기본 관리 | index, detail |
| question_views.py | 질문 관리 | question_create, question_modify, question_delete |
| answer_views.py | 답변 관리 | answer_create, answer_modify, answer_delete |
| comment_views.py | 댓글 관리 | comment_create_question, ..., comment_delete_answer(총 6개) |

3. base_views.py 파일 작성하기
base_views.py 파일을 다음과 같이 작성하자. 함수의 기능을 바꾸거나 하지 않았으므로 함수 내용은 모두 생략했다.
```
from django.core.paginator import Paginator
from django.shortcuts import render, get_object_or_404

from ..models import Question


def index(request):
    (... 생략 ...)


def detail(request, question_id):
    (... 생략 ...)
```
> index, detail 함수는 views.py 파일의 함수를 그대로 복사하고 import 문은 views.py의 모든 import 문을 복사하자.
> 파이참에서는 pybo\views.py의 모든 import 문을 그대로 복사한 다음 Ctrl + Alt + O를 누르면 import 문을 쉽게 정리할 수 있다.

하지만 import문을 수정해야 한다. 기존의 pybo\views.py 파일은 같은 디렉터리의 models.py 파일을 from .models import Question와 같이 임포트했지만 이제는 views 디렉터리에 파일이 위치하므로 from ..models import Question과 같이 임포트해야 한다.

4. question_views.py 파일 작성하기
question_views.py 파일은 다음과 같이 작성하자.
```
from django.contrib import messages
from django.contrib.auth.decorators import login_required
from django.shortcuts import render, get_object_or_404, redirect
from django.utils import timezone

from ..forms import QuestionForm
from ..models import Question


@login_required(login_url='common:login')
def question_create(request):
   (... 생략 ...)


@login_required(login_url='common:login')
def question_modify(request, question_id):
    (... 생략 ...)


@login_required(login_url='common:login')
def question_delete(request, question_id):
    (... 생략 ...)
```

5. answer_views.py, comment_views.py 파일 작성하기
나머지 파일도 작성하자.
```
from django.contrib import messages
from django.contrib.auth.decorators import login_required
from django.shortcuts import render, get_object_or_404, redirect
from django.utils import timezone

from ..forms import AnswerForm
from ..models import Question, Answer


@login_required(login_url='common:login')
def answer_create(request, question_id):
    (... 생략 ...)


@login_required(login_url='common:login')
def answer_modify(request, answer_id):
    (... 생략 ...)


@login_required(login_url='common:login')
def answer_delete(request, answer_id):
    (... 생략 ...)
```

```
from django.contrib import messages
from django.contrib.auth.decorators import login_required
from django.shortcuts import render, get_object_or_404, redirect
from django.utils import timezone

from ..forms import CommentForm
from ..models import Question, Answer, Comment


@login_required(login_url='common:login')
def comment_create_question(request, question_id):
    (... 생략 ...)


@login_required(login_url='common:login')
def comment_modify_question(request, comment_id):
    (... 생략 ...)


@login_required(login_url='common:login')
def comment_delete_question(request, comment_id):
    (... 생략 ...)


@login_required(login_url='common:login')
def comment_create_answer(request, answer_id):
    (... 생략 ...)


@login_required(login_url='common:login')
def comment_modify_answer(request, comment_id):
    (... 생략 ...)


@login_required(login_url='common:login')
def comment_delete_answer(request, comment_id):
    (... 생략 ...)
```

6. __init__.py 작성하기
마지막으로 views 디렉터리에 __init__.py 파일을 생성하여 아래와 같이 작성하자.
```
from .base_views import *
from .question_views import *
from .answer_views import *
from .comment_views import *
```
이렇게 하면 views 디렉터리의 __init__.py 파일에서 views 디렉터리의 모든 함수를 임포트하므로 views.py 파일을 사용할 때와 차이가 없어 views 모듈을 임포트하여 사용하는 다른 파일들을 수정할 필요가 없다.

7. pybo\views.py 삭제하기
파일 작성이 끝났으니 pybo\views.py 파일을 삭제하자. 정리후 views 디렉터리의 모습은 아래 그림과 같다. 수정후 반드시 파이보가 제대로 작동하는지 확인해 보자.

![3-11_1.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_11_Part/3-11_1.png)

## 유지보수에 유리한 구조로 만들기
앞서 살펴본 방법에는 프로그램 디버깅 시 urls.py 파일부터 함수를 찾을 때 urls.py 파일에는 매핑한 함수명만 있으므로 어떤 파일의 함수인지를 알 수 없다는 단점이 있다. 여러분이야 매핑한 함수가 어느 파일에 있는지 알겠지만 다른 사람은모를 것이다. 그래서 필자는 여럿이 함께 하는 프로젝트에는 앞선 방법을 절대로 추천하지 않는다. 따라서 다음의 방법을 통해 유지보수에 유리한 구조로 프로젝트를 변경하자.

1. views 디렉터리의 __init__.py 삭제하기
views 디렉터리의 __init__.py 파일을 삭제하자.
```
(mysite) D:\code\vscode\github\jump_to_django\src\projects\mysite\pybo\views>del __init__.py
```

2. urls.py 파일 수정하기
그리고 pybo\urls.py 파일을 다음과 같이 수정하자.
```
from django.urls import path

from .views import base_views, question_views, answer_views, comment_views

app_name = 'pybo'

urlpatterns = [
    # base_views.py
    path('',
         base_views.index, name='index'),
    path('<int:question_id>/',
         base_views.detail, name='detail'),

    # question_views.py
    path('question/create/',
         question_views.question_create, name='question_create'),
    path('question/modify/<int:question_id>/',
         question_views.question_modify, name='question_modify'),
    path('question/delete/<int:question_id>/',
         question_views.question_delete, name='question_delete'),

    # answer_views.py
    path('answer/create/<int:question_id>/',
         answer_views.answer_create, name='answer_create'),
    path('answer/modify/<int:answer_id>/',
         answer_views.answer_modify, name='answer_modify'),
    path('answer/delete/<int:answer_id>/',
         answer_views.answer_delete, name='answer_delete'),

    # comment_views.py
    path('comment/create/question/<int:question_id>/',
         comment_views.comment_create_question, name='comment_create_question'),
    path('comment/modify/question/<int:comment_id>/',
         comment_views.comment_modify_question, name='comment_modify_question'),
    path('comment/delete/question/<int:comment_id>/',
         comment_views.comment_delete_question, name='comment_delete_question'),
    path('comment/create/answer/<int:answer_id>/',
         comment_views.comment_create_answer, name='comment_create_answer'),
    path('comment/modify/answer/<int:comment_id>/',
         comment_views.comment_modify_answer, name='comment_modify_answer'),
    path('comment/delete/answer/<int:comment_id>/',
         comment_views.comment_delete_answer, name='comment_delete_answer'),
]
```
모듈명이 표시되도록 URL 매핑 시 views.index을 base_views.index와 같이 변경했다. 모듈명이 있기 때문에 이제 누가 보더라도 어떤 파일의 어떤 함수인지 명확하게 인지할 수 있다.
> # base_views.py와 같이 주석도 작성했다.

3. urls.py 파일 수정하기
config\urls.py 파일의 index 함수에 해당하는 URL 매핑도 views 대신 base_views와 같이 수정하자.
```
from django.contrib import admin
from django.urls import include, path
from pybo.views import base_views

urlpatterns = [
    path('pybo/', include('pybo.urls')),
    path('common/', include('common.urls')),
    path('admin/', admin.site.urls),
    path('', base_views.index, name='index'),  # '/' 에 해당되는 path
]
```