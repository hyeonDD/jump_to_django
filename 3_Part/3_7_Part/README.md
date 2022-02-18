<!-- -
![](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_7_Part/django.png)
-->
# 모델에 글쓴이 추가하기
회원가입, 로그인, 로그아웃 기능이 완성되어 질문과 답변을 '누가' 작성했는지 알 수 있게 되었다. 이제 기능을 조금씩 다듬어서 파이보를 완벽하게 만들어 보자. 여기서는 Question, Answer 모델을 수정하여 '글쓴이'에 해당하는 author 필드를 추가할 것이다.

## Question 모델 수정하기
1. Question 모델에 author 필드 추가하기
Question 모델에 author 필드를 추가하자.
```
from django.db import models
from django.contrib.auth.models import User


class Question(models.Model):
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    (... 생략 ...)
```
author 필드는 User 모델을 ForeignKey로 적용하여 선언했다. User 모델은 django.contrib.auth 앱이 제공하는 모델이다. on_delete=models.CASCADE는 계정이 삭제되면 계정과 연결된 Question 모델 데이터를 모두 삭제하라는 의미이다.

2. makemigrations 명령 실행하고 author 필드 추가 문제 해결하기
모델을 수정했으므로 makemigrations 명령과 migrate 명령을 실행해야 한다. 그런데 makemigrations 명령을 실행하면 다음 메시지를 볼 수 있다.
```
(mysite) D:\code\vscode\github\jump_to_django\src\projects\mysite>python manage.py makemigrations
---------- D:\code\vscode\github\jump_to_django\src\projects\mysite
You are trying to add a non-nullable field 'author' to question without a default; we can't do that (the database needs something to populate existing rows).
Please select a fix:
 1) Provide a one-off default now (will be set on all existing rows with a null value for this column)
 2) Quit, and let me add a default in models.py
Select an option:
```
이 메시지가 나타난 이유는 Question 모델에 author 필드를 추가하면 이미 등록되어 있던 게시물에 author 필드에 해당되는 값이 저장되어야 하는데, 장고는 author 필드에 어떤 값을 넣어야 하는지 모르기 때문이다. 그래서 장고가 여러분에게 기존에 저장된 Question 모델 데이터에는 author 필드값으로 어떤 값을 저장해야 하는지 묻는 것이다.
이 문제를 해결하는 방법에는 2가지가 있다. 첫 번째 방법은 author 필드를 null로 설정하는 방법이고, 두 번째 방법은 기존 게시물에 추가될 author 필드의 값에 강제로 임의 계정 정보를 추가하는 방법이다. 질문, 답변에는 author 필드값이 무조건 있어야 하므로 두 번째 방법을 사용할 것이다. 위 메시지를 유지한 상태에서 '1'을 입력하자.
```
Select an option: 1
Please enter the default value now, as valid Python
The datetime and django.utils.timezone modules are available, so you can do e.g. timezone.now
Type 'exit' to exit this prompt
```
그러면 파이썬 셸이 다음처럼 구동된다. 여기서 '1'을 입력하자. 그러면 makemigrations가 완료되고 파이썬 셸이 종료된다.
```
>>> 1
Migrations for 'pybo':
  pybo\migrations\0002_question_author.py
    - Add field author to question
```
파이썬 셸에서 입력한 '1'은 최초 생성했던 슈퍼 유저의 id값이므로 기존 게시물의 author에는 슈퍼 유저가 등록될 것이다.

3. migrate 명령 실행하기
이제 migrate 명령으로 변경된 내용을 데이터베이스에 적용하자.
```
(mysite) D:\code\vscode\github\jump_to_django\src\projects\mysite>python manage.py migrate       
---------- D:\code\vscode\github\jump_to_django\src\projects\mysite
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, pybo, sessions
Running migrations:
  Applying pybo.0002_question_author... OK
```

## Answer 모델 수정하기
1. Answer 모델에 author 필드 추가하고 문제 해결하기
Question 모델과 같은 방법으로 Answer 모델에 author 필드를 추가하자.
```
class Answer(models.Model):
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    (... 생략 ...)
```
이어서 makemigrations 명령을 실행한 다음, Question 모델에 했던 것과 마찬가지로 파이썬 셸 작업을 진행하자.
```
(mysite) D:\code\vscode\github\jump_to_django\src\projects\mysite>python manage.py makemigrations
---------- D:\code\vscode\github\jump_to_django\src\projects\mysite
You are trying to add a non-nullable field 'author' to answer without a default; we can't do that (the database needs something to populate existing rows).
Please select a fix:
 1) Provide a one-off default now (will be set on all existing rows with a null value for this column)
 2) Quit, and let me add a default in models.py
Select an option: 1
Please enter the default value now, as valid Python
The datetime and django.utils.timezone modules are available, so you can do e.g. timezone.now
Type 'exit' to exit this prompt
>>> 1
Migrations for 'pybo':
  pybo\migrations\0003_answer_author.py
    - Add field author to answer
```
이어서 migrate 명령을 실행하자.
```
(mysite) D:\code\vscode\github\jump_to_django\src\projects\mysite>python manage.py migrate       
---------- D:\code\vscode\github\jump_to_django\src\projects\mysite
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, pybo, sessions
Running migrations:
  Applying pybo.0003_answer_author... OK
```
여기까지 잘 진행되면 이상 없이 잘 처리된 것이다.

---

author 필드에 null 허용하기

author 필드에 null을 허용하려면 다음처럼 필드 정의 시 null=Ture를 지정하면 된다.
```
author = models.ForeginKey(User, on_delete=models.CASCADE, null=Ture)
```
이렇게 설정하면 author 필드에 null을 허용하므로 기존 게시물의 author 필드가 추가될 때 값이 없어도된다.

---

## author 필드 적용하기
이제 Question, Answer 모델에 author 필드가 추가되었으므로 질문 등록시에 author 필드를 추가해야 한다.
> 질문, 답변에 글쓴이를 추가한다는 느낌으로 작업을 진행하자.

1. 답변 등록 함수 수정하기
answer_create함수를 수정하자.
```
def answer_create(request, question_id):
    (... 생략 ...)
        if form.is_valid():
            answer = form.save(commit=False)
            answer.author = request.user  # author 속성에 로그인 계정 저장
            (... 생략 ...)
    (... 생략 ...)
```
답변 글쓴이는 현재 로그인한 계정이므로 answer.author = request.user로 처리했다. request.user가 바로 현재 로그인한 계정의 User 모델 객체이다.

2. 질문 등록 함수 수정하기
question_create 함수도 마찬가지 방법으로 수정하자.
```
def question_create(request):
    (... 생략 ...)
        if form.is_valid():
            question = form.save(commit=False)
            question.author = request.user  # author 속성에 로그인 계정 저장
    (... 생략 ...)
```
질문 글쓴이도 question.author = request.user로 처리했다. 이제 다시 개발 서버를 시작하고 로그인한 다음 질문,답변 등록을 테스트해보자. 잘 될 것이다.
> 개발 서버 종료는 Ctrl + c를 누르면 된다.
> 개발 서버 시작은 project\mystie에서 python manage.py runserver 명령을 실행하면 된다.

## 로그인이 필요한 함수 설정하기

1. 로그아웃 상태에서 질문, 답변 등록해 보기 - 오류 발생
로그아웃 상태에서 질문 또는 답변을 등록하면 다음과 같은 ValueError 오류가 발생한다. 오류는 웹 브라우저, 명령 프롬프트 양쪽에서 모두 확인할 수 있다. 로그아웃 상태에서 직접 오류를 발생시켜 보자.

![3-07_1.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_7_Part/3-07_1.png)

```
ValueError: Cannot assign "<SimpleLazyObject: <django.contrib.auth.models.AnonymousUser object at 0x000001AB676D0F10>>": "Question.author" must be a "User" instance.
```
이 오류는 request.user가 user 객체가 아닌 AnonymousUser 객체라서 발생한 것이다. 조금 더 자세히 설명하자면 request.user에는 로그아웃 상태이면 AnonymousUser 객체가, 로그인 상태이면 User 객체가 들어있는데, 앞에서 우리는 author 필드를 정의할 때 user를 이용하도록 했다. 그래서 answer.author = request.user에서 User 대신 AnonymousUser가 대입되어 오류가 발생한 것이다.

2. 로그인이 필요한 함수에 @login_required 애너테이션(데코레이터) 적용하기
이 문제를 해결하려면 로그인이 필요한 함수 answer_create, question_create에 @login_required 데코레이터를 사용해야 한다.
```
from django.shortcuts import render, get_object_or_404, redirect
from django.utils import timezone
from .models import Question
from .forms import QuestionForm, AnswerForm
from django.core.paginator import Paginator
from django.contrib.auth.decorators import login_required

(... 생략 ...)

@login_required(login_url='common:login')
def answer_create(request, question_id):
    (... 생략 ...)

@login_required(login_url='common:login')
def question_create(request):
    (... 생략 ...)
```
쉽게 말해 answer_create함수와 question_create 함수는 request.user를 포함하고 있으므로 @login_required 데코레이터를 통해 로그인이 되었는지를 우선 검사하여 1단계에서 본 오류를 방지한다. 만약 로그아웃 상태에서 @login_required 데코레이터가 적용된 함수가 호출되면 자동으로 로그인 화면으로 이동할 것이다. @login_required 데코레이터의 login_url은 이동해야 할 로그인 화면의 URL을 의미한다. 코드 수정 후 로그아웃 상태에서 질문 등록, 답변을 등록을 시도하면 로그인 화면으로 이동되는지 확인해 보자.

3. URL의 next 인자로 로그인 성공 후 이동할 URL 지정하기
그런데 로그아웃 상태에서 '질문 등록하기'를 눌러 로그인 화면으로 전환된 상태에서 웹 브라우저 주소창의 URL을 보면 next 파라미터가 있을 것이다.

![3-07_2.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_7_Part/3-07_2.png)

이는 로그인 성공 후 next 파라미터에 있는 URL로 페이지를 이동해야 한다는 의미이다. 그런데 지금은 그렇게 되고 있지 않다.

4. 로그인 템플릿에 hidden 항목 추가하여 next 파라미터 활용하기
로그인 후 next 파라미터에 있는 URL로 페이지를 이동하려면 로그인 템플릿에 다음과 같이 hidden 항목 next를추가해야 한다.
```
(... 생략 ...)
<form method="post" class="post-form" action="{% url 'common:login' %}">
    {% csrf_token %}
    <input type="hidden" name="next" value="{{ next }}">  <!-- 로그인 성공후 이동되는 URL -->
    {% include "form_errors.html" %}
(... 생략 ...)
```
그러면 로그인 후 next 파라미터의 URL로 이동할 수 있을 것이다.

5. 로그아웃 상태에서 아예 글을 작성할 수 없게 만들기
하나만 더 고쳐 보자. 현재 질문 등록은 로그아웃 상태에서는 아예 글을 작성할 수 없어서 만족스럽다. 하지만 답변 등록은 로그아웃 상태에서도 글을 작성할 수 있다. 물론 답변 작성 후 <저장하기>를 누르면 자동으로 로그인 화면으로 이동되므로 큰 문제는 아니지만 '글을 작성할 수 있는 것처럼 보이는 현상'은 부자연스럽다. 로그아웃 상태에서 아예 답변을 작성할 수 없도록 만드는 방법은 disabled 속성을 이용하는 것이다. pybo\question_detail.html 파일을 다음과 같이 수정하자.
```
(... 생략 ...)
<div class="form-group">
    <textarea {% if not user.is_authenticated %}disabled{% endif %}
              name="content" id="content" class="form-control" rows="10"></textarea>
</div>
<input type="submit" value="답변등록" class="btn btn-primary">
(... 생략 ...)
```
로그인 상태가 아닌 경우 textarea 엘리먼트에 disabled 속성을 적용하여 입력을 못하게 만들었다.

![3-07_3.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_7_Part/3-07_3.png)
