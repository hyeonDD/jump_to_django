<!-- -
![](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_8_Part/django.png)
-->
# 글쓴이 표시하기
앞서 Question 모델과 Answer 모델에 auther 필드를 추가했다. 게시판의 게시물에는 '글쓴이'를 표시하는 것이 일반적이다. 질문 목록, 질문 상세 화면에 auther 필드를 이용하여 글쓴이를 표시해 보자.

## 질문 목록 화면에 글쓴이 표시하기

1. 질문 목록 템플릿 수정하기
글쓴이를 표사힉 위해 테이블 헤더에 글쓴이 항목을 추가하자.
```
(... 생략 ...)
<tr class="text-center thead-dark">
    <th>번호</th>
    <th style="width:50%">제목</th>
    <th>글쓴이</th>
    <th>작성일시</th>
</tr>
(... 생략 ...)
```
<th>글쓴이</th>를 추가했다. 그리고 th 엘리먼트를 가운데 정렬하도록 tr 엘리먼트에 text-center 클래스를 추가하고 제목의 너비가 전체 50$를 차지하도록 style="width:50%"도 지정해 주었다. 이어서 for 문에도 글쓴이를 적용하자.
```
(... 생략 ...)
{% for question in question_list %}
<tr class="text-center">
    <td>
        <!-- 번호 = 전체건수 - 시작인덱스 - 현재인덱스 + 1 -->
        {{ question_list.paginator.count|sub:question_list.start_index|sub:forloop.counter0|add:1 }}
    </td>
    <td class="text-left">
        <a href="{% url 'pybo:detail' question.id %}">{{ question.subject }}</a>
        {% if question.answer_set.count > 0 %}
        <span class="text-danger small ml-2">{{ question.answer_set.count }}</span>
        {% endif %}
    </td>
    <td>{{ question.author.username }}</td>  <!-- 글쓴이 추가 -->
    <td>{{ question.create_date }}</td>
</tr>
{% endfor %}
(... 생략 ...)
```
<td>{{ question.author.username }}</td>를 삽입하여 질문의 글쓴이를 표시했다. 그리고 테이블 내용을 가운데 정렬하도록 tr 엘리먼트에 text-center 클래스를 추가하고, 제목을 왼쪽 정렬하도록 text-left 클래스를 추가했다. 질문 목록 화면에 글쓴이가 추가되었다. 변경된 질문 목록 화면은 아래 그림과 같다.

![3-08_1.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_8_Part/3-08_1.png)

## 질문 상세 화면에 글쓴이 표시하기
1. 질문 상세 템플릿 수정하기
질문 상세 화면에도 다음과 같이 글쓴이를 추가하자.
```
(... 생략 ...)
<div class="card-body">
    <div class="card-text" style="white-space: pre-line;">{{ question.content }}</div>
    <div class="d-flex justify-content-end">
        <div class="badge badge-light p-2 text-left">
            <div class="mb-2">{{ question.author.username }}</div>
            <div>{{ question.create_date }}</div>
        </div>
    </div>
</div>
(... 생략 ...)
```
글쓴이와 작성일시가 함께 보이도록 수정했다. 그리고 부트스트랩을 이용하여 여백과 정렬등의 디자인도 살짝 변경했다.

2. 답변에 글쓴이 표시 기능 추가하기
답변에도 글쓴이를 추가하자. 1단계와 마찬가지로 작성일시 바로 위에 글쓴이를 표시하면 된다.
```
(... 생략 ...)
<div class="card-body">
    <div class="card-text" style="white-space: pre-line;">{{ question.content }}</div>
    <div class="d-flex justify-content-end">
        <div class="badge badge-light p-2 text-left">
            <div class="mb-2">{{ question.author.username }}</div>
            <div>{{ question.create_date }}</div>
        </div>
    </div>
</div>
(... 생략 ...)
```
질문 상세 화면의 질문과 답변에 글쓴이가 추가되었다.

![3-08_2.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_8_Part/3-08_2.png)