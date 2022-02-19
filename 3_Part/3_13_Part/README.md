<!-- -
![](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_13_Part/django.png)
-->
# 스크롤 초기화 문제점 해결하기
파이보 서비스가 점점 완성되어 간다. 여기서는 파이보에 더 많은 기능을 추가하기 전에 '스크롤 초기화'문제점을 해결해 본다.

## 어떤 문제가 있을까?
답변을 작성하거나 수정하면 질문 상세 화면의 브라우저 스크롤바가 항상 페이지 상단으로 고정된다. 코드 오류는 아니지만 서비스답지 못한 현상이다. 답변을 작성한 다음에는 답변 위치에 스크롤이 있어야 자연스럽다. 이 문제를 해결해 보자. 여기서는 가장 쉽게 해결하는 방법을 소개한다.

**앵커 엘리먼트로 스크롤 문제 해결하는 원리 알아보기**
HTML에는 URL을 호출하면 원하는 위치로 스크롤을 이동시키는 앵커 엘리먼트 a가 있다. 예를 들어 HTML 중간에 <a name="django"></a>와 같이 앵커 엘리먼트를 위치시키고, 해당 HTML을 호출하는 URL 뒤에 #django를 붙여 주면 바로 해당 앵커 엘리먼트 위치로 스크롤이 이동한다. 이 원리로 스크롤 초기화 문제를 해결할 것이다. 그러면 답변 등록, 답변 수정, 댓글 등록, 댓글 수정 기능 순으로 앵커 엘리먼트를 적용해 보자.

## 답변 등록, 답변 수정 시 앵커 기능 추가하기

1. 질문 상세 화면에 앵커 엘리먼트 추가하기
질문 상세 화면에 답변 등록, 답변 수정을 할 때 이동해야 할 앵커 엘리먼트를 추가하자.
```
(... 생략 ...)
<h5 class="border-bottom my-3 py-2">{{question.answer_set.count}}개의 답변이 있습니다.</h5>
{% for answer in question.answer_set.all %}
<a name="answer_{{ answer.id }}"></a>
(... 생략 ...)
```
답변이 반복되어 표시되는 for 문 바로 다음에 <a name="answer_{{ answer.id }}"></a>와 같이 앵커 엘리먼트를 추가했다. 앵커 엘리먼트의 name 속성은 유일해야 하므로 answer _{{ answer.id }}와 같이 답변 id를 사용했다.

2. 앵커 엘리먼트로 이동할 수 있도록 redirect 수정하기
이제 답변을 등록하거나 수정할 때 1단계에서 지정한 앵커 엘리먼트로 이동하도록 코드를 수정하자. 다음은 답변 등록 또는 답변 수정을 한 뒤 사용했던 기존 코드의 일부이다.
```
return redirect('pybo:detail', question_id=question.id)
```
여기에 앵커 엘리먼트를 포함하면 다음과 같다.
```
return redirect('{}#answer_{}'.format(
    resolve_url('pybo:detail', question_id=question.id), answer.id))
```
pybo:detail에 #answer_2와 같은 앵커를 추가하기 위해 format과 resolve_url 함수를 사용했다. resolve_url 함수는 실제 호출되는 URL을 문자열로 반환하는 장고 함수이다. 위를 참고하여 answer_views.py 파일의 **answer_create, answer_modify**함수를 다음과 같이 수정하자.
```
(... 생략 ...)
from django.shortcuts import render, get_object_or_404, redirect, resolve_url
(... 생략 ...)

@login_required(login_url='common:login')
def answer_create(request, question_id):
    (... 생략 ...)
            return redirect('{}#answer_{}'.format(
                resolve_url('pybo:detail', question_id=question.id), answer.id))
    (... 생략 ...)


@login_required(login_url='common:login')
def answer_modify(request, answer_id):
    (... 생략 ...)

    if request.method == "POST":
        form = AnswerForm(request.POST, instance=answer)
        if form.is_valid():
            (... 생략 ...)
            return redirect('{}#answer_{}'.format(
                resolve_url('pybo:detail', question_id=answer.question.id), answer.id))
    (... 생략 ...)
```
answer_modify 함수를 보면 redirect 함수를 사용한 부분이 두 군데 있으므로 수정에 주의하자. 오류가 발생하면 실행하는 redirect 함수는 앵커 엘리먼트로 이동할 필요가 없으므로 수정하지 않았다. 수정한 후 답변을 등록할 때 스크롤이 지정한 앵커 엘리먼트로 이동하는지 확인해 보자.
> 답변 수정을 할 때에도 똑같이 작동하는지 확인해 보자.

![3-13_1.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_13_Part/3-13_1.png)

화면에 1로 표시한 부분을 보면 상세 화면 URL에 #answer_9가 추가되었고, 2로 표시한 부분을 보면 스크롤이 해당 위치로 이동했음을 알 수 있다.

## 댓글에 앵커 기능 추가하기

1. 댓글 앵커 엘리먼트 추가하기
댓글에도 앵커 기능을 추가하자. 우선 댓글이 반복되는 구간에 댓글 앵커 엘리먼트를 추가하자.
```
(... 생략 ...)
<!-- 질문 댓글 Start -->
{% if question.comment_set.count > 0 %}
<div class="mt-3">
{% for comment in question.comment_set.all %}
    <a name="comment_{{ comment.id }}"></a>

(... 생략 ...)

<!-- 답변 댓글 Start -->
{% if answer.comment_set.count > 0 %}
<div class="mt-3">
{% for comment in answer.comment_set.all %}
    <a name="comment_{{ comment.id }}"></a>
(... 생략 ...)
```
질문,답변 댓글에 모두 앵커 엘리먼트 <a name="comment_{{ comment.id }}"></a>를 추가했다.

2. redirect 함수 수정하기
그리고 다음과 같이 comment_views.py 파일을 수정하자.
```
(... 생략 ...)
from django.shortcuts import render, get_object_or_404, redirect, resolve_url
(... 생략 ...)

@login_required(login_url='common:login')
def comment_create_question(request, question_id):
    (... 생략 ...)
    if request.method == "POST":
        form = CommentForm(request.POST)
        if form.is_valid():
            (... 생략 ...)
            return redirect('{}#comment_{}'.format(
                resolve_url('pybo:detail', question_id=comment.question.id), comment.id))
    (... 생략 ...)

@login_required(login_url='common:login')
def comment_modify_question(request, comment_id):
    (... 생략 ...)
    if request.method == "POST":
        form = CommentForm(request.POST, instance=comment)
        if form.is_valid():
            (... 생략 ...)
            return redirect('{}#comment_{}'.format(
                resolve_url('pybo:detail', question_id=comment.question.id), comment.id))
    (... 생략 ...)

@login_required(login_url='common:login')
def comment_create_answer(request, answer_id):
    (... 생략 ...)
    if request.method == "POST":
        form = CommentForm(request.POST)
        if form.is_valid():
            (... 생략 ...)
            return redirect('{}#comment_{}'.format(
                resolve_url('pybo:detail', question_id=comment.answer.question.id), comment.id))
    (... 생략 ...)

@login_required(login_url='common:login')
def comment_modify_answer(request, comment_id):
    (... 생략 ...)
    if request.method == "POST":
        form = CommentForm(request.POST, instance=comment)
        if form.is_valid():
            (... 생략 ...)
            return redirect('{}#comment_{}'.format(
                resolve_url('pybo:detail', question_id=comment.answer.question.id), comment.id))
    (... 생략 ...)
```
질문 또는 답변 댓글을 등록하거나 수정을 완료하면 해당 앵커 엘리먼트로 이동하도록 리다이렉트 URL을 수정했다. 댓글 삭제일 경우에는 화면을 자동으로 이동시킬 필요가 없으므로 앵커 엘리먼트를 추가하지 않았다. 이제 댓글을 작성한 다음 화면이 해당 앵커 엘리먼트로 이동하는지 확인하자.

![3-13_2.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_13_Part/3-13_2.png)

1로 표시한 부분을 보면 댓글을 작성할 때 URL에 #comment_4가 추가되고, 2로 표시한 부분을 보면 웹 브라우저의 스크롤바가 해당 위치로 이동하는 것을 확인할 수 있다.