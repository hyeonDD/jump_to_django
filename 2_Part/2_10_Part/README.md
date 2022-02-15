<!-- -
![](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_10_Part/django.png)
-->
# 질문 등록 기능 만들기

지금까지는 질문을 등록하기 위해 장고 셸이나 장고 Admin을 사용했다. 이번에는 파이보 서비스를 통해 질문을 등록하는 기능을 만들어 보자.

## 질문 등록 기능 만들기

질문 목록 화면 아래에 질문 등록 버튼을 만든 다음, 질문 등록 기능을 완성해 보자. 참고로 질문 등록 기능은 이 장 끝까지 진행해야 완벽하게 작동한다.

1. 질무느 등록 버튼 만들기
우선 다음처럼 질문 목록 템플릿을 열고 </table> 태그 아래에 질문 등록 버튼을 생성하자.
```
(... 생략 ...)
    </table>
    <a href="{% url 'pybo:question_create' %}" class="btn btn-primary">
        질문 등록하기
    </a>
</div>
{% endblock %}
```
a 엘리먼트에 href 속성으로 질문 등록 URL을 {% url 'pybo:question_create' %}처럼 추가하고 부트스트랩 클래스 "btn btn-primary"를 지정했다.

2. URL 매핑 추가를 위해 pybo\urls.py 수정하기
1단계에서 {% url 'pybo:question_create' %}이 추가되었으니 pybo\urls.py 파일에 URL 매핑을 추가하자.
```
(... 생략 ...)
urlpatterns = [
    (... 생략 ...)
    path('question/create/', views.question_create, name='question_create'),
]
```

3. pybo\views.py 수정하기
2단계에서 URL 매핑에 의해 실행될 views.question_create 함수를 작성하자.
```
from django.shortcuts import render, get_object_or_404, redirect
from django.utils import timezone
from .models import Question
from .forms import QuestionForm

(... 생략 ...)

def question_create(request):
    """
    pybo 질문등록
    """
    form = QuestionForm()
    return render(request, 'pybo/question_form.html', {'form': form})
```

question_create 함수는 QuestionForm 클래스로 생성한 객체 form을 사용할 것이다. 여기서 QuestionForm 클래스는 질문을 등록하기 위해 사용하는 장고의 폼이다. render 함수에 전달한 {'form': form}은 템플릿에서 폼 엘리먼트를 생성할 때 사용한다. 템플릿을 작성할 때 자세히 알아보겠다.
> QuestionFrom을 아직 작성하지 않아 IDE에서는 오류가 표시될 것이다. QuestionForm은 곧 만들 것이니 일단으 오류가 나오는 상태로 놔두자.

4. pbyo/forms.py에 장고 폼 작성하기
pybo 디렉터리 바로 아래에 forms.py 파일을 새로 만들어 ModelForm을 상속받은 QuestionForm 클래스를 작성하자. QuestionForm 클래스 안에 내부 클래스로 Meta 클래스를 작성하고, Meta 클래스 안에는 model, fields 속성을 다음과 같이 작성하자.
```
from django import forms
from pybo.models import Question


class QuestionForm(forms.ModelForm):
    class Meta:
        model = Question  # 사용할 모델
        fields = ['subject', 'content']  # QuestionForm에서 사용할 Question 모델의 속성
```
이 같은 클래스를 장고 폼이라 한다. 장고 폼은 사실 2개의 폼으로 구분할 수 있는데, forms.Form을 상속받으면 폼, forms.ModelForm을 상속받으면 모델 폼이라 부른다. 여기서는 form.ModelForm을 상속받아 모델 폼을 만들었다. 모델 폼은 말 그대로 모델과 연결된 폼이며, 모델폼 객체를 저장하면 연결된 모델의 데이터를 저장할 수 있다. 아직 모델 폼 객체를 저장한다는 의미가 잘 이해되지는 않겠지만, 곧 질문 등록 기능을 완성하여 이 내용을 자세히 설명하겠다. 내부 클래스로 선언한 Meta 클래스가 눈에 띌 것이다. 장고 모델 폼은 내부 클래스로 Meta 클래스를 반드시 가져야 하며, Meta 클래스에는 모델 폼이 사용할 모델과 모델의 필드들을 적어야 한다. QuestionForm 클래스는 Question 모델과 연결되어 있으며, 필드로 subject, content를 사용한다고 정의했다.

5. pybo\question_form.html 만들어 장고 폼 사용하기
질문 등록을 위해 pybo\question_form.html 파일을 생성하고 다음과 같이 작성하자.
```
{% extends 'base.html' %}

{% block content %}
<div class="container">
    <h5 class="my-3 border-bottom pb-2">질문등록</h5>
    <form method="post" class="post-form my-3">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit" class="btn btn-primary">저장하기</button>
    </form>
</div>
{% endblock %}
```
코드의 {{ form.as_p }}에서 form이 바로 question_create 함수에서 전달한 QuestionForm 객체이다. 여기서 {{ form.as_p }}는 모델 폼과 연결된 입력 항목 subject, content에 값을 입력할 수 있는 HTML 코드로 자동으로 만들어 준다.

6. 질문 등록 화면 확인하기
이제 웹 브라우저에서 지금까지 작업한 내용의 결과를 확인해 보자. \pybo\ 페이지를 요청해보자. 그러면 <질문 등록하기> 버튼이 표시될 것이다. <질문 등록하기> 버튼을 누르자.

![2-10_br1.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_10_Part/2-10_br1.png)

그러면 다음과 같은 질문 등록 화면이 나타나고 질문 등록 화면에는 템플릿에 작성한 {{ form.as_p }}에 의해 나타난 입력 항목 subject, content를 확인할 수 있다. 값을 입력하고 <저장하기> 버튼을 눌러 보자.

![2-10_br2.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_10_Part/2-10_br2.png)

그런데 아무런 반응이 없다. 왜냐하면 pybo\views.py 파일에 정의한 question_create 함수에 입력 데이터를 저장하기 위한 코드를 작성하지 않았기 때문이다. 이제 입력 데이터를 저장하는 방법에 대해 알아보자.

7. 입력 데이터 저장하기
pybo\views.py 파일의 question_create 함수를 수정하자. 코드는 꽤 기니 주의해서 입력하자.
```
def question_create(request):
    """
    pybo 질문등록
    """
    if request.method == 'POST':
        form = QuestionForm(request.POST)
        if form.is_valid():
            question = form.save(commit=False)
            question.create_date = timezone.now()
            question.save()
            return redirect('pybo:index')
    else:
        form = QuestionForm()
    context = {'form': form}
    return render(request, 'pybo/question_form.html', context)
```

가장 눈에 띄는 부분은 동일한 URL 요청을, POST, GET 요청 방식에 따라 다르게 처리한 부분이다. 질문 목록 화면에서 <질문 등록하기> 버튼을 누르면 \pybo\question\create\가 GET 방식으로 요청되어 질문 등록 화면이 나타나고, 질문 등록 화면에서 입력값을 채우고 <저장하기> 버튼을 누르면 \pybo\question\create\가 POST 방식으로 요청되어 데이터가 저장된다.
그리고 QuestionForm 객체도 GET 방식과 POST 방식일 경우 다르게 생성한 것에 주목하자. GET 방식의 경우 QuestionForm()과 같이 입력값 없이 객체를 생성했고 POST 방식의 경우에는 QuestionForm(request.POST)처럼 화면에서 전달받은 데이터로 폼의 값이 채워지도록 객체를 생성했다. from.is_valid 함수는 POST 요청으로 받은 form이 유효한지 검사한다. 폼이 유효하지 않다면 폼에 오류가 저장되어 화면에 전달될 것이다.
그리고 question = form.save(commit=Flase)는 form으로 Question 모델 데이터를 저장하기 위한 코드이다. 여기서 commit=Flase는 임시 저장을 의미한다. 즉, 실제 데이터는 아직 저장되지 않은 상태를 말한다. 이렇게 임시 저장을 사용하는 이유는 폼으로 질문 데이터를 저장할 경우 Question 모델의 create_date에 값이 설정되지 않아 오류가 발생하기 때문이다(폼에는 현재 subject, content 필드만 있고 create_date 필드는 없다).
이러한 이유로 임시 저장을 한 후 question 객체를 반환받아 create_date에 값을 설정한 후 question.save()로 실제 저장하는 것이다.
> from.save(commit=Flase) 대신 from.save()를 수행하면 create_date 속성값이 없다는 오류 메시지가 나타난다.

8. 질문 등록 기능 확인하기
웹 브라우저에서 질문 등록 기능을 확인해 보자. \pybo에 접속하여 <질문 등록하기> 버튼을 눌러 질문 등록 화면으로 이동하자. 그런 다음 질문 등록 기능을 확인하자.

![question.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_10_Part/question.png)

![2-10_5.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_10_Part/2-10_5.png)

9. 폼에 부트스트랩 적용하기
여기까지 진행하면 질문 등록 화면에 부트스트랩이 적용되지 않아서 아쉬움이 느껴질 것이다.그렇다. {{ form.as_p }}태그는 form 엘리먼트와 입력 항목을 자동으로 생성해 주므로 편리하기는 하지만 부트스트랩을 적용할 수 없다는 단점이 있다. 이 문제를 해결할 수는 없을까? 완벽하지는 않지만, QuestionForm 클래스 내부에 있는 Meta 클래스에 widgets 속성을 다음과 같이 추가하면 이 문제를 해결할 수 있다.
```
from django import forms
from pybo.models import Question


class QuestionForm(forms.ModelForm):
    class Meta:
        model = Question
        fields = ['subject', 'content']
        widgets = {
            'subject': forms.TextInput(attrs={'class': 'form-control'}),
            'content': forms.Textarea(attrs={'class': 'form-control', 'rows': 10}),
        }
```
다시 질문 등록 화면을 요청해 보면 다음과 같이 부트스트랩이 적용된 화면을 볼 수 있다.

![2-10_6.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_10_Part/2-10_6.png)

10. label 속성 수정하여 Subject, Content 한글로 변경하기
화면의 'Subject', 'Content'를 영문이 아니라 한글로 표시하고 싶다면 label 속성을 지정하면 된다.
```
from django import forms
from pybo.models import Question


class QuestionForm(forms.ModelForm):
    class Meta:
        model = Question
        fields = ['subject', 'content']
        widgets = {
            'subject': forms.TextInput(attrs={'class': 'form-control'}),
            'content': forms.Textarea(attrs={'class': 'form-control', 'rows': 10}),
        }
        labels = {
            'subject': '제목',
            'content': '내용',
        }  
```
그러면 'Subject'는 '제목'으로 'Content'는 '내용'으로 변경된다.

![2-10_7.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_10_Part/2-10_7.png)

> 장고 폼에 대한 자세한 내용은 장고 공식 문서를 참고하자.
> 장고 공식문서(폼): https://docs.djangoproject.com/en/3.0/topics/forms

11. 수작업으로 폼 작성하기
{{ form.as_p }}를 사용하면 빠르게 템플릿을 만들 수 있지만 HTML 코드가 자동으로 생성되므로 디자인 측면에서 많은 제한이 생기게 된다. 예를 들어 특정 태그를 추가하거나 필요한 클래스를 추가하는 작업에 제한이 생기게 된다. 또 디자인 영역과 서버 프로그램 영역이 혼재되어 웹 디자이너와 개발자의 역할을 분리하기도 애매해진다. 이번에는 자동으로 HTML 코드를 생성하지 말고 수작업으로 HTML코드를 작성해 보자. 우선 froms.py 파일의 widget 항목을 제거하자.
```
from django import forms
from pybo.models import Question


class QuestionForm(forms.ModelForm):
    class Meta:
        model = Question  # 사용할 모델
        fields = ['subject', 'content']  # QuestionForm에서 사용할 Question 모델의 속성
        """widgets = {
            'subject': forms.TextInput(attrs={'class': 'form-control'}),
            'content': forms.Textarea(attrs={'class': 'form-control', 'rows': 10}),
        }""" # widget 항목 삭제
        labels = {
            'subject': '제목',
            'content': '내용',
        }
```
그런 다음 질문 등록 템플릿을 다음과 같이 수정하자.
```
{% extends 'base.html' %}

{% block content %}
<div class="container">
    <h5 class="my-3 border-bottom pb-2">질문등록</h5>
    <form method="post" class="post-form my-3">
        {% csrf_token %}
        <!-- 오류표시 Start -->
        {% if form.errors %}
            <div class="alert alert-danger" role="alert">
            {% for field in form %}
                {% if field.errors %}
                <strong>{{ field.label }}</strong>
                {{ field.errors }}
                {% endif %}
            {% endfor %}
            </div>
        {% endif %}
        <!-- 오류표시 End -->
        <div class="form-group">
            <label for="subject">제목</label>
            <input type="text" class="form-control" name="subject" id="subject"
                   value="{{ form.subject.value|default_if_none:'' }}">
        </div>
        <div class="form-group">
            <label for="content">내용</label>
            <textarea class="form-control" name="content"
                      id="content" rows="10">{{ form.content.value|default_if_none:'' }}</textarea>
        </div>
        <button type="submit" class="btn btn-primary">저장하기</button>
    </form>
</div>
{% endblock %}
```
{{ form.as_p }}에 의해 자동 생성되는 HTML 대신 제목과 내용을 위한 HTML을 직접 작성했다. 그리고 question_create 함수에서 form.is_valid()가 실패했을 때 오류를 표시하기위해 오류 표시 영역도 추가했다. 제목의 value에는 {{ form.subject.value|default_if_none:'' }}을 대입했는데, 이는 오류 발생 시 기존 입력값을 유지하기 위함이다. |default_if_none:''는 form.subject.value에 값이 없으면 'None'이라는 문자열이 표시되는데, 이를 공백으로 표시하기 위해 사용한 템플릿 필터이다. 수정 후 질문 등록 화면으로 돌아가 제목만 입력하고 <저장하기> 버튼을 누르면 다음 화면을 볼 수 있다.

![2-10_br3.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_10_Part/2-10_br3.png)

필수 항목인 내용을 입력하지 않았으니 '내용을 입력하라'는 오류 메시지를 볼 수 있다. 또한 오류가 발생해도 이미 입력한 제목에 해당되는 값은 value 설정으로 인해 그대로 유지되는 것도 확인할 수 있다.

## 답변 등록 기능헤 장고 폼 적용하기

1. AnswerForm 클래스 추가하고 answer_create 함수 수정하기

질문 등록 기능헤 장고 폼을 적용한 것처럼 답변 등록 기능에도 장고 폼을 적용하자. 답변을 등록할 때 사용할 AnswerForm 클래스를 pybo\forms.py 파일에 같이 작성하자.
```
from django import forms
from pybo.models import Question, Answer

(... 생략 ...)

class AnswerForm(forms.ModelForm):
    class Meta:
        model = Answer
        fields = ['content']
        labels = {
            'content': '답변내용',
        }
```
그런 다음 pybo\views.py 파일의 answer_create 함수를 수정하자. answer_create 함수는 question-create 함수와 거의 동일하므로 자세한 설명은 생략한다.
```
(... 생략 ...)
from .forms import QuestionForm, AnswerForm
(... 생략 ...)

def answer_create(request, question_id):
    """
    pybo 답변등록
    """
    question = get_object_or_404(Question, pk=question_id)
    if request.method == "POST":
        form = AnswerForm(request.POST)
        if form.is_valid():
            answer = form.save(commit=False)
            answer.create_date = timezone.now()
            answer.question = question
            answer.save()
            return redirect('pybo:detail', question_id=question.id)
    else:
        form = AnswerForm()
    context = {'question': question, 'form': form}
    return render(request, 'pybo/question_detail.html', context)
```

2. 질문 상세 템플릿에 오류 표시 영역 추가하기
질문 상세 템플릿에 오류 표시 영역을 추가하자.
```
(... 생략 ...)
<form action="{% url 'pybo:answer_create' question.id %}" method="post" class="my-3">
    {% csrf_token %}
    {% if form.errors %}
    <div class="alert alert-danger" role="alert">
    {% for field in form %}
        {% if field.errors %}
        <strong>{{ field.label }}</strong>
        {{ field.errors }}
        {% endif %}
    {% endfor %}
    </div>
    {% endif %}
(... 생략 ...)
```
이렇게 수정하고 답변 내용 없이 답변을 등록하려고 하면 오류 메시지가 그림처럼 나타난다.

![2-10_9.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_10_Part/2-10_9.png)
