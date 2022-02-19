<!-- -
![](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_10_Part/django.png)
-->
# 댓글 기능 추가하기
이제 질문과 답변에 댓글 기능을 추가해 보자. 댓글 기능을 추가하다 보면 파이보가 완벽한 게시판 서비스에 가까워지고 있다는 느낌이 들 것이다.

## 댓글에 사용할 모델 만들기

1. Comment 모델 만들기
댓글에 사용할 Comment 모델을 작성하자. 필드가 꽤 많으므로 누락되지 않도록 주의하며 입력하자.
```
(... 생략 ...)

class Comment(models.Model):
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    content = models.TextField()
    create_date = models.DateTimeField()
    modify_date = models.DateTimeField(null=True, blank=True)
    question = models.ForeignKey(Question, null=True, blank=True, on_delete=models.CASCADE)
    answer = models.ForeignKey(Answer, null=True, blank=True, on_delete=models.CASCADE)
```

Comment 모델의 필드 구성은 다음과 같다.

| 필드 | 설명 |
| :--- | :--- |
| author | 댓글 글쓴이 |
| content | 댓글 내용 |
| create_date | 댓글 작성일시 |
| modify_date | 댓글 수정일시 |
| question | 이 댓글이 달린 질문 |
| answer | 이 댓글이 달린 답변 |

질문에 댓글을 작성하면 Comment 모델의 question 필드에 값이 저장되고, 답변에 댓글이 작성되면 answer에 값이 저장된다. 다시 말해 Comment 모델 데이터에는 question 필드 또는 answer 필드 중 하나에만 값이 저장되므로 두 필드는 모두 null=True, blank=Ture여야 한다. 또한 question, answer 필드는 질문이나 답변 삭제 시 그에 달린 댓글까지 삭제되도록 on_delete=models.CASCADE를 설정했다.

2. makemigrations, migrate 명령 실행하기

Comment 모델이 추가되었으므로 makemigrations, migrate 명령을 실행하자.
```
(mysite) c:\projects\mysite>python manage.py makemigrations
Migrations for 'pybo':
  pybo\migrations\0005_comment.py
    - Create model Comment

(mysite) c:\projects\mysite>python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, pybo, sessions
Running migrations:
  Applying pybo.0005_comment(... 생략 ...) OK
```

## 질문 댓글 기능 추가하기
질문에 댓글을 등록할 수 있는 기능을 추가해 보자. 이제 장고에 새 기능을 추가하는 패턴에 익숙해졌을 것이다. 실습을 조금 빠르게 진행해 보자.

---

**장고 기능 개발 패턴 정리**
1. 화면에 버튼 등 기능 추가하기
2. urls.py 파일에 기능에 해당하는 URL 매핑 추가하기
3. forms.py 파일에 폼 작성하기
4. views.py 파일에 URL 매핑에 추가한 함수 작성하기
5. 함수에서 사용할 템플릿 작성하기

---

1. 질문 상세 화면에 댓글 목록과 댓글 입력 링크 추가하기
질문 상세 화면에 등록한 댓글을 보여 주고, 댓글을 등록할 수 있는 링크를 추가하자.
> div class="comment py-2 text-muted"에서 comment는 styles.css의 글씨를 작게 표시하기 위한 설정이다.
```
(... 생략 ...)
<div class="card-body">
    <div class="card-text" style="white-space: pre-line;">{{ question.content }}</div>
    <div class="d-flex justify-content-end">
        {% if question.modify_date %}
        <div class="badge badge-light p-2 text-left mx-3">
            <div class="mb-2">modified at</div>
            <div>{{ question.modify_date }}</div>
        </div>
        {% endif %}
        <div class="badge badge-light p-2 text-left">
            <div class="mb-2">{{ question.author.username }}</div>
            <div>{{ question.create_date }}</div>
        </div>
    </div>
    {% if request.user == question.author %}
    <div class="my-3">
        <a href="{% url 'pybo:question_modify' question.id  %}"
            class="btn btn-sm btn-outline-secondary">수정</a>
        <a href="#" class="delete btn btn-sm btn-outline-secondary"
            data-uri="{% url 'pybo:question_delete' question.id  %}">삭제</a>
    </div>
    {% endif %}
    <!-- 질문 댓글 Start -->
    {% if question.comment_set.count > 0 %}
    <div class="mt-3">
    {% for comment in question.comment_set.all %}  <!-- 등록한 댓글을 출력 -->
        <div class="comment py-2 text-muted">  <!-- 댓글 각각에 comment 스타일 지정 -->
            <span style="white-space: pre-line;">{{ comment.content }}</span>
            <span>
                - {{ comment.author }}, {{ comment.create_date }}
                {% if comment.modify_date %}
                (수정:{{ comment.modify_date }})
                {% endif %}
            </span>
            {% if request.user == comment.author %}
            <a href="{% url 'pybo:comment_modify_question' comment.id  %}" class="small">수정</a>,
            <a href="#" class="small delete" 
               data-uri="{% url 'pybo:comment_delete_question' comment.id  %}">삭제</a>
            {% endif %}
        </div>
    {% endfor %}
    </div>
    {% endif %}
    <div>
        <a href="{% url 'pybo:comment_create_question' question.id  %}" 
           class="small"><small>댓글 추가 ..</small></a>  <!-- 댓글 추가 링크 -->
    </div>
    <!-- 질문 댓글 End -->
</div>
(... 생략 ...)
```
질문에 등록된 댓글을 보여 주도록 {% for comment in question.comment_set.all %}와 {% endfor %}사이에 댓글 내용과 글쓴이, 작성일시를 출력했다. 또 댓글 글쓴이와 로그인한 사용자가 같으면 '수정', '삭제' 링크가 보이도록 했다. for 문 바깥쪽에는 댓글을 작성할 수 있는 '댓글 추가 ..'링크도 추가했다.
'코드를 자세히 보면 for 문으로 표시되는 댓글(div 엘리먼트)에 클래스값으로 comment를 지정했다. comment 클래스는 댓글을 작게 보여주는 클래스로 CSS를 별도로 작성해야 한다.

2. comment 클래스의 CSS 작성하기
지금까지 빈 파일로 남아 있던 style.css를 사용할 차례이다. 댓글마다 상단에 점선을 추하가ㅗ, 글꼴 크기를 0.7em으로 설정하도록 CSS를 작성하자.
```
.comment {
    border-top:dotted 1px #ddd;
    font-size:0.7em;
}
```

3. 질문 댓글 URL 매핑 추가하기
1단계에서 추가한 <수정>, <삭제>, <댓글 추가 ..>에 해당하는 URL 매핑을 pybo\urls.py 파일에 추가하자.
```
(... 생략 ...)

urlpatterns = [
    (... 생략 ...)
    path('comment/create/question/<int:question_id>/', views.comment_create_question, name='comment_create_question'),
    path('comment/modify/question/<int:comment_id>/', views.comment_modify_question, name='comment_modify_question'),
    path('comment/delete/question/<int:comment_id>/', views.comment_delete_question, name='comment_delete_question'),
]
```
댓글 등록, 수정, 삭제에 대한 URL 매핑을 한꺼번에 축했다. 댓글을 등록할 때는 질문의 id 번호 <int:question_id>가 필요하고 댓글을 수정하거나 삭제할 때는 댓글의 id 번호 <int:comment_id>가 필요함에 주의하자.

4. 질문 댓글 폼 작성하기
댓글 등록 시 사용할 CommentForm을 작성하자.
```
from django import forms
from pybo.models import Question, Answer, Comment

(... 생략 ...)

class CommentForm(forms.ModelForm):
    class Meta:
        model = Comment
        fields = ['content']
        labels = {
            'content': '댓글내용',
        }
```
댓글은 content 필드 하나면 충분하다.

5. 질문 댓글 등록 함수 추가하기
pybo\views.py 파일에 댓글 등록 함수 comment_create_question을 추가하자.
```
(... 생략 ...)
from .forms import QuestionForm, AnswerForm, CommentForm
(... 생략 ...)

@login_required(login_url='common:login')
def comment_create_question(request, question_id):
    """
    pybo 질문댓글등록
    """
    question = get_object_or_404(Question, pk=question_id)
    if request.method == "POST":
        form = CommentForm(request.POST)
        if form.is_valid():
            comment = form.save(commit=False)
            comment.author = request.user
            comment.create_date = timezone.now()
            comment.question = question
            comment.save()
            return redirect('pybo:detail', question_id=question.id)
    else:
        form = CommentForm()
    context = {'form': form}
    return render(request, 'pybo/comment_form.html', context)
```
질문 댓글 수정 함수는 질문 댓글 등록 함수와 다르지 않다. GET 방식이면 기존 댓글을 조회하여 폼에 반영하고 POST 방식이면 입력된 값으로 댓글을 업데이트한다. 업데이트 시 modify_date에 수정일시를 반영하는 것도 잊지 말자.

6. 질문 댓글 등록을 위한 템플릿 작성하기
댓글 등록을 위한 comment_form.html 템플릿 파일은 다음과 같이 작성하자. 파일 생성 후 다음과 같은 코드를 작성하자.
```
{% extends 'base.html' %}

{% block content %}
<div class="container my-3">
    <h5 class="border-bottom pb-2">댓글등록하기</h5>
    <form method="post" class="post-form my-3">
        {% csrf_token %}
        {% include "form_errors.html" %}
        <div class="form-group">
            <label for="content">댓글내용</label>
            <textarea class="form-control"name="content" id="content" 
                      rows="3">{{ form.content.value|default_if_none:'' }}</textarea>
        </div>
        <button type="submit" class="btn btn-primary">저장하기</button>
    </form>
</div>
{% endblock %}
```

7. 질문 댓글 수정 함수 추가하기
질문 댓글 수정을 위한 comment_modify_question 함수를 다음과 같이 추가하자.
```
from .models import Question, Answer, Comment
(... 생략 ...)

@login_required(login_url='common:login')
def comment_modify_question(request, comment_id):
    """
    pybo 질문댓글수정
    """
    comment = get_object_or_404(Comment, pk=comment_id)
    if request.user != comment.author:
        messages.error(request, '댓글수정권한이 없습니다')
        return redirect('pybo:detail', question_id=comment.question.id)

    if request.method == "POST":
        form = CommentForm(request.POST, instance=comment)
        if form.is_valid():
            comment = form.save(commit=False)
            comment.modify_date = timezone.now()
            comment.save()
            return redirect('pybo:detail', question_id=comment.question.id)
    else:
        form = CommentForm(instance=comment)
    context = {'form': form}
    return render(request, 'pybo/comment_form.html', context)
```

8. 질문 댓글 삭제 함수 추가하기
질문 댓글 삭제 함수 comment_delete_question을 추가하자.
```
(... 생략 ...)

@login_required(login_url='common:login')
def comment_delete_question(request, comment_id):
    """
    pybo 질문댓글삭제
    """
    comment = get_object_or_404(Comment, pk=comment_id)
    if request.user != comment.author:
        messages.error(request, '댓글삭제권한이 없습니다')
        return redirect('pybo:detail', question_id=comment.question.id)
    else:
        comment.delete()
    return redirect('pybo:detail', question_id=comment.question.id)
```
댓글 삭제 시 댓글이 있던 상세 화면으로 리다이렉트한다. 코드 수정 후 질문 상세 화면에서 <댓글 추가 ..>를 눌러 댓글을 추가해 보고 수정과 삭제 기능도 잘 작동하는지 확인해 보자.

![3-10_1.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_10_Part/3-10_1.png)

## 답변 댓글 기능 추가하기
답변에 댓글 기능을 추가하는 과정은 질문에 댓글 기능을 추가하는 과정과 크게 차이 나지 않는다. 실습을 빠르게 진행하자.

1. 답변에 댓글 목록을 보여주고 댓글 등록 링크 추가하기
질문 상세 화면의 답변 부분에 댓글의 목록을 보여주고 댓글을 추가할 수 있는 링크를 추가하자.
```
(... 생략 ...)
<h5 class="border-bottom my-3 py-2">{{question.answer_set.count}}개의 답변이 있습니다.</h5>
{% for answer in question.answer_set.all %}
<div class="card my-3">
    <div class="card-body">
        <div class="card-text" style="white-space: pre-line;">{{ answer.content }}</div>
        <div class="d-flex justify-content-end">
            {% if answer.modify_date %}
            <div class="badge badge-light p-2 text-left mx-3">
                <div class="mb-2">modified at</div>
                <div>{{ answer.modify_date }}</div>
            </div>
            {% endif %}
            <div class="badge badge-light p-2 text-left">
                <div class="mb-2">{{ answer.author.username }}</div>
                <div>{{ answer.create_date }}</div>
            </div>
        </div>
        {% if request.user == answer.author %}
        <div class="my-3">
            <a href="{% url 'pybo:answer_modify' answer.id  %}"
               class="btn btn-sm btn-outline-secondary">수정</a>
            <a href="#" class="delete btn btn-sm btn-outline-secondary "
               data-uri="{% url 'pybo:answer_delete' answer.id  %}">삭제</a>
        </div>
        {% endif %}
        {% if answer.comment_set.count > 0 %}
        <div class="mt-3">
        {% for comment in answer.comment_set.all %}
            <div class="comment py-2 text-muted">
                <span style="white-space: pre-line;">{{ comment.content }}</span>
                <span>
                    - {{ comment.author }}, {{ comment.create_date }}
                    {% if comment.modify_date %}
                    (수정:{{ comment.modify_date }})
                    {% endif %}
                </span>
                {% if request.user == comment.author %}
                <a href="{% url 'pybo:comment_modify_answer' comment.id  %}" class="small">수정</a>,
                <a href="#" class="small delete"
                   data-uri="{% url 'pybo:comment_delete_answer' comment.id  %}">삭제</a>
                {% endif %}
            </div>
        {% endfor %}
        </div>
        {% endif %}
        <div>
            <a href="{% url 'pybo:comment_create_answer' answer.id  %}"
               class="small"><small>댓글 추가 ..</small></a>
        </div>
    </div>
</div>
{% endfor %}
(... 생략 ...)
```
질문 부분에 댓글 기능을 추가했던 것과 차이가 없다. 다만 question.comment_set 대신 answer.comment_set을 사용한 점과, 댓글을 <수정>, <삭제>, <추가> 시 호출되는 URL만 다르다.

2. 답변 댓글 URL 매핑 추가하기
urls.py 파일에 1단계에 해당하는 URL 매핑을 추가하자.
```
(... 생략 ...)

urlpatterns = [
    (... 생략 ...)
    path('comment/create/answer/<int:answer_id>/', views.comment_create_answer, name='comment_create_answer'),
    path('comment/modify/answer/<int:comment_id>/', views.comment_modify_answer, name='comment_modify_answer'),
    path('comment/delete/answer/<int:comment_id>/', views.comment_delete_answer, name='comment_delete_answer'),
]
```
댓글을 등록할 때는 답변 id번호 <int:answer_id>가, 댓글을 수정 또는 삭제할 때는 댓글 id번호 <int:comment_id>가 사용되니 주의하자.

3. 답변 댓글 등록, 수정, 삭제 함수 추가하기
views.py 파일에 답변의 댓글을 등록, 수정, 삭제하기 위한 함수를 추가하자.
```
(... 생략 ...)

@login_required(login_url='common:login')
def comment_create_answer(request, answer_id):
    """
    pybo 답글댓글등록
    """
    answer = get_object_or_404(Answer, pk=answer_id)
    if request.method == "POST":
        form = CommentForm(request.POST)
        if form.is_valid():
            comment = form.save(commit=False)
            comment.author = request.user
            comment.create_date = timezone.now()
            comment.answer = answer
            comment.save()
            return redirect('pybo:detail', question_id=comment.answer.question.id)
    else:
        form = CommentForm()
    context = {'form': form}
    return render(request, 'pybo/comment_form.html', context)


@login_required(login_url='common:login')
def comment_modify_answer(request, comment_id):
    """
    pybo 답글댓글수정
    """
    comment = get_object_or_404(Comment, pk=comment_id)
    if request.user != comment.author:
        messages.error(request, '댓글수정권한이 없습니다')
        return redirect('pybo:detail', question_id=comment.answer.question.id)

    if request.method == "POST":
        form = CommentForm(request.POST, instance=comment)
        if form.is_valid():
            comment = form.save(commit=False)
            comment.modify_date = timezone.now()
            comment.save()
            return redirect('pybo:detail', question_id=comment.answer.question.id)
    else:
        form = CommentForm(instance=comment)
    context = {'form': form}
    return render(request, 'pybo/comment_form.html', context)


@login_required(login_url='common:login')
def comment_delete_answer(request, comment_id):
    """
    pybo 답글댓글삭제
    """
    comment = get_object_or_404(Comment, pk=comment_id)
    if request.user != comment.author:
        messages.error(request, '댓글삭제권한이 없습니다')
        return redirect('pybo:detail', question_id=comment.answer.question.id)
    else:
        comment.delete()
    return redirect('pybo:detail', question_id=comment.answer.question.id)
```
이번에 추가한 답변 댓글과 관련된 함수들은 질문 댓글의 함수들과 거의 차이가 없으므로 별도의 설명은 생략하겠다. 하나 짚고 넘어가자면 답변의 댓글을 등록하거나 수정하기 위해 사용한 폼과 템플릿은 질문 댓글에서 사용한 CommentForm과 comment_form.html 파일을 재활용할 수 있어서 별도의 코드를 작성할 필요가 없었다는 점이다.
> 답변 댓글에서 question_id를 얻어내기 위해 comment.answer.question과 같이 answer를 통해 question을 참조한 점도 확인하자.