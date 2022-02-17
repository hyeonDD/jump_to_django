<!-- -
![](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_4_Part/django.png)
-->
# 질눔네 달린 답변 개수 표시하기

이제 질문 목록에 '해당 질문에 달린 답변 개수'를 표시할 수 있는 기능을 추가해 보자. 코드와 분량은 많지 않지만, '게시판 서비스를 더욱 서비스답게 만들어 주는 기능'이다.

## 질문에 달린 답변의 개수 표시하기
답변 개수는 다음처럼 게시물 제목 바로 오른쪽에 표시하자.
```
(... 생략 ...)
<td>
    <a href="{% url 'pybo:detail' question.id %}">{{ question.subject }}</a>
    {% if question.answer_set.count > 0 %}
    <span class="text-danger small ml-2">{{ question.answer_set.count }}</span>
    {% endif %}
</td>
<...>
```
{ if question.answer_set.count > 0 %}로 답변이 있는 경우를 검사하고, {{ question.answer_set.count }}로 답변 개수를 표시했다. 이제 답변이 있는 질문은 제목 오른쪽에 빨간색 숫자가 표시된다.

![3-04_1.png](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_4_Part/3-04_1.png)