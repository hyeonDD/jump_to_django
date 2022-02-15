<!-- -
![](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_8_Part/django.png)
-->
# 부트스트랩으로 더 쉽게 화면 꾸미기

웹 디자이너 없이 웹 프로그램을 만들다 보면 화면 디자인 작업을 하는 데 얼마나 많은 시간과 고민이 필요한지 알 수 있을 것이다. 이번에 소개하는 부트스랩은 개발자 혼자서도 화면을 괜찮은 수준으로 만들 수 있게 도와주는 도구다. 부트스랩은 트위터를 개발하면서 만들어졌고 지속적으로 관리되고 있는 오픈소스 프로젝트이다.

## 파이보에 부트스트랩 적용하기

파이보에 부트스랩을 적용해 멋진 모습으로 변신시켜 보자.

1. 부트스트랩 설치하기
웹 브라우저를 열고 다음 URL에 접속한 다음 <Download>를 눌러 부트스랩 설치 파일을 내려받자.
> 부트스트랩 4.5.3 버전 다운로드: https://getbootstrap.com/docs/4.5/getting-started/download
> 이 책에 실린 코드는 부트스랩 버전5에서 정상 동작하지 않는다. 혹시 실습한 내용이 동일하게 보이지 않으면 부트스트랩 버전이 5가 아닌지 확인해 보자. 이 책은 부트스트랩 4.5.3버전(버전 4의 마지막 버전)을기준으로 실습을 진행한다.
> 부트스랩은 2020년 12월 7일 기준으로 버전 5.0(베타)가 출시되었다.

![bootstrap_download.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_8_Part/bootstrap_download.png)

그러면 bootstrap-4.5.3-dist.zip 파일이 다운로드된다. 내려받은 파일의 압축을 해제하면 많은 파일이 들어 있다. 이 중에서 bootstrap.min.css 파일만 복사해서 mystie\static 디렉터리에 저장하자.

---

* 압축 파일에 있는 bootstrap.min.css 경로
bootstrap-4.5.3-dist.zip/bootstrap-4.5.3-dist/css/bootstrap.min.css

* 복사할 경로
C:\projects\mystie\static\bootstrap.min.css

---

2. 질문 목록 템플릿에 부트스트랩 적용하기
pybo\question_list.html 파일에 부트스트랩을 적용하기 위해 {% load static %} 태그와 link 엘리먼트를 추가하자.
```
{% load static %}
<link rel="stylesheet" type="text/css" href="{% static 'bootstrap.min.css' %}">
{% if question_list %}
(... 생략 ...)
```
이어서 부트스트랩을 이용하여 템플릿을 다음과 같이 재구성하자. 여기서 사용된 container, my-3, thread-dark 등이 바로 부트스랩이 제공하는 클래스이다.
```
{% load static %}
<link rel="stylesheet" type="text/css" href="{% static 'bootstrap.min.css' %}">
<div class="container my-3">
    <table class="table">
        <thead>
        <tr class="thead-dark">
            <th>번호</th>
            <th>제목</th>
            <th>작성일시</th>
        </tr>
        </thead>
        <tbody>
        {% if question_list %}
        {% for question in question_list %}
        <tr>
            <td>{{ forloop.counter }}</td>
            <td>
                <a href="{% url 'pybo:detail' question.id %}">{{ question.subject }}</a>
            </td>
            <td>{{ question.create_date }}</td>
        </tr>
        {% endfor %}
        {% else %}
        <tr>
            <td colspan="3">질문이 없습니다.</td>
        </tr>
        {% endif %}
        </tbody>
    </table>
</div>
```

![2-08_br2.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_8_Part/2-08_br2.png)

3. 질문 상세 템플릿에 부트스트랩 사용하기
질문 상세 템플릿도 다음과 같이 부트스트랩을 적용하자.
```
{% load static %}
<link rel="stylesheet" type="text/css" href="{% static 'bootstrap.min.css' %}">
<div class="container my-3">
    <h2 class="border-bottom py-2">{{ question.subject }}</h2>
    <div class="card my-3">
        <div class="card-body">
            <div class="card-text" style="white-space: pre-line;">{{ question.content }}</div>
            <div class="d-flex justify-content-end">
                <div class="badge badge-light p-2">
                    {{ question.create_date }}
                </div>
            </div>
        </div>
    </div>
    <h5 class="border-bottom my-3 py-2">{{question.answer_set.count}}개의 답변이 있습니다.</h5>
    {% for answer in question.answer_set.all %}
    <div class="card my-3">
        <div class="card-body">
            <div class="card-text" style="white-space: pre-line;">{{ answer.content }}</div>
            <div class="d-flex justify-content-end">
                <div class="badge badge-light p-2">
                    {{ answer.create_date }}
                </div>
            </div>
        </div>
    </div>
    {% endfor %}
    <form action="{% url 'pybo:answer_create' question.id %}" method="post" class="my-3">
        {% csrf_token %}
        <div class="form-group">
            <textarea name="content" id="content" class="form-control" rows="10"></textarea>
        </div>
        <input type="submit" value="답변등록" class="btn btn-primary">
    </form>
</div>
```

질문, 답변은 하나의 뭉치에 해당되므로 부트스랩의 card 컴포넌트를 사용했고, 질문 내용과 답변 내용은 style 속성으로 white-space: pre-line을 적용하여 텍스트의 줄바꿈을 정상적으로 보이게 만들었다. 부트스트랩 클래스 my-3은 상하 마진값 3을 의미한다. py-2는 상하 패딩값 2, p-2는 상하좌우 패딩값 2를 의미한다. d-flex justify-content-end는 컴포넌트 오른쪽 정렬을 의미한다.
> 부트스트랩 공식 문서(card 컴포넌트) : https://getbootstrap.com/docs/4.5/components/card

4. 질문 상세 화면 확인하기
질문 상세 화면이 어떻게 바뀌었는지 확인해 보자. 부트스랩을 사용하면 정말 빠르게 만족스러운 화면을 만들 수 있다.

![2-08_br3.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_8_Part/2-08_br3.png)