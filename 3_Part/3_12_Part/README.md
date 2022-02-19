<!-- -
![](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_12_Part/django.png)
-->
# 추천 기능 추가하기
커뮤니티 성격의 게시판 서비스라면 '추천(좋아요)' 기능은 필수이므로 추천 기능을 만들어 보자.

## Question, Answer 모델 변경하기 - 다대다 관계
추천은 질문이나 답변에 적용해야 하는 요소이다. 그러려면 Question, Answer 모델에 추천인 필드 voter를 추가해야 한다. 게시판 서비스를 사용해 봤다면 글 1개에 여러 명이 추천할 수 있고, 반대로 1명이 여러 개의 글을 추천할 수 있음을 쉽게 알 수 있따. 그리고 이런 경우에는 모델의 다대다관계를 사용해야 한다.

1. Question 모델 수정하기
장고에서는 다대다 관계를 위해 **ManyToManyFiled**를 지원하므로 Question 모델에 이를 추가하면 된다.
```
(... 생략 ...)
class Question(models.Model):
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    subject = models.CharField(max_length=200)
    content = models.TextField()
    create_date = models.DateTimeField()
    modify_date = models.DateTimeField(null=True, blank=True)
    voter = models.ManyToManyField(User)  # 추천인 추가

    def __str__(self):
        return self.subject
(... 생략 ...)
```
voter = models.ManyToManyField(User)는 추천인 voter 필드를 ManyToManyFiled 관계로 추가한 것이다.

2. makemigrations 오류 해결하기
수정 후 makemigrations 명령을 실행하면 당므과 같은 오류가 발생한다.
> 개발 서버가 실행중이라면 같은 오류가 발생할 것이다.
```
ERRORS:
pybo.Question.author: (fields.E304) Reverse accessor for 'Question.author' clashes with reverse accessor for 'Question.voter'.
        HINT: Add or change a related_name argument to the definition for 'Question.author' or 'Question.voter'.
pybo.Question.voter: (fields.E304) Reverse accessor for 'Question.voter' clashes with reverse accessor for 'Question.author'.
        HINT: Add or change a related_name argument to the definition for 'Question.voter' or 'Question.author'.
```
오류 내용은 Question 모델에서 사용한 author와 voter 필드가 모두 User 모델을 참조하고 있는데, 추후 User.question_set과 같이 User 모델을 통해 Question 데이터에 접근할 경우 author 필드를 기준으로 할지 voter 필드를 기준으로 할지 장고는 알 수 없으므로 직접 정하라는 뜻이다. 이 문제는 오류 메시지의 'HINT'에 있는 related_name 옵션 추가로 해결할 수 있다.

3. Quesiton 모델에 related_name 옵션 추가하기
Question 모델을 다음과 같이 수정하자.
```
class Question(models.Model):
    author = models.ForeignKey(User, on_delete=models.CASCADE, related_name='author_question')
    subject = models.CharField(max_length=200)
    content = models.TextField()
    create_date = models.DateTimeField()
    modify_date = models.DateTimeField(null=True, blank=True)
    voter = models.ManyToManyField(User, related_name='voter_question')

    def __str__(self):
        return self.subject
```
author 필드에는 related_name='author_question'을 추가하고, voter 필드에는 related_name='voter_question'을 추가했다. 이렇게 하면 특정 사용자가 작성한 질문을 얻기 위해 some_user.author_question.all()같은 코드를 사용할 수 있다.
> 특정 사용자가 추천한 질문을 얻기 위한 코드는 some_user.voter_question.all()이다.

4. Answer 모델 수정하기
마찬가지 방법으로 Answer 모델도 수정하자.
```
(... 생략 ...)
class Answer(models.Model):
    author = models.ForeignKey(User, on_delete=models.CASCADE, related_name='author_answer')
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    content = models.TextField()
    create_date = models.DateTimeField()
    modify_date = models.DateTimeField(null=True, blank=True)
    voter = models.ManyToManyField(User, related_name='voter_answer')
(... 생략 ...)
```

5. makemigrations, migrate 명령 실행하기
makemigrations 명령과 migrate 명령을 실행하자.
```
(mysite) c:\projects\mysite>python manage.py makemigrations
Migrations for 'pybo':
  pybo\migrations\0006_auto_20200423_1358.py
    - Add field voter to answer
    - Add field voter to question
    - Alter field author on answer
    - Alter field author on question

(mysite) c:\projects\mysite>python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, pybo, sessions
Running migrations:
  Applying pybo.0006_auto_20200423_1358(... 생략 ...) OK

(mysite) c:\projects\mysite> 
```
모델 변경을 완료했으니 이제 질문 추천 기능을 만들어 보자. 질문 추천을 할 수 있는 위치는 어디일까? 그렇다. 질문 상세 화면이다. 질문 상세 템플릿을 수정하자.

## 질문 추천 기능 만들기

1. 질문 추천 버튼 만들기
지금부터 질문 상세 템플릿의 구조를 꽤 많이 변경할 것이다. 클래스값 row, col-1, col-11을 이용하여 추천 영역의 너비는 전체 너비의 1/12로, 질문 영역의 너비는 전체 너비의 11/12로 만들자. 추천 영역에는 질문 추천 개수, <추천> 버튼을 추가한다. 질문 영역은 기존 내용을 그대로 복사하면 된다. HTML 작성이 처음이라면 어려울 수 있다. 코드를 잘 보고 따라 입력하자.
> row, col-1, col-11의 자세한 내용은 부트스트랩의 Gird System 공식 문서를 참고하자.
> 부트스트랩 Grid System 공식 문서: https://getbootstrap.com/docs/4.5/layout/grid
> 소스가 복잡하므로 https://github.com/pahkey/djangobook/tree/3-12 를 참고하기 바란다.

```
{% extends 'base.html' %}
{% block content %}
<div class="container my-3">
    (... 생략 ...)
    <h2 class="border-bottom py-2">{{ question.subject }}</h2>
    <div class="row my-3">
        <div class="col-1"> <!-- 추천영역 -->
            <div class="bg-light text-center p-3 border font-weight-bolder mb-1">{{question.voter.count}}</div>
            <a href="#" data-uri="{% url 'pybo:vote_question' question.id  %}"
               class="recommend btn btn-sm btn-secondary btn-block my-1">추천</a>
        </div>
        <div class="col-11"> <!-- 질문영역 -->
            <!-- 기존내용 -->
            <div class="card"my-3>  <!-- my-3 삭제 -->
                <div class="card-body"> 
                   (... 생략 ...)
                </div>
            </div>
        </div>
    </div>
    <h5 class="border-bottom my-3 py-2">{{question.answer_set.count}}개의 답변이 있습니다.</h5>
    {% for answer in question.answer_set.all %}
    (... 생략 ...)
(... 생략 ...)
```

2. 추천 버튼 확인 창 만들기
<추천> 버튼을 눌렀을 때 '정말로 추천하시겠습니까?'라는 확인 창이 나타나야 하므로 다음코드를 추가하자.
```
(... 생략 ...)
{% block script %}
<script type='text/javascript'>
$(document).ready(function(){
    (... 생략 ...)
    $(".recommend").on('click', function() {
        if(confirm("정말로 추천하시겠습니까?")) {
            location.href = $(this).data('uri');
        }
    });
});
</script>
{% endblock %}
```
<추천> 버튼에 class="recommend"가 적용되어 있으므로 해당 엘리먼트를 찾아 주는 제이쿼리 코드 $(".recommend")를 이용했다. 또한 확인 창에서 <확인>을 누르면 data-uri 속성에 정의한 URL이 호출되도록 했다.

3. 질문 추천 URL 매핑 추가하기
{% url 'pybo:vote_question' question.id %}URL에 해당하는 URL 매핑을 추가하자.
```
(... 생략 ...)
from .views import base_views, question_views, answer_views, comment_views, vote_views
(... 생략 ...)

urlpatterns = [
    (... 생략 ...)
    # vote_views.py
    path('vote/question/<int:question_id>/', vote_views.vote_question, name='vote_question'),
]
```

4. 질문 추천 함수 추가하기
URL 매핑에 의해 실행될 함수를 추가하자. vote_views.py 파일 생성 후 다음 코드를 작성하면 된다.
```
from django.contrib import messages
from django.contrib.auth.decorators import login_required
from django.shortcuts import get_object_or_404, redirect

from ..models import Question


@login_required(login_url='common:login')
def vote_question(request, question_id):
    """
    pybo 질문추천등록
    """
    question = get_object_or_404(Question, pk=question_id)
    if request.user == question.author:
        messages.error(request, '본인이 작성한 글은 추천할수 없습니다')
    else:
        question.voter.add(request.user)
    return redirect('pybo:detail', question_id=question.id)
```
이때 자기 추천 방지를 위해 추천자와 글쓴이가 동일하면 추천 시 오류가 발생하도록 하였다. 그리고 Question 모델의 voter 필드는 ManyToManyField로 정의했으므로 question.voter.add(request.user)와 같이 add 함수로 추천인을 추가해야 한다.
> 같은 사용자가 하나의 질문을 여러 번 추가해도 추천 수가 증가하지 않는다. ManyToManyField는 중복을 허락하지 않는다.

5. 자신의 글 추천 시 ㅇ류 기능 추가하기
그리고 '본인이 작성한 글은 추천할 수 없습니다'라는 오류가 표시되도록 질문 상세 화면 위쪽에 오류 영역을 추가하자.
```
    <!-- 사용자오류 표시 -->
    {% if messages %}
    <div class="alert alert-danger my-3" role="alert">
    {% for message in messages %}
        <strong>{{ message.tags }}</strong>
        <ul><li>{{ message.message }}</li></ul>
    {% endfor %}
    </div>
    {% endif %}
    <h2 class="border-bottom py-2">{{ question.subject }}</h2>
    (... 생략 ...)
(... 생략 ...)
```
질문 상세 화면의 본문 왼쪽을 보면 <추천> 버튼이 생겼을 것이다. 버튼이 잘 작동하는지 확인하자. 자기 추천 방지도 잘 작동하는지 확인해 보자.

![3-12_1.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_12_Part/3-12_1.png)

![3-12_2.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_12_Part/3-12_2.png)

## 답변 추천 기능 만들기
답변 추천 기능은 질문 추천 기능과 같으므로 빠르게 만들 수 있다.

1. 답변 추천 버튼 만들기
답변의 추천 개수를 표시하고, <답변 추천> 버튼을 질문 상세 화면에 추가하자.
```
(... 생략 ...)
<h5 class="border-bottom my-3 py-2">{{question.answer_set.count}}개의 답변이 있습니다.</h5>
{% for answer in question.answer_set.all %}
<div class="row my-3">
    <div class="col-1">  <!-- 추천영역 -->
        <div class="bg-light text-center p-3 border font-weight-bolder mb-1">{{answer.voter.count}}</div>
        <a href="#" data-uri="{% url 'pybo:vote_answer' answer.id  %}" 
            class="recommend btn btn-sm btn-secondary btn-block my-1">추천</a>
    </div>
    <div class="col-11">  <!-- 답변영역 -->
        <!-- 기존내용 -->
        <div class="card"my-3>  <!-- my-3 삭제 -->
            <div class="card-body">
                (... 생략 ...)
            </div>
        </div>
    </div>
</div>
{% endfor %}
(... 생략 ...)
```

2. 답변 추천 URL 매핑 정보 추가하기
{% url 'pybo:vote_answer' answer.id %}이 추가되었으므로 URL 매핑을 추가하자.
```
(... 생략 ...)

urlpatterns = [
    (... 생략 ...)
    path('vote/answer/<int:answer_id>/', vote_views.vote_answer, name='vote_answer'),
]
```

3. 답변 추천 함수 작성하기
URL 매핑에 의해 실행될 vote_views.vote_answer 함수를 다음처럼 작성하자.
```
(... 생략 ...)

from ..models import Question, Answer

(... 생략 ...)

@login_required(login_url='common:login')
def vote_answer(request, answer_id):
    """
    pybo 답글추천등록
    """
    answer = get_object_or_404(Answer, pk=answer_id)
    if request.user == answer.author:
        messages.error(request, '본인이 작성한 글은 추천할수 없습니다')
    else:
        answer.voter.add(request.user)
    return redirect('pybo:detail', question_id=answer.question.id)
```
코드 수정 후 답변 추천 기능도 확인해 보자.

![3-12_3.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_12_Part/3-12_3.png)

## 질문 목록 화면에서 추천수 표시하기

1. 질문 목록에 추천수 표시하기
질문 목록에서 추천 개수를 표시하면 서비스를 잘하는 친절한 게시판으로 보인다. 질문 목록화면에 추천수를 추가해 보자.
```
(... 생략 ...)
<table class="table">
    <thead>
    <tr class="text-center thead-dark">
        <th>번호</th>
        <th>추천</th>
        <th style="width:50%">제목</th>
        <th>글쓴이</th>
        <th>작성일시</th>
    </tr>
    </thead>
    <tbody>
    {% if question_list %}
    {% for question in question_list %}
    <tr class="text-center">
        <td>
            <!-- 번호 = 전체건수 - 시작인덱스 - 현재인덱스 + 1 -->
            {{ question_list.paginator.count|sub:question_list.start_index|sub:forloop.counter0|add:1 }}
        </td>
        <td>
            {% if question.voter.all.count > 0 %}
            <span class="badge badge-warning px-2 py-1">{{ question.voter.all.count }}</span>
            {% endif %}
        </td>
        (... 생략 ...)
    </tr>
    {% endfor %}
    {% else %}
    (... 생략 ...)
    {% endif %}
    </tbody>
</table>
(... 생략 ...)
```
질문 목록 테이블에 '추천' 열을 추가하고, 부트스트랩의 badge 컴포넌트로 표시했다.

![3-12_4.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_12_Part/3-12_4.png)