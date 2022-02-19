<!-- -
![](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_9_Part/django.png)
-->
# 게시물 수정 & 삭제 기능 추가하기
이번에는 작성한 게시물을 수정 & 삭제할 수 있는 기능을 추가할 것이다. 실습을 진행하다 보면 '비슷한 기능을 반복해서 구현한다는 느낌'을 받아 조금 지루할 수 있다. 하지만 게시물 수정 & 삭제 기능은 게시물 작성만큼 중요하다. 장고 개발 패턴을 연습할 수 있는 좋은 기회이므로 실습을 이해하며 따라가 보자.

## 모델 수정하기
1. Question, Answer 모델에 modify_date 필드 추가하기
질문, 답변을 언제 수정했는지 확인할 수 있도록 Question 모델과 Answer 모델에 수정일시를 의미하는 modify_date 필드를 추가하자.
```
(... 생략 ...)

class Question(models.Model):
    (... 생략 ...)
    modify_date = models.DateTimeField(null=True, blank=True)
    (... 생략 ...)

class Answer(models.Model):
    (... 생략 ...)
    modify_date = models.DateTimeField(null=True, blank=True)
```
null=True는 데이터베이스에서 modify_date 컬럼에 null을 허용한다는 의미이며, blank=Ture는 form.is_valid()를 통한 입력 폼 데이터 검사 시 값이 없어도 된다는 의미이다. 즉, null=True, blank=True는 어떤 조건으로든 값을 비워둘 수 있음을 의미한다.
수정일시는 수정한 경우에만 생성되는 데이터이므로 null=True, blank=True를 지정했다.

2. makemigrations, migrate 명령 수행하기
모델이 변경되었으므로 makemigrations, migrate 명령을 수행하자.
```
(mysite) c:\projects\mysite>python manage.py makemigrations
Migrations for 'pybo':
  pybo\migrations\0004_auto_20200331_0945.py
    - Add field modify_date to answer
    - Add field modify_date to question

(mysite) c:\projects\mysite>python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, pybo, sessions
Running migrations:
  Applying pybo.0004_auto_20200331_0945(... 생략 ...) OK

(mysite) c:\projects\mysite>  
```

## 질문 수정 기능 추가하기

1. 질문 수정 버튼 추가하기
질문 상세 화면에 질문 수정 버튼을 추가하자.
```
(... 생략 ...)
<div class="card my-3">
    <div class="card-body">
        <div class="card-text" style="white-space: pre-line;">{{ question.content }}</div>
        <div class="d-flex justify-content-end">
            <div class="badge badge-light p-2 text-left">
                <div class="mb-2">{{ question.author.username }}</div>
                <div>{{ question.create_date }}</div>
            </div>
        </div>
        {% if request.user == question.author %}
        <div class="my-3">
            <a href="{% url 'pybo:question_modify' question.id  %}" 
               class="btn btn-sm btn-outline-secondary">수정</a>
        </div>
        {% endif %}
    </div>
</div>
(... 생략 ...)
```
질문 수정 버튼은 로그인한 사용자와 글쓴이가 같은 경우에만 보여야 하므로 {if request.user == question.author %}와 같이 추가했다.

2. 질문 수정 버튼의 URL 매핑 추가하기
{% url 'pybo:question_modify' question.id %} URL이 추가되었으니 pybo/urls.py 파일을 수정하여 URL 매핑을 추가해야 한다.
```
(... 생략 ...)

urlpatterns = [
    (... 생략 ...)
    path('question/modify/<int:question_id>/', views.question_modify, name='question_modify'),
]
```

3. 질문 수정 함수 추가하기
URL 매핑에 등록한 views.question_modify 함수를 추가하자.
```
(... 생략 ...)
from django.contrib import messages

(... 생략 ...)

@login_required(login_url='common:login')
def question_modify(request, question_id):
    """
    pybo 질문수정
    """
    question = get_object_or_404(Question, pk=question_id)
    if request.user != question.author:
        messages.error(request, '수정권한이 없습니다')
        return redirect('pybo:detail', question_id=question.id)

    if request.method == "POST":
        form = QuestionForm(request.POST, instance=question)
        if form.is_valid():
            question = form.save(commit=False)
            question.modify_date = timezone.now()  # 수정일시 저장
            question.save()
            return redirect('pybo:detail', question_id=question.id)
    else:
        form = QuestionForm(instance=question)
    context = {'form': form}
    return render(request, 'pybo/question_form.html', context)
```
question_modify 함수는 로그인한 사용자(request.user)와 수정하려는 글쓴이(question.author)가 다르면 '수정권한이 없습니다'라는 오류가 발생하도록 작성했다.
> '수정권한이 없습니다' 오류를 발생시키기 위해 messages 모듈을 이용했다.
> messages 모듈은 장고가 제공하는 기능으로 오류를 임의로 발생시키고 싶은 경우에 사용한다. 이때 임의로 발생시킨 오류는 폼 필드와 관련이 없으므로 넌필드 오류에 해당한다.
> 03-5에서 필드, 넌필드 오류를 설명했다.

질문 상세 화면에서 <수정>을 누르면 \pybo\question\modify\2\ 페이지가 GET 방식으로 호출되어 질문 수정 화면이 나타나고, 질문 수정 화면에서 <저장하기>를 누르면 \pybo\question\modify\2\ 페이지가 POST 방식으로 호출되어 데이터 수정이 이뤄진다.
> 데이터 저장시 form 엘리먼트에 action 속성이 없으면 현재의 페이지로 폼을 전송한다.
> 질문 수정에서 사용한 템플릿은 질문 등록 시 사용한 pybo\question_form.html 파일을 그대로 사용한다.

이때 GET 요청으로 질문 수정 화면이 나타날 때 기존에 저장되어 있던 제목, 내용이 반영된 상태에서 수정을 시작할 수 있도록 다음과 같이 폼을 생성했다.
```
form = QuestionForm(instance=question)
```
이처럼 instance 매개변수에 question을 지정하면 기존 값을 폼에 채울 수 있다. 그러면 사용자는 질문 수정 시 제목, 내용이 채워진 상태의 폼에서 수정을 시작할 수 있을 것이다. POST요청으로 수정 내용을 반영하는 경우에는 다음과 같이 폼을 생성해야 한다.
```
form = QuestionForm(request.POST, instance=question)
```
위 코드는 조회한 질문 question을 기본값으로 하여 화면으로 전달받은 입력값들을 덮어써서 QuestionForm을 생성하라는 의미이다. 그리고 질문의 수정일시는 다음처럼 현재일시로 저장했다.
```
question.modify_date = timezone.now()
```

4. 질문 수정 확인하기
이제 로그인 사용자와 글쓴이가 같으면 질문 상세 화면에 <수정> 버튼이 보일 것이다.

![3-09_1.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_9_Part/3-09_1.png)

수정 기능이 잘 작동하는지 확인해 보자.

## 질문 삭제 기능 추가하기

1. 질문 삭제 버튼 추가하기
이제 질문은 삭제하는 기능을 추가해 보자. 다음처럼 <수정> 버튼 ㅂ로 옆에 <삭제> 버튼을 추가하자.
```
(... 생략 ...)
{% if request.user == question.author %}
<div class="my-3">
    <a href="{% url 'pybo:question_modify' question.id  %}"
       class="btn btn-sm btn-outline-secondary">수정</a>
    <a href="#" class="delete btn btn-sm btn-outline-secondary"
       data-uri="{% url 'pybo:question_delete' question.id  %}">삭제</a>
</div>
{% endif %}
```
<삭제> 버튼은 <수정> 버튼과는 달리 href 속성값을 "#"로 설정했다. 그리고 삭제를 실행할 URL을 얻기 위해 data-uri 속성을 추가하고, 삭제 함수가 실행될 수 있도록 class 속성에 "delete" 항목을 추가해 주었다.
> data-uri 속성은 제이쿼리에서 $(this).data('uri')와 같이 사용하여 그 값을 얻을 수 있따.

2. 질문 삭제 버튼에 jQuery 사용하기
삭제 기능에서 <삭제> 버튼을 구현할 때 '정말로 삭제하시겠습니까?'와 같은 확인 창을 보여주어야 한다. 이를 구현하려면 다음과 같은 코드가 필요하다. 잠시 제이쿼리 코드를 적용해야하는 단계이므로 손을 떼고 책을 읽어 보자.
```
<script type='text/javascript'>
$(document).ready(function(){
    $(".delete").on('click', function() {
        if(confirm("정말로 삭제하시겠습니까?")) {
            location.href = $(this).data('uri');
        }
    });
});
</script>
```
이 코드에서는 제이쿼리를 사용했다. 제이쿼리가 생소할 수도 있다. 하지만 코드는 크게 어렵지 않다. 정리하자면 <삭제> 버튼을 누르면 확인 창이 나타나고, 확인 창에서 <확인> 버튼을 누르면 앞서 입력했던 data-uri 속성값으로 URL이 호출된다. <취소> 버튼을 누르면 아무 일도 발생하지 않는다.
> $(document).ready(function())은 화면이 로드된 다음 자동으로 호출되는 제이쿼리 함수이다.

3. jQuery 실행을 위해 pybo\base.html 파일 수정하기
제이쿼리를 이용하여 자바스크립트를 작성하기 위해서는 제이쿼리 라이브러리 파일을 먼저 로드해야 한다. 알다시피 base.html 파일에 제이쿼리 라이브러리를 로드하는 코드가 이미 추가되어 있다. 따라서 모든 템플릿에서 제이쿼리 라이브러리를 사용하기 위해서는 다음처럼 base.html 파일을 수정해야 한다.
```
(... 생략 ...)
<!-- jQuery JS -->
<script src="{% static 'jquery-3.4.1.min.js' %}"></script>
<!-- Bootstrap JS -->
<script src="{% static 'bootstrap.min.js' %}"></script>
<!-- 자바스크립트 Start -->
{% block script %}
{% endblock %}
<!-- 자바스크립트 End -->
</body>
</html>
```

"<script src="{{ url_for('static', filename='jquery-3.4.1.min.js')}}> </script>" 문장 이후에 {% block script %}{% endblock %}를 추가했다. 이렇게 하면 base.html 파일을 상속받는 템플릿이 이 블록을 구현하여 제이쿼리를 사용한 코드를 작성할 수 있다.

4. 질문 템플릿에 삭제 알림 창 기능 추가하기
이제 question_detail.html 파일 아래에 {% block script %} {% endblock %}를 추가하고 질문을 삭제할 수 있도록 코드를 추가하자.
```
(... 생략 ...)
{% endblock %}
{% block script %}
<script type='text/javascript'>
$(document).ready(function(){
    $(".delete").on('click', function() {
        if(confirm("정말로 삭제하시겠습니까?")) {
            location.href = $(this).data('uri');
        }
    });
});
</script>
{% endblock %}
```

5. 질문 삭제 URL 매핑 추가하기
그리고 data-uri 속성에 {% url 'pybo:question_delete' question.id %}이 추가되었으므로 pybo\urls.py 파일에 URL 매핑을 추가해야 한다.
```
(... 생략 ...)

urlpatterns = [
    (... 생략 ...)
    path('question/delete/<int:question_id>/', views.question_delete, name='question_delete'),
]
```

6. 질문 삭제 함수 추가하기
URL 매핑에 추가한 views.question_delete 함수를 다음처럼 작성하자.
```
(... 생략 ...)

@login_required(login_url='common:login')
def question_delete(request, question_id):
    """
    pybo 질문삭제
    """
    question = get_object_or_404(Question, pk=question_id)
    if request.user != question.author:
        messages.error(request, '삭제권한이 없습니다')
        return redirect('pybo:detail', question_id=question.id)
    question.delete()
    return redirect('pybo:index')
```
question_delete 함수 역시 로그인이 필요하므로 @login_required 데코레이터를 적용하고 로그인한 사용자와 글쓴이가 동일한 경우에만 삭제할 수 있도록 했다.
질문 글쓴이와 로그인 상요자가 동일하면 질문 상세 화면에 이제 <삭제> 버튼이 나타날 것이다. 삭제가 잘 실행되는지 확인해 보자.

![3-09_2.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_9_Part/3-09_2.png)

## 답변 수정 & 삭제 기능 추가하기
이번에는 답변 수정 & 삭제 기능을 추가하자. 질문 수정 & 삭제 기능과 거의 비슷한 구성으로 실습을 진행한다. 다만 답변 수정은 답변 등록 템플릿이 따로 없으므로 답변 수정에 사용할 템플릿이 추가로 필요하다.
> 답변 등록은 질문 상세 화면 아래쪽에 텍스트 입력 창을 추가하여 만든 것이므로 질문 상세 템플릿을 답변 수정용ㅇ으로 사용하는 데는 적합하지 않다.
> 답변 수정 & 삭제 기능은 질문 수정 & 삭제 기능과 크게 차이 나지 않으므로 간단히 설명하고 넘어가겠다.

1. 답변 수정 버튼 추가하기
답변 목록이 출력되는 부분에 답변 수정 버튼을 추가하자.
```
(... 생략 ...)
{% for answer in question.answer_set.all %}
<div class="card my-3">
    <div class="card-body">
        (... 생략 ...)
        {% if request.user == answer.author %}
        <div class="my-3">
            <a href="{% url 'pybo:answer_modify' answer.id  %}" 
               class="btn btn-sm btn-outline-secondary">수정</a>
        </div>
        {% endif %}
    </div>
</div>
{% endfor %}
(... 생략 ...)
```

2. 답변 수정 URL 매핑 추가하기
{% url 'pybo:answer_modify' answer.id %}가 추가되었으므로 pybo\urls.py 파일에 URL 매핑을 추가해야 한다.
```
urlpatterns = [
    (... 생략 ...)
    path('answer/modify/<int:answer_id>/', views.answer_modify, name='answer_modify'),
]
```

3. 답변 수정 함수 추가하기
계속해서 views.answer_modify 함수를 추가하자.
```
(... 생략 ...)
from .models import Question, Answer

(... 생략 ...)

@login_required(login_url='common:login')
def answer_modify(request, answer_id):
    """
    pybo 답변수정
    """
    answer = get_object_or_404(Answer, pk=answer_id)
    if request.user != answer.author:
        messages.error(request, '수정권한이 없습니다')
        return redirect('pybo:detail', question_id=answer.question.id)

    if request.method == "POST":
        form = AnswerForm(request.POST, instance=answer)
        if form.is_valid():
            answer = form.save(commit=False)
            answer.modify_date = timezone.now()
            answer.save()
            return redirect('pybo:detail', question_id=answer.question.id)
    else:
        form = AnswerForm(instance=answer)
    context = {'answer': answer, 'form': form}
    return render(request, 'pybo/answer_form.html', context)
```

4. 답변 수정 폼 작성하기
답변 수정을 위한 템플릿 answer_form.html 파일은 별도로 만들어야 한다. 파일 생성 후 다음과 같은 코드를 입력하자.
```
{% extends 'base.html' %}

{% block content %}
<div class="container my-3">
    <form method="post" class="post-form">
        {% csrf_token %}
        {% include "form_errors.html" %}
        <div class="form-group">
            <label for="content">답변내용</label>
            <textarea class="form-control" name="content" id="content" 
                      rows="10">{{ form.content.value|default_if_none:'' }}</textarea>
        </div>
        <button type="submit" class="btn btn-primary">저장하기</button>
    </form>
</div>
{% endblock %}
```

답변 수정 기능도 질문 수정 기능과 마찬가지로 답변 등록 사용자와 로그인 사용자가 동일할때만 <수정> 버튼이 나타난다. 답변 수정 기능이 잘 작동하는지 확인해 보자.

![3-09_3.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_9_Part/3-09_3.png)

5. 답변 삭제 버튼 추가하기
질문 상세 화면에 답변 삭제 버튼을 추가하자.
```
(... 생략 ...)
{% for answer in question.answer_set.all %}
<div class="card my-3">
    <div class="card-body">
        (... 생략 ...)
        {% if request.user == answer.author %}
        <div class="my-3">
            <a href="{% url 'pybo:answer_modify' answer.id  %}" 
               class="btn btn-sm btn-outline-secondary">수정</a>
            <a href="#" class="delete btn btn-sm btn-outline-secondary " 
               data-uri="{% url 'pybo:answer_delete' answer.id  %}">삭제</a>
        </div>
        {% endif %}
    </div>
</div>
{% endfor %}
(... 생략 ...)
```
<수정> 버튼 옆에 <삭제> 버튼을 추가했다. 질문의 <삭제> 버튼과 마찬가지로 <삭제> 버튼에 delete 클래스를 적용했으므로 <삭제> 버튼을 누르면 data-uri 속성에 설정한 url이 실행될 것이다.

6. 답변 삭제 URL 매핑 추가하기
{% url 'pybo:answer_delete' answer.id %}이 추가되었으므로 pybo\urls.py 파일에 URL 매핑을 추가하자.
```
(... 생략 ...)

urlpatterns = [
    (... 생략 ...)
    path('answer/delete/<int:answer_id>/', views.answer_delete, name='answer_delete'),
]
```

7. 답변 삭제 함수 추가하기
URL 매핑에 의해 실행될 views.answer_delete 함수를 추가하자.
```
(... 생략 ...)

@login_required(login_url='common:login')
def answer_delete(request, answer_id):
    """
    pybo 답변삭제
    """
    answer = get_object_or_404(Answer, pk=answer_id)
    if request.user != answer.author:
        messages.error(request, '삭제권한이 없습니다')
    else:
        answer.delete()
    return redirect('pybo:detail', question_id=answer.question.id)
```
이제 질문 상세 화면에서 답변을 작성한 사용자와 로그인 사용자가 같으면 <삭제> 버튼이 나타날 것이다. 잘 작동하는지 확인해 보자.

![3-09_4.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_9_Part/3-09_4.png)

## 수정일시 표시하기
마지막으로 질문 상세 화면에서 수정일시를 확인할 수 있도록 템플릿을 수정해 보자.

1. 작성일시 왼쪽에 수정일시 추가하기
질문과 답변에는 이미 작성일시를 표시하고 있다. 작성일시 바로 왼쪽에 수정일시를 추가하자.
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
    (... 생략 ...)
</div>
(... 생략 ...)
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
        (... 생략 ...)
    </div>
</div>
{% endfor %}
(... 생략 ...)
```
이제 질문이나 답변을 수정하면 다음처럼 수정일시가 표시될 것이다.

![3-09_5.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_9_Part/3-09_5.png)